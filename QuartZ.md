## 1. 核心概念

### 1.1 Job和JobDetail

Job是一个接口，只有一个方法void execution(JobExecutionContext context)。一个Job代表一个任务。

QuartZ执行Job时，会重新创建一个Job实例。

QuartZ接收一个JobDetail。JobDetail存储着Job实现类和相关静态信息，包括Job名字，描述，关联监听等信息。每个JobDetail都有一个JobKey与其关联，JobKey可以唯一标识一个JobDetail，由name和group组成。调度器触发，停止，删除一个任务都是通过JobKey来实现的。

### 1.2 触发器(Trigger)

触发器是一个接口，描述触发Job执行的时间规则。主要有两个子接口：SimpleTrigger和CronTrigger。当需要触发一次或以固定时间周期执行，使用SimpleTrigger；而CronTrigger通过cron表达式定义各种复杂的调度时间。

TriggerBuilder会实例化一个Trigger。

每个Trigger都有一个TriggerKey与它关联。TriggerKey唯一标识一个Trigger，由name和group组成。

### 1.3 调度器(Scheduler)

Scheduler是一个接口，代表一个QuartZ的独立运行容器。JobDetail和Trigger成对地注册到Scheduler中。一旦注册成功，Scheduler负责在Trigger触发后执行相应的Job。通过SchedulerFactory创建Scheduler，并且在使用前必须调用start方法。

Scheduler提供了一系列对任务进行调度的方法，如触发、停止、重新开始。

一个Job可以对应多个Trigger，但是一个Trigger只能对应一个Job。


