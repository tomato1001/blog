---
layout: post
title: spring容器扩展
categories: spring extension
date: 2015-09-10 13:23:52
---

# BeanPostProcessor

当spring完成实例化、配置、初始化bean之后将调用此接口的方法。

{% highlight java %}
public static class SamplePostProcessor2 implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
{% endhighlight %}

# BeanFactoryPostProcessor

自定义bean的元数据信息，当spring容器读取并解析所有的配置之后，将调用此接口的方法。通过该接口可以在bean实例化之前，修改bean的配置信息。

{% highlight java %}
public static class SamplePostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        String[] names = beanFactory.getBeanNamesForType(Sample.class);
        for (String name : names) {
            System.out.println(name);
        }
    }
}
{% endhighlight %}