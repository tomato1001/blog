---
layout: post
title: spring quartz clustering
categories: quartz
date: 2016-01-20 10:45:02 +0800
---

# 介绍
如果你的app不需要集群时,不要使用quartz.不需要集群时,使用spring提供的scheduler将更加简单和方便.如果你的app处于集群环境中,则需要使用quartz clustering.因为spring scheduler将所有信息都存储在内存中,且不会在多个app中同步job信息,这样会导致以下问题: 

* 当1台机器挂掉后,scheduler的信息全部都会丢失
* 在集群环境中,会出现多个app同时执行相同job的情况.

使用quartz clustering可以解决上面的问题,当quartz处于clustering模式时,会将所有的信息存储在数据库中,且通过数据库锁机制解决了并发的问题,从而保证多个app只会有一个app执行指定的job.当一个app挂掉后,自动将job转移到其它可用的app.

spring-context-support模块提供了quartz的简单封装,可以使用spring来管理quartz.但是,我们需要自己解决下面的问题:

* 如何在job中注入spring的bean.(当job触发时,quartz会重新创建新的job实例)
* 如何以可读的方式存储job数据到数据库中

下面介绍spring与quartz集群环境集成方法

# spring quartz clustering

## 创建数据库与表
首先,在quartz官网下载完整的发布包,然后在docs/dbTables目录下找到对应的数据库脚本,比如:**tables_mysql_innodb.sql**,最后,使用该脚本创建数据库.
比较重要的表如下:

* qrtz_job_details - 包含job class的名称以及job使用的参数
* qrtz_cron_triggers - 包含所有的cron triggers

## Job与trigger
quartz包含2个主要的概念,**job**用于执行具体的逻辑,**trigger**用于触发job的执行,如:在未来的时间点执行指定的job.**注意:与spring的scheduler不同,quartz的job每次被触发执行时,将创建新的job实例**.
首先实现job.可以通过实现quartz的`Job`接口,或者spring的`QuartzJobBean`,下面是一个job实现,继承自`QuartzJobBean`.

* @DisallowConcurrentExecution限制job只能在一个app中执行
* @PersistJobDataAfterExecution当job执行完成后,将job关联的JobDataMap存储到表中.当使用该注解时,应同时使用@DisallowConcurrentExecution,用于防止并发造成的锁竞争

{% highlight java %}
@DisallowConcurrentExecution
@PersistJobDataAfterExecution
public class FirstJob extends QuartzJobBean {

    private String value;

    public void setValue(String value) {
        this.value = value;
    }

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        System.out.println(value);
    }
}
{% endhighlight %}
### 配置job与trigger

注意我们传递给job的参数是一个map,job直接以属性的方式接收值.该功能是通过`JobDetailFactoryBean`实现的.也可以在job的executeInternal方法中直接从JobExecutionContext中获取JobDataMap.

下面配置trigger并与job关联

{% highlight xml %}
<util:map id="firstJobDetailData">
    <entry key="value" value="11"/>
</util:map>
<!-- Defines jobdetail and trigger -->
<bean id="firstJobDetail" class="org.springframework.scheduling.quartz.JobDetailFactoryBean"
      p:jobClass="com.phome.samples.quartz.job.FirstJob"
      p:durability="true"
      p:jobDataAsMap-ref="firstJobDetailData"
/>
<bean id="firstTrigger" class="com.phome.samples.quartz.PersistableSpringCronTriggerFactoryBean"
      p:jobDetail-ref="firstJobDetail"
      p:cronExpression="0/20 * * * * ?"
/>
{% endhighlight %}

## 注入bean引用到job
在之前的job例子中,我们可以看到基本的值(支持序列化)可以传递到job并且当job每次创建时,会自动注入对应的值.这些值将会存储到`qrtz_job_details`表中,所以必须支持序列化.

如果我们的job需要依赖其它的spring bean时,比如service bean?这些bean又依赖其它的bean,将会包含注解,AOP等.我们不能将这些bean存储到job.

之前有提到,quartz的job每次被触发执行时,将创建新的job实例.这意味着quartz的job不是spring的bean,所以我们无法获得任何spring提供的好处.因此,我们需要重写`SpringBeanJobFactory`实现自己的job factory,增加自动注入支持.

{% highlight java %}
/**
 * Provides autowire Quartz Jobs with Spring context dependencies.
 *
 * @author zw
 */
public class AutowiringSpringBeanJobFactory extends SpringBeanJobFactory implements ApplicationContextAware {

    private transient AutowireCapableBeanFactory beanFactory;

    public void setApplicationContext(final ApplicationContext context) {
        beanFactory = context.getAutowireCapableBeanFactory();
    }

    @Override
    protected Object createJobInstance(final TriggerFiredBundle bundle) throws Exception {
        final Object job = super.createJobInstance(bundle);
        beanFactory.autowireBean(job);
        return job;
    }
}
{% endhighlight %}
上面自定义的Job factory将允许大部分的依赖注入方式.比如:@Autowired和@Resource注解方式的注入

## 配置Quartz scheduler
SchedulerFactoryBean用于创建Quartz scheduler.

让我们先来了解下spring的<i>*FactoryBean</i>是什么.当spring的bean使用一个普通的class定义时,spring将直接实例化该class的对象.但是,当class为<i>*FactoryBean</i>时,spring将先创建<i>*FactoryBean</i>的实例,然后调用**getObject()**用于创建bean对象.

比如SchedulerFactoryBean将会创建org.quartz.Scheduler对象实例.

以下为对应的SchedulerFactoryBean配置片段

{% highlight xml %}
<bean id="quartzScheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">

    <property name="configLocation" value="classpath:quartz.properties"/>
    <property name="dataSource" ref="dataSource"/>
    <property name="transactionManager" ref="transactionManager"/>

    <!-- This name is persisted to SCHED_NAME column in db. for local testing you could change to unique name
         to avoid collision -->
    <property name="schedulerName" value="quartzScheduler"/>

    <!-- Will update database cron triggers to what is in this jobs file on each deploy.
         Replaces all previous trigger and job data that was in the database. YMMV  -->
    <property name="overwriteExistingJobs" value="true"/>
    <property name="autoStartup" value="true"/>
    <property name="applicationContextSchedulerContextKey" value="applicationContext"/>
    <property name="jobFactory">
        <bean class="com.phome.samples.quartz.AutowiringSpringBeanJobFactory"/>
    </property>

    <!-- NOTE: Must add both the jobDetail and trigger to the scheduler! -->
    <property name="jobDetails">
        <list>
            <ref bean="firstJobDetail"/>
        </list>
    </property>
    <property name="triggers">
        <list>
            <ref bean="firstTrigger"/>
        </list>
    </property>
</bean>


<util:map id="firstJobDetailData">
    <entry key="value" value="11"/>
</util:map>

<!-- Defines jobdetail and trigger -->
<bean id="firstJobDetail" class="org.springframework.scheduling.quartz.JobDetailFactoryBean"
      p:jobClass="com.phome.samples.quartz.job.FirstJob"
      p:durability="true"
      p:jobDataAsMap-ref="firstJobDetailData"
/>

<bean id="firstTrigger" class="com.phome.samples.quartz.PersistableSpringCronTriggerFactoryBean"
      p:jobDetail-ref="firstJobDetail"
      p:cronExpression="0/20 * * * * ?"
/>
{% endhighlight %}

quartz.properties配置

{% highlight sh %}
#============================================================================
# Configure Main Scheduler Properties
#============================================================================


# Can be any string, and the value has no meaning to the scheduler itself - but rather
# serves as a mechanism for client code to distinguish schedulers when multiple instances
# are used within the same program. If you are using the clustering features, you must use
# the same name for every instance in the cluster that is 'logically' the same Scheduler.
org.quartz.scheduler.instanceName=LogStatisticClusteredScheduler

# Can be any string, but must be unique for all schedulers working as if they
# are the same 'logical' Scheduler within a cluster. You may use the value "AUTO"
# as the instanceId if you wish the Id to be generated for you. Or the value "SYS_PROP"
# if you want the value to come from the system property "org.quartz.scheduler.instanceId".
org.quartz.scheduler.instanceId=AUTO


#============================================================================
# Configure ThreadPool
#============================================================================

org.quartz.threadPool.class=org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount=50

# A boolean value ('true' or 'false') that specifies whether the threads spawned by Quartz
# will inherit the context ClassLoader of the initializing thread (thread that initializes the Quartz instance).
# This will affect Quartz main scheduling thread, JDBCJobStore's misfire handling thread (if JDBCJobStore is used),
# cluster recovery thread (if clustering is used), and threads in SimpleThreadPool (if SimpleThreadPool is used).
# Setting this value to 'true' may help with class loading, JNDI look-ups, and other issues related to using
# Quartz within an application server.
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread=true

#============================================================================
# Configure JobStore
#============================================================================

# The "use properties" flag instructs JDBCJobStore that all values in JobDataMaps will be Strings,
# and therefore can be stored as name-value pairs, rather than storing more complex objects in their
# serialized form in the BLOB column. This is can be handy, as you avoid the class versioning issues
# that can arise from serializing your non-String classes into a BLOB.
org.quartz.jobStore.useProperties=true
org.quartz.jobStore.isClustered=true
org.quartz.jobStore.tablePrefix=QRTZ_
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.clusterCheckinInterval=5000

# In case of we use Spring-managed DataSource instead of using a Quartz-managed connection pool
# so we don't need set it and spring framework will care about for us.You can see more details in
# org.springframework.scheduling.quartz.LocalDataSourceJobStore.
# You must set it if you doesn't use Spring-managed DataSource instead of using a Quartz-managed connection pool.
#org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX

# Disable the update check
org.quartz.scheduler.skipUpdateCheck=true
{% endhighlight %}

## 以可读的方式存储job数据到数据库中
因quartz采用java序列化机制存储数据,所以序列化后的数据完全没有可读性.当我们想知道当前job的执行参数时,只能自己写代码进行反序列化.可以通过设置**org.quartz.jobStore.useProperties=true**指定序列化的数据为字符串.**注意:当设置后所有的job参数类型只能为String,否则会报错**.

当spring与quartz集成时,上面的方法将会导致错误.因为Spring的<i>*TriggerFactoryBean</i>将job存储在trigger内.因job是复杂的对象,存储时必须使用java的序列化机制,这样就违反了**useProperties=true**设置.
所以,我们需要根据以下步骤来实现正确的**useProperties=true**行为:

* 重写**TriggerFactoryBean*,删除trigger中的job details
* 显式的传递所有的job到SchedulerFactoryBean

### 重写CronTriggerFactoryBean

{% highlight java %}
/**
 * An persistable <code>CronTriggerFactoryBean</code> patch for <code>spring + quartz</code>.
 * <p>You must set Quartz <code>org.quartz.jobStore.useProperties=true</code> when using Spring classes,
 * because Spring sets an object reference on JobDataMap that is not a String and will cause Quartz to fail
 * with an error.
 *
 * @author zw
 */
public class PersistableSpringCronTriggerFactoryBean extends CronTriggerFactoryBean {

    @Override
    public void afterPropertiesSet() {
        super.afterPropertiesSet();

        //Remove the JobDetail element
        getJobDataMap().remove(JobDetailAwareTrigger.JOB_DETAIL_KEY);
    }
}
{% endhighlight %}
配置trigger时,用PersistableSpringCronTriggerFactoryBean代替CronTriggerFactoryBean.

### 显式的传递所有的job到SchedulerFactoryBean
参考之前的`配置Quartz scheduler`部分

{% highlight xml %}
<!-- NOTE: Must add both the jobDetail and trigger to the scheduler! -->
<property name="jobDetails">
    <list>
        <ref bean="firstJobDetail"/>
    </list>
</property>
<property name="triggers">
    <list>
        <ref bean="firstTrigger"/>
    </list>
</property>
{% endhighlight %}

## 使用注解动态配置job和trigger

{% highlight java %}
@Configuration
public class AnnotationConfiguration {

    private static final List<String> domains = Arrays.asList(
            "a.com",
            "b.com"
    );

    @Autowired
    private ApplicationContext context;

    private static final String PREFIX_JOB_DETAIL = "jobDetail";
    private static final String PREFIX_JOB_TRIGGER = "jobTrigger";
    private static final Map<String, JobDetail> jobDetails = new HashMap<>();

    public JobDetailFactoryBean jobDetailFactoryBean(Class clz, String domain) {
        JobDetailFactoryBean jobDetailFactoryBean = new JobDetailFactoryBean();
        jobDetailFactoryBean.setDurability(true);
        jobDetailFactoryBean.setJobClass(clz);
        jobDetailFactoryBean.setName(PREFIX_JOB_DETAIL + domain);
        jobDetailFactoryBean.setApplicationContext(context);
        jobDetailFactoryBean.afterPropertiesSet();
        jobDetailFactoryBean.getJobDataMap().put("value", "3");
        return jobDetailFactoryBean;
    }

    @Bean(name = "job-details")
    public List<JobDetail> jobDetails() {
        List<JobDetail> details = new ArrayList<>();
        for (String domain: domains) {
            JobDetail detail = jobDetailFactoryBean(FirstJob.class, domain).getObject();
            details.add(detail);
            jobDetails.put(detail.getKey().getName(), detail);
        }
        return details;
    }

    @Bean(name = "job-triggers")
    @DependsOn("job-details")
    public List<CronTrigger> triggers() {
        List<CronTrigger> triggers = new ArrayList<>();
        for (String domain: domains) {
            PersistableSpringCronTriggerFactoryBean bean = new PersistableSpringCronTriggerFactoryBean();
            bean.setJobDetail(jobDetails.get(PREFIX_JOB_DETAIL + domain));
            bean.setCronExpression("0/20 * * * * ?");
            bean.setName(PREFIX_JOB_TRIGGER + domain);
            bean.afterPropertiesSet();
            triggers.add(bean.getObject());
        }
        return triggers;
    }
}
{% endhighlight %}
在SchedulerFactoryBean的配置中直接引用

{% highlight xml %}
<property name="jobDetails" ref="job-details"/>
<property name="triggers" ref="job-triggers"/>
{% endhighlight %}

## 手动控制quartz
根据之前的spring+quartz的集成方式,可以发现所有的job和参数以及trigger都是写在配置文件中,且都是在spring容器初始化时创建的.当我们需要在程序中动态的创建或停止job时,现有的方式无法达到目标.

其实,SchedulerFactoryBean只是封装了quartz并增加了一些实用的功能,实际上真正的操作还是由quartz的Scheduler来执行的.所以,我们只需要获取到quartz的Scheduler对象,然后使用quartz的api就可以实现动态创建和停止了.

代码片段如下:

{% highlight java %}
SchedulerFactoryBean schedulerFactoryBean = ctx.getBean(SchedulerFactoryBean.class);
JobDetail detail = JobBuilder.newJob(FirstJob.class)
        .usingJobData("value", "11111")
        .withIdentity("unique-1")
        .build();
Trigger trigger = TriggerBuilder.newTrigger()
        .withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(5))
        .build();
try {
    schedulerFactoryBean.getScheduler().scheduleJob(detail, trigger);
    schedulerFactoryBean.getScheduler().scheduleJob(detail, trigger);
} catch (Exception e) {
    // We will enconter exception if the job with name "unique-1" is added before
    e.printStackTrace();
}
Thread.sleep(10000);
schedulerFactoryBean.getScheduler().deleteJob(JobKey.jobKey("unique-1"));

Thread.sleep(10000);
{% endhighlight %}