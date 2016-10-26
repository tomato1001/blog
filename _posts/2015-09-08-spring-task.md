---
layout: post
title: Spring任务执行与调度
categories: spring task
date: 2015-09-08 15:19:52
---
# 简介
Spring框架通过```TaskExecutor```与```TaskScheduler```接口提供异步执行及调度功能。Spring内部通过2个接口的实现线程池或者代理到应用服务器环境中的```CommonJ```。

# Spring TaskExecutor
Spring的```TaskExecutor```接口与```java.util.concurrent.Executor```类似，该接口只有一个方法```execute(Runnable task)```，该方法接受一个task并在配置的线程池中执行。

# TaskExecutor类型
Spring内部提供了以下内部实现，以下实现基本可以达到日常使用的目的。

* ```SimpleAsyncTaskExecutor```

	异步执行实现，不支持线程重用，每次调用都创建一个新线程。支持并发限制，当达到限制后，调用方将阻塞。如果需要完整的线程池功能，可以使用```SimpleThreadPoolTaskExecutor```或者```ThreadPoolTaskExecutor```实现

* ```SyncTaskExecutor```

	同步执行实现，将在调用者的线程中执行。该实现的设计目的是用于多线程测试环境

* ```ConcurrentTaskExecutor```

	一个适配对象，适配到```java.util.concurrent.Executor```。

* ```SimpleThreadPoolTaskExecutor```

	Quartz集成实现。此类是Quartz的```SimpleThreadPool```子类，监听spring生命周期回调方法。当需要在Quartz和非Quartz环境中共享线程池时，你将需要使用此实现

* ```ThreadPoolTaskExecutor```

	最常用的线程池实现。通过该实现可以设置```java.util.concurrent.ThreadPoolExecutor```的参数。如果你需要适配到不同的```java.util.concurrent.Executor```时，建议使用```ConcurrentTaskExecutor```

* ```WorkManagerTaskExecutor```

	CommonJ集成实现

# 使用TaskExecutor
```TaskExecutor```使用方式与一般的javabean一样。下面的例子中，我们定义一个bean使用ThreadPoolTaskExecutor异步打印消息：

{% highlight java %}
import org.springframework.core.task.TaskExecutor;
public class TaskExecutorExample {
    private class MessagePrinterTask implements Runnable {
        private String message;
        public MessagePrinterTask(String message) {
            this.message = message;
        }
        public void run() {
            System.out.println(message);
        }
    }
    private TaskExecutor taskExecutor;
    public TaskExecutorExample(TaskExecutor taskExecutor) {
        this.taskExecutor = taskExecutor;
    }
    public void printMessages() {
        for(int i = 0; i < 25; i++) {
            taskExecutor.execute(new MessagePrinterTask("Message" + i));
        }
    }
}
{% endhighlight %}

{% highlight xml %}
<bean id="taskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
    <property name="corePoolSize" value="5" />
    <property name="maxPoolSize" value="10" />
    <property name="queueCapacity" value="25" />
</bean>
<bean id="taskExecutorExample" class="TaskExecutorExample">
    <constructor-arg ref="taskExecutor" />
</bean>
{% endhighlight %}

# TaskScheduler

{% highlight java %}
public interface TaskScheduler {
    ScheduledFuture schedule(Runnable task, Trigger trigger);
    ScheduledFuture schedule(Runnable task, Date startTime);
    ScheduledFuture scheduleAtFixedRate(Runnable task, Date startTime, long period);
    ScheduledFuture scheduleAtFixedRate(Runnable task, long period);
    ScheduledFuture scheduleWithFixedDelay(Runnable task, Date startTime, long delay);
    ScheduledFuture scheduleWithFixedDelay(Runnable task, long delay);
}
{% endhighlight %}
