---
layout: post
title: spring jpa entityManagerFactory自定义加载实体类
categories: spring jpa
date: 2015-09-02 09:12:42
---
在开发过程中，我们需要根据不同的环境加载不同的实体类。特别是实体类采用oop设计时，一个父类包含多个子类，但是，在系统运行过程中，永远只会使用一个子类。这样导致的后果就是当开启了自动建表的功能时，这些无用的子类也会体现在数据库中。下面是一种解决方案,通过设置spring的```LocalContainerEntityManagerFactoryBean```的属性```persistenceUnitPostProcessors```进行后处理。

使用sping集成jpa时，一般都需要配置JPA的```EntityManagerFactory```，spring对应的类为```org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean```，该类可以配置jpa实现所需要的信息，其中有一个```persistenceUnitPostProcessors```属性，该属性可以注入多个```PersistenceUnitPostProcessor```接口。通过注入自定义的```PersistenceUnitPostProcessor```实现，即可实现排除指定实体类的功能。

以下是一种实现方式，通过自定义的注解表示各个不同的环境，在```PersistenceUnitPostProcessor```实现中，根据当前的环境，排除其它环境的实体类。

{% highlight java %}
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
public @interface Production {
}

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
public @interface Sartf {
}

public class ProfileScanningPersistenceUnitPostProcessor implements PersistenceUnitPostProcessor {

    private static final Logger LOG = LoggerFactory.getLogger(ProfileScanningPersistenceUnitPostProcessor.class);

    private static final Map<Class<? extends Annotation>, String> EXCLUDE_PROFILE_MAPS = new HashMap<>();

    static {
        // Add all of profile annotation map
        EXCLUDE_PROFILE_MAPS.put(Production.class, Environment.Profiler.STR_PRODUCTION);
        EXCLUDE_PROFILE_MAPS.put(Sartf.class, Environment.Profiler.STR_SARTF);


        Class<? extends Annotation> activeAnnotationCls = null;
        for (Map.Entry<Class<? extends Annotation>, String> e : EXCLUDE_PROFILE_MAPS.entrySet()) {
            if (Environment.getProfiler().hasProfile(e.getValue())) {
                // We only enable single profile at the same time
                activeAnnotationCls = e.getKey();
                break;
            }
        }
        // Remove currently active profile
        if (activeAnnotationCls != null) {
            EXCLUDE_PROFILE_MAPS.remove(activeAnnotationCls);
        } else {
            EXCLUDE_PROFILE_MAPS.clear();
        }
    }


    @Override
    public void postProcessPersistenceUnitInfo(MutablePersistenceUnitInfo pui) {
        List<String> managedClassNames = pui.getManagedClassNames();
        for (Iterator<String> it = managedClassNames.iterator(); it.hasNext();) {
            String managedClassName = it.next();
            Class cls = ClassUtils.forName(managedClassName, false);
            if (cls != null) {
                Annotation[] annotations = cls.getAnnotations();
                for (Annotation annotation : annotations) {
                    if (EXCLUDE_PROFILE_MAPS.containsKey(annotation.annotationType())) {
                        LOG.info("Remove domain class: " + managedClassName);
                        it.remove();
                    }
                }
            }
        }
    }
}
{% endhighlight %}

{% highlight xml %}
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="persistenceUnitPostProcessors">
            <bean class="com.arcsoft.supervisor.commons.spring.ProfileScanningPersistenceUnitPostProcessor"/>
        </property>
        <property name="dataSource" ref="dataSource"/>
        <property name="packagesToScan" value="com.model.*"/>
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>
        </property>
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.dialect">${hibernate.dialect}</prop>
                <prop key="hibernate.hbm2ddl.auto">${hibernate.hbm2ddl.auto}</prop>
                <prop key="hibernate.show_sql">${hibernate.show_sql}</prop>
                <prop key="hibernate.format_sql">${hibernate.format_sql}</prop>
            </props>
        </property>
    </bean>
{% endhighlight %}