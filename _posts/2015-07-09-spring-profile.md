---
layout: post
title: spring中profile注解使用
categories: spring profile
date: 2015-09-30 15:39:08
---
# profile注解使用

## 简述
在开发过程中，随着功能的增加，我们的service可能同时会存在多个不同的实现，但是，同一时间只会使用一个特定的实现。比如：当我们的产品在多个不同的企业上线之后，A企业和B企业发现产品的用户管理模块无法满足他们现有的需求，然后各自针对实际情况提出了修改需求。一般我们可能会通过以下几种方式来解决这种问题：

* 为每个企业单独建立一个代码分支，为每个企业单独进行修改

* 不建立分支，重新编写一个实现

以上2种方式都存在不同的问题。第一种方式最极端，如果我们的主干代码进行了更新，比如：核心功能部分都是在主干代码库上维护，当核心库更新时，我们要对每一个分支进行合并操作，有过合并经验的人肯定知道合并的痛苦，非常浪费开发时间和成本。第二种方式相对完善一点，直接在一个分支里实现所有功能，免去了合并的问题，但是，使用spring时，当容器初始化时会将所有定义的bean都初始化，而我们在同一个企业里，只需要使用对应这个企业的实现，这样就会导致多余的实现被加载，而且这些实现永远都不会被使用。

通过profile就可以完美的解决上述的问题，而不会导致任何的缺点。使用profile后，当spring容器初始化时只会加载当前使用的profile。

通过设置系统属性`spring.profiles.active=dev`(多个用逗号分隔)或者通过spring的上下文设置当前使用的profile

``` java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("dev");
```

## 注解实现

### 扩展profile注解，定义每个profile
使用自定义注解方便使用和扩展，也可以直接使用`@profile('dev')`

{% highlight java %}
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Profile(Profiles.DEV)
public @interface Dev {
}

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Profile(Profiles.PRODUCTION)
public @interface Production {
}
{% endhighlight %}

### 使用

{% highlight java %}
@Configuration
public class AppConfig {

    /**
     * This bean only existed in dev
     *
     * @return
     */
    @Bean(name = "devSampleService")
    @Dev
    public SampleService sampleService1() {
        return new SampleServiceImpl1();
    }

    /**
     * This bean only existed in production
     *
     * @return
     */
    @Bean(name = "productionSampleService")
    @Production
    public SampleService sampleService2() {
        return new SampleServiceImpl2();
    }

    @Bean(name = "defaultSampleService")
    public SampleService defaultSampleService() {
        return new DefaultSampleServiceImpl();
    }

}
{% endhighlight %}

### 测试

{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
        classes = {
                AppConfig.class
        }
)
public class ProfileWithAnnotationTests {

    @Autowired
    private ApplicationContext ctx;

    @BeforeClass
    public static void beforeClass() {
        // Sets profile through system property. you
        // can also use -Dspring.profiles.active
//        System.setProperty("spring.profiles.active", "dev");

//        System.setProperty("spring.profiles.active", "production");
    }

    @Test
    public void test() {
        String activeProfile = System.getProperty("spring.profiles.active");
        SampleService sampleService = getSampleService("defaultSampleService");
        SampleService devSampleService = getSampleService("devSampleService");
        SampleService productionSampleService = getSampleService("productionSampleService");

        assertNotNull(sampleService);

        if (activeProfile == null) {
            assertNull(devSampleService);
            assertNull(productionSampleService);
        } else if (activeProfile.equals("dev")) {
            assertNotNull(devSampleService);
            assertNull(productionSampleService);
        } else if (activeProfile.equals("production")) {
            assertNotNull(productionSampleService);
            assertNull(devSampleService);
        }
    }

    private SampleService getSampleService(String name) {
        try {
            return ctx.getBean(name, SampleService.class);
        } catch (NoSuchBeanDefinitionException e) {
            System.err.println("No bean definition found with '" + name + "'");
        }
        return null;
    }
}
{% endhighlight %}

## 结合xml使用

{% highlight java %}
public interface Service {
}

@org.springframework.stereotype.Service("devService")
@Dev
public class ServiceImpl1 implements Service {
}

@org.springframework.stereotype.Service("productionService")
@Production
public class ServiceImpl2 implements Service {
}
{% endhighlight %}

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--
        Compose the annotation and xml.
     -->

    <context:component-scan base-package="profile.xml"/>

</beans>
{% endhighlight %}

{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {
        "classpath:profile/xml/profile.xml"
})
public class ProfileWithAnnotationAndXmlTests {

    @Autowired
    public ApplicationContext ctx;

    @BeforeClass
    public static void beforeClass() {
//        System.setProperty("spring.profiles.active", "dev");
        System.setProperty("spring.profiles.active", "production");
    }

    @Test
    public void test() {
        Service devService = getService("devService"),
                productionService = getService("productionService");
        String activeProfile = System.getProperty("spring.profiles.active");
        if (activeProfile == null) {
            assertNull(devService);
            assertNull(productionService);
        } else if (activeProfile.equals("dev")) {
            assertNotNull(devService);
            assertNull(productionService);
        } else if (activeProfile.equals("production")) {
            assertNotNull(productionService);
            assertNull(devService);
        }
    }

    private Service getService(String name) {
        try {
            return ctx.getBean(name, Service.class);
        } catch (NoSuchBeanDefinitionException e) {
            System.err.println("No bean definition found with '" + name + "'");
        }
        return null;
    }
}
{% endhighlight %}