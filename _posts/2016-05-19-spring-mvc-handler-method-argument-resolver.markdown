---
title: spring自定义HandlerMethodArgumentResolver
layout: post
categories: spring mvc handler-method-argument-resolver
date: 2016-05-19 16:28:23 +0800
---

当controller中的某个方法的参数为pojo时，spring会自动根据url参数名称采用反射的方式，将值设置到对应pojo的属性。如果，我们需要自定义参数解析过程，比如：对于特定的对象，根据自定义的方式实例化，此时，我们可以使用spring提供的HandlerMethodArgumentResolver来实现该功能。

# HandlerMethodArgumentResolver

该接口包含2个方法，supportsParameter用于验证当前实例是否支持对方法参数进行解析，当返回true时才会执行resolveArgument方法;resolveArgument方法则为实际的解析方法.

{% highlight java %}

public interface HandlerMethodArgumentResolver {

	boolean supportsParameter(MethodParameter parameter);

	Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception;

}
{% endhighlight %}

# 下面使用HandlerMethodArgumentResolver来实现一个分页参数自动解析的功能

分页在web开发中是一个常见的功能，通过url参数，pageNumber，pageSize，order以及orderBy等来实现跳页，指定页面尺寸，排序方式以及排序字段等。当前比较普遍的实现方式是，将这些属性放到一个分页的基类中，如：PageSupport，然后，需要分页条件的就继承该类。这样虽然能实现功能，但是却牺牲了代码的可扩展性以及灵活性。

使用HandlerMethodArgumentResolver就可以避免上面的情况，将分页相关的属性放到单独的类中，如：PageParameter，然后实现HandlerMethodArgumentResolver，对PageParameter进行单独解析。根据面向对象分类的话，此方案为组合，而之前提到的方案为继承方式。

以下为代码片段，该实现结合自定义注解，对orderBy字段进行校验。

{% highlight java %}
/**
 * Annotation to define some options for {@link PageParameter} to be used in controller.
 *
 * @author zw
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Page {

    /**
     * Defines all of fields that can be order by.
     *
     * @return fields can be order by
     */
    String[] orderBys();

    /**
     * Defines the order by field.
     *
     * @return the order by field
     */
    String orderBy() default "";


}
{% endhighlight %}

{% highlight java %}
public class PageParameterMethodArgumentResolver implements HandlerMethodArgumentResolver {

    private static final String DEFAULT_PAGE_NUMBER_PARAMETER = "pageNumber";
    private static final String DEFAULT_PAGE_SIZE_PARAMETER = "pageSize";
    private static final String DEFAULT_ORDER_BY_PARAMETER = "orderBy";
    private static final String DEFAULT_ORDER_PARAMETER = "order";

    private static final int DEFAULT_PAGE_SIZE = 10;
    private static final int DEFAULT_PAGE_NUMBER = 1;
    private static final Order DEFAULT_ORDER = Order.desc;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return PageParameter.class.equals(parameter.getParameterType());
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        Page annotation = parameter.getMethodAnnotation(Page.class);
        Assert.notNull(annotation, "Couldn't find @Page annotation in method " + parameter.getMethod());
        String pageNumberStr = webRequest.getParameter(DEFAULT_PAGE_NUMBER_PARAMETER);
        int pageNumber = StringUtils.isNotBlank(pageNumberStr)
                ? parseAndApplyBoundaries(pageNumberStr, DEFAULT_PAGE_NUMBER, Integer.MAX_VALUE)
                : DEFAULT_PAGE_NUMBER;

        String pageSizeStr = webRequest.getParameter(DEFAULT_PAGE_SIZE_PARAMETER);
        int pageSize = StringUtils.isNotBlank(pageSizeStr)
                ? parseAndApplyBoundaries(pageSizeStr, DEFAULT_PAGE_SIZE, Integer.MAX_VALUE)
                : DEFAULT_PAGE_SIZE;

        String orderByField = webRequest.getParameter(DEFAULT_ORDER_BY_PARAMETER);
        Order order = DEFAULT_ORDER;
        if (StringUtils.isNotBlank(orderByField)) {
            Order tempOrder = EnumUtils.getEnum(Order.class, webRequest.getParameter(DEFAULT_ORDER_PARAMETER));
            if (tempOrder != null) {
                order = tempOrder;
            }
        }
        return new PageParameter(pageNumber, pageSize, validateAndRetrieveOrderByField(annotation, orderByField), order);
    }

    private String validateAndRetrieveOrderByField(Page annotation, String orderBy) {
        String[] orderBys = annotation.orderBys();
        if (ArrayUtils.indexOf(orderBys, orderBy) == -1) {
            return StringUtils.isBlank(annotation.orderBy()) ? orderBys[0] : annotation.orderBy();
        }
        return orderBy;
    }


    private static int parseAndApplyBoundaries(String str, int lower, int upper) {
        int value = NumberUtils.toInt(str, lower);
        return value <= 0 ? lower : (value < lower ? value : value > upper ? upper : value);
    }
}
{% endhighlight %}

注册HandlerMethodArgumentResolver

{% highlight java %}
@Configuration
@EnableWebMvc
public class WebAppConfig extends WebMvcConfigurerAdapter{

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        PageParameterMethodArgumentResolver pageArgResolver = new PageParameterMethodArgumentResolver();
        argumentResolvers.add(pageArgResolver);
    }
}
{% endhighlight %}

使用

{% highlight java %}
@RequestMapping(value = "/result", method = RequestMethod.GET)
@ResponseBody
@Page(orderBys = {"end_time"})
public ResponseResult queryPagingBillResult(PageParameter page, BillResultCondition request) {
    return ResponseResults.ok(
            resultDao.pagingResult(page, request)
    );
}
{% endhighlight %}