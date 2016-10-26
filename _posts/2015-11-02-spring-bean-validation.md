---
layout: post
title: spring-bean-validation
categories: ['spring', 'bean-validation']
date: 2015-11-02 17:04:54
---
添加依赖

{% highlight xml %}
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.2.Final</version>
</dependency>

<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
{% endhighlight %}

* 配置spring

使用spring提供的LocalValidatorFactoryBean，并设置validationMessageSource作为国际化消息解析

{% highlight xml %}
<mvc:annotation-driven/>
<bean id="localValidatorFactoryBean" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"
        p:validationMessageSource-ref="messageSource"/>

<!-- Defines i18n message source -->
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource"
      p:basename="messages.message"
      p:defaultEncoding="UTF-8"/>
{% endhighlight %}

* 使用

bean文件

{% highlight java %}
@NotBlank
@Size(min = 2, max = 4)
private String name;

@NotNull
@Range(min = 1, max = 150)
private Integer age;
{% endhighlight %}

@NotBlank
判断空或null，截断空格

@NotNull
判断是否为null

controller
在需要验证的方法参数前使用```@Valid```注册并增加```BindingResult```参数

{% highlight java %}
public Map<String, String> saveOrUpdateUser(@Valid User user, BindingResult result) {
    if (result.hasErrors()) {
        return convertBindingResultErrors(result);
    } else {
        userService.saveOrUpdate(user);
        return Collections.emptyMap();
    }
}

protected Map<String, String> convertBindingResultErrors(BindingResult result) {
    Map<String, String> errors = new HashMap<>();
    for (ObjectError error: result.getAllErrors()) {
        errors.put(error.getObjectName(), messageSource.getMessage(error, null));
    }
    return errors;
}
{% endhighlight %}

消息文件
LocalValidatorFactoryBean在解析消息时的规则为注解名称.对象名.属性名，如：上面的saveOrUpdateUser中user对象包含一个name和age的字段，当验证未通过时，将在国际化资源文件中查找以下信息：

	name属性对应消息：
		NotBlank.user.name
		Size.user.name

	age属性对应消息:
		NotNull.user.age
		Range.user.age
	消息文件内容：

	NotBlank.user.name=姓名不能为空
	Size.user.name=姓名至少在2-4个字符
	typeMismatch.user.age=请输入合法的姓名
	NotNull.user.age=年龄不能为空
	Range.user.age=年龄必须介于1-150之间

消息文件中也可以使用表达式，比如：{0}, {1}，可以减少重复代码，但是增加了复杂度

	user.name=姓名
	age=年龄
	NotBlank.user.name={0}不能为空
	Size.user.name={0}至少在{2}-{1}个字符
	typeMismatch.user.age=请输入合法的{0}
	NotNull.user.age={0}不能为空
	Range.user.age={0}必须介于{2}-{1}之间

	SpringValidatorAdapter在传递消息参数时，默认将```对象名.属性名```对应的值作为参数，其它参数为validation注解定义的参数。参数的访问顺序对应为国际化资源key的拼音升序，如：上面的Size.user.name，Size注解包含min和max参数，属性对应的键user.name，然后将这3个参数按照拼音升序排序，最终的参数数组为：{user.name, max, min}

typeMismatch为类型不匹配的前缀，如：在age输入框中输入字符串

