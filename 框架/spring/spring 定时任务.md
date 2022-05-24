# 一、Spring Boot定时任务为什么是单线程的？

想要解释为什么，一定要从源码入手，直接从@EnableScheduling这个注解入手，找到了这个**ScheduledTaskRegistrar**类，其中有一段代码如下：

```java

	/**
	 * Schedule all registered tasks against the underlying
	 * {@linkplain #setTaskScheduler(TaskScheduler) task scheduler}.
	 */
	@SuppressWarnings("deprecation")
	protected void scheduleTasks() {
		if (this.taskScheduler == null) {
            // 如果，如果taskScheduler为null，则创建单线程的线程池
			this.localExecutor = Executors.newSingleThreadScheduledExecutor();
			this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
		}
		if (this.triggerTasks != null) {
			for (TriggerTask task : this.triggerTasks) {
				addScheduledTask(scheduleTriggerTask(task));
			}
		}
		if (this.cronTasks != null) {
			for (CronTask task : this.cronTasks) {
				addScheduledTask(scheduleCronTask(task));
			}
		}
		if (this.fixedRateTasks != null) {
			for (IntervalTask task : this.fixedRateTasks) {
				addScheduledTask(scheduleFixedRateTask(task));
			}
		}
		if (this.fixedDelayTasks != null) {
			for (IntervalTask task : this.fixedDelayTasks) {
				addScheduledTask(scheduleFixedDelayTask(task));
			}
		}
	}
```

# 二、配置多线程定时任务

- 方案1  **重写SchedulingConfigurer#configureTasks()**

```java
@Configuration
public class ScheduleConfig implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        //设定一个长度10的定时任务线程池
        taskRegistrar.setScheduler(Executors.newScheduledThreadPool(10));
    }
}
```

- 方案2. 通过配置开启

```properties
spring.task.scheduling.pool.size=10
```

- 方案3. **@Async**

使用@Async这个注解之前一定是要先配置线程池的，配置如下：

```java
@Bean
public ThreadPoolTaskExecutor taskExecutor() {
    ThreadPoolTaskExecutor poolTaskExecutor = new ThreadPoolTaskExecutor();
    poolTaskExecutor.setCorePoolSize(4);
    poolTaskExecutor.setMaxPoolSize(6);
    // 设置线程活跃时间（秒）
    poolTaskExecutor.setKeepAliveSeconds(120);
    // 设置队列容量
    poolTaskExecutor.setQueueCapacity(40);
    poolTaskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
    // 等待所有任务结束后再关闭线程池
    poolTaskExecutor.setWaitForTasksToCompleteOnShutdown(true);
    return poolTaskExecutor;
}
```

然后在@Scheduled方法上标注@Async这个注解即可实现多线程定时任务，代码如下：

```java
@Async
@Scheduled(cron = "0/5 * * * * ? ")
public void scheduled() {
    System.out.println("执行当时任务, 当前线程" + );
}
```

