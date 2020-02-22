---
title: Java基础-调度任务
abbrlink: 23e9e262
date: 2020-01-30 16:36:38
categories:
  - Java
  - Task
tags: [Java, Task]
---

# 调度任务
定时调度是给定时间点,给定时间间隔,给定次数进行的进行某项任务,实现方式有`timer`类实现和`quartz`插件实现.



## 实现方式

### [`timer`][1] 类

JDK提供的实现类,单线程实现对多个定时任务的调度.



### [`quartz`][2] 类

第三方提供插件,实现多线程调度任务.



## `timer` 类定时任务调度
JDK提供的定时调度实现,可以很方便的对指定次数,指定周期,指定时间的任务进行执行.
但是由于为单线程执行,并发难以应付;同时,如果一个定时调度抛出异常后,将会使整个调度系统停止.

### 示例代码

[GitHub地址](https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/timedTask/timer)

- `TimerTaskTest`自定义要被调度的任务
- `TimerTest`调度任务执行类



### `timer` 类说明

#### `TimerTaskTest`类
通过继承`TimerTask`类,创建自定义执行任务方法`printTask()`,并在`run()`方法中执行,1实现只打印三次线程执行者的任务.

- 继承自`TimerTask`类,创建线程属性
- 编写自定义方法,执行三次后`cancel()`取消调度
- 在`run()`方法中调用自定义方法

``` java
/**自定义定时任务*/
public class TimerTaskTest extends TimerTask{

	/**封装属性*/
	private String name;
	public  String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}

	/**执行计数*/
	public int count = 0;
	/**自定义打印任务*/
	public void printTask() {
		if (name != null) {
			if (count <= 3) {
				System.out.println("任务执行.....姓名:"+name);
				SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
				System.out.println("下次执行时间:"
							+ simpleDateFormat.format(scheduledExecutionTime()));
				count ++;
			}else{
				cancel();		//取消执行
				System.out.println("任务取消");
				scheduledExecutionTime();
			}
		}
	}

	@Override
	/**覆盖执行方法*/
	public void run() {
		//进行任务调度
		printTask();
	}
}
```

#### `TimerTest`类
定时调度任务类,可以根据时间点,间隔时间,自定义完成调度任务.

| 方法                                   | 参数                           | 说明                                              |
| -------------------------------------- | ------------------------------ | ------------------------------------------------- |
| schedule(task,time)                    | 执行任务,执行时间              | 在时间等于或者超过time的时候执行一次任务          |
| schedule(task,time,period)             | 执行任务,执行时间,间隔时间     | 在指定时间执行一次任务,随后间隔多少毫秒继续执行   |
| scheduleAtFixedRate(task,time,period)  | 执行任务,执行时间,间隔时间     | 在指定时间执行一次任务,随后间隔多少毫秒继续执行   |
| schedule(task,delay)                   | 执行任务,执行前的停顿          | 调用后延时一段时间后执行一次                      |
| schedule(task,delay,period)            | 执行任务,执行前的停顿,间隔时间 | 调用后延时一段时间后执行,随后间隔多少毫秒继续执行 |
| scheduleAtFixedRate(task,delay,period) | 执行任务,执行前的停顿,间隔时间 | 调用后延时一段时间后执行,随后间隔多少毫秒继续执行 |
| cancel()                               |                                | 取消执行方法                                      |
| scheduledExecutionTime()               |                                | 返回最近任务执行的时间,返回值为long型,需要转换    |
| purge()                                |                                |                                                   |


### **基本用法**
- 创建`Timer`调度实例
- 创建任务实例,并设置名称
- 调度执行`schedule()`传入参数

``` java
/**基本用法*/
public void testOne() {
	//1.创建调度实例
	Timer timer = new Timer();
	//2.创建任务实例
	TimerTaskTest task = new TimerTaskTest();
	task.setName("JION.JION");
	//3.执行定时调度:当前时间进行调度,随后没两秒进行一次.
	//执行任务的run方法,当前时间后多少毫秒,随后间隔多少毫秒
	timer.schedule(task, 2000L, 1000L);	
}
```

#### **`schedule()`指定时间进行调度**

``` java
/**设定时间,随后进行调度*/
public void testTwo() {
	Timer timer = new Timer();
	TimerTaskTest task = new TimerTaskTest();
	task.setName("JION.JION");
	//创建时间对象
	Calendar calendar = Calendar.getInstance();//单例模式
	//创建时间格式对象 
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
	//打印当前时间
	System.out.println("执行时间"+simpleDateFormat.format(calendar.getTime()));
	//为当前时间加5秒
	calendar.add(Calendar.SECOND, 5);
	//执行任务的run方法,当前时间后多少毫秒,随后线程阻塞
	timer.schedule(task, calendar.getTime());
}
```

#### **`schedule()`指定时间和间隔进行调度**

``` java
/**设定时间,随后进行调度,每隔毫秒数进行*/
public void testThree() {
	Timer timer = new Timer();
	TimerTaskTest task = new TimerTaskTest();
	task.setName("JION.JION");
	//创建时间对象
	Calendar calendar = Calendar.getInstance();//单例模式
	//创建时间格式对象 
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
	//打印当前时间
	System.out.println("执行时间"+simpleDateFormat.format(calendar.getTime()));
	//为当前时间加5秒
	calendar.add(Calendar.SECOND, 5);
	//执行任务的run方法,当前时间后多少毫秒,随后间隔2000毫秒,进行执行
	timer.schedule(task, calendar.getTime(),2000L);
}
```

#### **`schedule()`指定时间和延时时间进行调度**

``` java
/**设定时间,随后进行延时调度*/
public void testFour() {
	Timer timer = new Timer();
	TimerTaskTest task = new TimerTaskTest();
	task.setName("JION.JION");
	//创建时间对象
	Calendar calendar = Calendar.getInstance();//单例模式
	//创建时间格式对象 
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
	//打印当前时间
	System.out.println("执行时间"+simpleDateFormat.format(calendar.getTime()));
	//执行任务的run方法,延时2000毫秒,进行执行
	timer.schedule(task,2000L);
}
```

#### **`schedule()`指定时间和延时时间以及间隔进行调度**

``` java
/**设定时间,随后进行延时调度,每隔毫秒数进行*/
public void testFive() {
	Timer timer = new Timer();
	TimerTaskTest task = new TimerTaskTest();
	task.setName("JION.JION");
	//创建时间对象
	Calendar calendar = Calendar.getInstance();//单例模式
	//创建时间格式对象 
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
	//打印当前时间
	System.out.println("执行时间"+simpleDateFormat.format(calendar.getTime()));
	//执行任务的run方法,延时2000毫秒,进行执行,随后每5s执行
	timer.schedule(task,2000L,5000L);
}
```


#### **`scheduleAtFixedRate()`指定时间和间隔进行调度**

``` java
/**设定时间,随后指定时间进行调度,每隔毫秒数进行*/
public void testSix() {
	Timer timer = new Timer();
	TimerTaskTest task = new TimerTaskTest();
	task.setName("JION.JION");
	//创建时间对象
	Calendar calendar = Calendar.getInstance();//单例模式
	//创建时间格式对象 
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
	//打印当前时间
	System.out.println("执行时间"+simpleDateFormat.format(calendar.getTime()));
	//为当前时间加5秒
	calendar.add(Calendar.SECOND, 5);
	//执行任务的run方法,执行时间为当前时间后加5s,随后每2s执行
	timer.scheduleAtFixedRate(task, calendar.getTime(),2000L);
}
```

#### **`scheduleAtFixedRate()`指定时间和延时时间以及间隔进行调度**

``` java
/**设定时间,随后进行调度,每隔毫秒数进行*/
public void testSeven() {
	Timer timer = new Timer();
	TimerTaskTest task = new TimerTaskTest();
	task.setName("JION.JION");
	//创建时间对象
	Calendar calendar = Calendar.getInstance();//单例模式
	//创建时间格式对象 
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
	//打印当前时间
	System.out.println("执行时间"+simpleDateFormat.format(calendar.getTime()));
	//为当前时间加5秒
	calendar.add(Calendar.SECOND, 5);
	//执行任务的run方法,延时5s后执行,随后每2s执行
	timer.scheduleAtFixedRate(task, 5000L,2000L);
}
```


#### **测试:当执行时间早于当前或者大于间隔时间时,两者的区别**

- 任务执行时间为当前时间之前.
`schedule`:实际执行时间为当前时间,并为了赶上进度而在后面多次执行
`scheduleAtFixedRate`:实际执行为当前时间,并为了赶上进度而立即多次补上执行.

- 任务执行时间超过间隔时间
`schedule`:下一次的执行时间从本次实际的开始进行,因此任务不断调整实际执行时间
`scheduleAtFixedRate`:继续按照正常时间进行,有并发性

``` java
public void testEight() {
	Timer timer = new Timer();
	//创建时间对象
	Calendar calendar = Calendar.getInstance();//单例模式
	//创建时间格式对象 
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
	//打印当前时间
	System.out.println("执行时间"+simpleDateFormat.format(calendar.getTime()));
	//为当前时间5秒之前
	calendar.add(Calendar.SECOND, -5);
	//执行任务的run方法,当前时间后多少毫秒,随后间隔2000毫秒,进行执行
	timer.scheduleAtFixedRate(new TimerTask() {
//		timer.schedule(new TimerTask() {
		@Override
		public void run() {
			SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd : hh:mm:ss");
			System.out.println("匿名内部类正在进行....");
			try {
				Thread.sleep(5000);
			} catch (InterruptedException e) {
				e.printStackTrace();
				System.err.println("线程执行出现问题...");
			}
			System.out.println("最近执行时间"+simpleDateFormat.format(scheduledExecutionTime()));
		}
	}, calendar.getTime(),2000L);	
}
```



## `quartz` 类定时任务调度
`quartz` 具有强大的调度功能,配置方式多样,可以适用分布式和集群管理.
调度器,任务,触发器是构造调度框架的核心.
![调度组成][3]

其中任务`Job`和触发器`Trigger`结合,共同传入调度器`Scheduler`中,完成定时任务的创建.`Job`创建任务,`Trigger`由时间`SimpleTrigger`或者日历`CronTrigger`实现,两者供调度器`Scheduler`使用,最终创建一个对象.



### 示例代码

[GitHub地址](https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/timedTask/quartz)

- `CronTriggerTest.java`日历调度作业触发器
- `HelloJob`自定义任务实例
- `HelloScheduler` 任务调度器
- `SchedulerTest` 任务调度器测试
- `SimpleTriggerTest` 简单调度作业触发器



### `quartz`类说明

#### `Job`
每次在调度器在执行`Job`时,它在`execute()`方法前会创建一个新的job实例,当调用结束后,相关`Job`会被释放回收.
#### `Job Detail`
为`Job`实例提供了很多设置,以及JobDataMap成员变量,用来储存Job的实例状态信息,通过其添加具体的Job实例.

- name:名字
- group:组,参数为空时表示默认组
- jobClass:job的具体实现类
- jobDataMap:传参

#### `JobExecutionContext`
当` Scheduler`调用一个Job时,就会将`JobExecutionContext`传入`Job`的`execute()`方法
`Job`通过该实例可以访问到正在运行的环境信息和本身的参数明细.

#### `JobDataMap`
当进行任务调度时,使用`JobDataMap`存储可以序列化的数据,并放入`JobExecutionContext`中,便于获取.
- 可以通过Map中获取`JobDataMap`中的数据
- `Job`实现类中添加`JobDataMap`中的键的`setter()`方法

#### `Trigger`
触发器,用来确定`Job`触发事件.可以基于时间或者日历.

#### `JobKey`
表示`Job`实例的标志,触发器触发时,该标识指定的`Job`将会被执行.

#### `StartTime`
触发器的时间表首次触发的时间,类型为`Java.util.Date`.

#### `EndTime`
触发器不再执行的时间,类型为`Java.util.Date`.

#### `SimpleTrigger`
在一个指定时间段内执行一次作业任务或是指定的时间间隔内多次执行作业任务
- 重复次数必须为0,正整数
- 重复执行间隔必须为0或长整数
- 指定了`endTime`后会覆盖重复次数

#### `CronTrigger`
基于日历的作业调度器

#### ` Cron表达式`
配置`CronTrigger`实例,由7个表达式构成的字符串,且不区分大小写
`[秒] [分] [时] [日] [月] [周] [年]`
![Cron表达式][2]
![Cron表达式][3]

#### ` Scheduler`
调度器,由其工厂类创建,其可以通过读取配置文件传入初始参数完成工厂创建.

### 功能编码
#### 快速开始
**创建任务`Job`类**

 1. 继承`Job`类,重写其`execute()`方法
 2. `JobExecutionContext`参数包含了运行时`Quartz`的具体细节

``` java
/**
 *	定时任务模型
 *	Job类
 */
public class HelloJob implements Job {

	/**
	 * 	编写任务的执行逻辑
	 * 	并打印执行的时间
	 * 	@param JobExecutionContext 定时过程的上下文容器.可以通过其访问到Quartz运行时候的具体细节
	 * 	@exception JobExecutionException 运行失败后的抛出异常
	 */
	@SuppressWarnings("unused")
	@Override
	public void execute(JobExecutionContext context) throws JobExecutionException {
		//执行时间
		String now = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
		//执行打印任务
		System.out.println( now + ":任务执行了...." );
		//获取任务信息
		JobKey jobKey = context.getJobDetail().getKey();
		JobDataMap jobDataMap = context.getJobDetail().getJobDataMap();
		System.out.println("任务名:"+jobKey.getName() + "任务组:"+jobKey.getGroup() + "参数信息:"+jobDataMap.getString("message"));
		//获取触发器信息
		TriggerKey triggerKey = context.getTrigger().getKey();
		JobDataMap triggerDataMap = context.getTrigger().getJobDataMap();
		System.out.println("调度名"+triggerKey.getName() + "调度组"+triggerKey.getGroup() + "参数信息:"+ getMessage());			//使用get方法,完成参数的绑定
	}
	
	/*
	 * 	通过get/set方式获得传入参数 
	 */
	private String message;
	public String getMessage() {
		return message;
	}
	public void setMessage(String message) {
		this.message = message;
	}
}
```
**创建调度者`HelloScheduler`类**

``` java
public class HelloScheduler {

	public static void main(String[] args) throws Exception{
		//创建一个JobDetail实现,该实例与Job实现绑定.使用建造者模式
		JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)									//传入具体类类类型,通过反射描述创建实例
							.withIdentity("job_01","job_group_01")								//为任务分配任务名和任务组
							.usingJobData("message", "任务创建..")
								.build();
		System.out.println("任务明细的描述"+jobDetail.getDescription());
		System.out.println("任务明细的名字"+jobDetail.getKey().getName());
		System.out.println("任务明细的分组"+jobDetail.getKey().getGroup());
		System.out.println("任务明细的实现类"+jobDetail.getJobClass().getName());
		
		//日期时间,距离现5s后
		Date startTime = new Date();
		startTime.setTime(startTime.getTime()+5000);
		//日期时间,距离现25s后
		Date endTime = new Date();
		endTime.setTime(endTime.getTime()+25000);
		
		
		//创建一个Trigger实例,定义重复周期时间
		Trigger trigger = TriggerBuilder.newTrigger()											//创建计划实例
							.withIdentity("trigger_01", "trigger_group_01")						//分配周期名称和周期组
							.usingJobData("message","任务准备立刻开始..")
//								.startNow()														//立刻执行
								.startAt(startTime)												//开始时间 
								.endAt(endTime)													//结束时间
									.withSchedule(SimpleScheduleBuilder.simpleSchedule()		//SimpleSchedule模式进行简单间隔定义
											.withIntervalInSeconds(2)							//每隔两秒执行一次
												.repeatForever())								//重复执行
													.build();

		//创建Scheduler实例
		SchedulerFactory factory = new StdSchedulerFactory();									//工厂 模式,创建实例
		Scheduler scheduler = factory.getScheduler();											//创建实例
		scheduler.scheduleJob(jobDetail,trigger);												//绑定任务和定时周期
		scheduler.start();																		//开始执行
	}
}
```

#### 基于时间点的触发器

``` java
public class SimpleTriggerTest {

	/**	指定时间后,执行一次或者多次*/
	@Test
	public void testOne() {
		try {
			//创建一个JobDetail实现,该实例与Job实现绑定.使用建造者模式
			JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)									//传入具体类类类型,通过反射描述创建实例
								.withIdentity("job_01","job_group_01")								//为任务分配任务名和任务组
								.usingJobData("message", "任务创建..")
									.build();
			
			//日期时间,距离现5s后
			Date startTime = new Date();
			startTime.setTime(startTime.getTime()+5000);
			//日期时间,距离现25s后
			Date endTime = new Date();
			endTime.setTime(endTime.getTime()+25000);
			
			//创建一个Trigger实例,定义重复周期时间
			SimpleTrigger trigger = (SimpleTrigger) TriggerBuilder.newTrigger()						//创建计划实例
								.withIdentity("trigger_01", "trigger_group_01")						//分配周期名称和周期组
								.usingJobData("message","任务准备立刻开始..")
								.startAt(startTime)
								.endAt(endTime)
									.withSchedule(
											SimpleScheduleBuilder.simpleSchedule()					//SimpleSchedule模式进行简单间隔定义
											.withIntervalInSeconds(2)								//每隔两秒执行一次
												.withRepeatCount(SimpleTrigger.REPEAT_INDEFINITELY))//withRepeatCount定义重复的次数
													.build();

			//创建Scheduler实例
			SchedulerFactory factory = new StdSchedulerFactory();									//工厂 模式,创建实例
			Scheduler scheduler = factory.getScheduler();											//创建实例
			scheduler.scheduleJob(jobDetail,trigger);												//绑定任务和定时周期
			scheduler.start();		
			
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	//测试
	public static void main(String[] args) {
		new SimpleTriggerTest().testOne();
	}
}
```


#### 基于日历的触发器
`Cron`表达式举例
| 秒   | 分   | 时   | 日   | 月   | 周   | 年        | 含义                                                         |
| ---- | ---- | ---- | ---- | ---- | ---- | --------- | ------------------------------------------------------------ |
| *    | *    | *    | *    | *    | ?    | *         | 每秒都触发一次                                               |
| 0    | 15   | 10   | ?    | *    | ?    | *         | 每天10点15分触发                                             |
| 0    | 0/5  | 14   | *    | *    | ?    | *         | 每天下午的2点到2点59分,每隔5分钟触发一次.		0/5:从0分开始,每五分钟执行一次 |
| 0    | 15   | 10   | ?    | *    | 2-6  | *         | 每周一到周五10点15分触发                                     |
| 0    | 15   | 10   | ?    | *    | 6#3  | *         | 每月的第三周的星期五触发                                     |
| 0    | 15   | 10   | ?    | *    | 6L   | 2016-2017 | 2016年到2017年每月最后一周的星期五10点15分触发	6L:表示最后一周的周五 |

``` java
public class CronTriggerTest {
	/**	通过日历进行调用*/
	@Test
	public void testOne() {
		try {
			//创建一个JobDetail实现,该实例与Job实现绑定.使用建造者模式
			JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)									//传入具体类类类型,通过反射描述创建实例
								.withIdentity("job_01","job_group_01")								//为任务分配任务名和任务组
								.usingJobData("message", "任务创建..")
									.build();
			
			
			//创建一个Trigger实例,定义重复周期时间
			CronTriggerImpl trigger = (CronTriggerImpl) TriggerBuilder.newTrigger()						//创建计划实例
								.withIdentity("trigger_01", "trigger_group_01")						//分配周期名称和周期组
								.usingJobData("message","任务准备立刻开始..")
									.withSchedule(
											CronScheduleBuilder.cronSchedule("* * * * * ? *"))		
													.build();

			//创建Scheduler实例
			SchedulerFactory factory = new StdSchedulerFactory();									//工厂 模式,创建实例
			Scheduler scheduler = factory.getScheduler();											//创建实例
			scheduler.scheduleJob(jobDetail,trigger);												//绑定任务和定时周期
			scheduler.start();		
			
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		new CronTriggerTest().testOne();
	}
}
```

#### 自定义调度器属性创建


``` java
public class SchedulerTest {

	/**	调度者的测试类*/
	@Test
	public void testOne() {
		try {
			//创建一个JobDetail实现,该实例与Job实现绑定.使用建造者模式
			JobDetail jobDetail = JobBuilder.newJob(HelloJob.class)									//传入具体类类类型,通过反射描述创建实例
								.withIdentity("job_01","job_group_01")								//为任务分配任务名和任务组
								.usingJobData("message", "任务创建..")
									.build();
			
			
			//创建一个Trigger实例,定义重复周期时间
			CronTriggerImpl trigger = (CronTriggerImpl) TriggerBuilder.newTrigger()						//创建计划实例
								.withIdentity("trigger_01", "trigger_group_01")						//分配周期名称和周期组
								.usingJobData("message","任务准备立刻开始..")
									.withSchedule(
											CronScheduleBuilder.cronSchedule("0 * * * * ? *"))		//每分钟执行一次		
													.build();

			//创建Scheduler实例
			SchedulerFactory factory = new StdSchedulerFactory();									//工厂 模式,创建实例
			Scheduler scheduler = factory.getScheduler();											//创建实例
			Date nextTime = scheduler.scheduleJob(jobDetail,trigger);								//绑定任务和定时周期,返回最近一次执行时间
			scheduler.start();				//开始		
			System.out.println("下一次执行时间:" + new SimpleDateFormat("yyyy-MM-dd hh:mm:ss").format(nextTime));
			
			//线程休眠1分钟后
			Thread.sleep(60000);
			//挂起
			scheduler.standby();
			//重启开启
			scheduler.start();
			//关闭,等待最后一个执行完关闭后
			scheduler.shutdown(true);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		new SchedulerTest().testOne();
	}
}
```

属性文件`ente1`的配置

``` profile
#配置文件

# 调度器属性
org.quartz.scheduler.instanceName: DefaultQuartzScheduler				#区分	调度器
org.quartz.scheduler.instanceid:AUTO									#自动设置ID
org.quartz.scheduler.rmi.export: false									
org.quartz.scheduler.rmi.proxy: false
org.quartz.scheduler.wrapJobExecutionInUserTransaction: false

#线程池属性
org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool			#线程池的实现类
org.quartz.threadPool.threadCount: 10									#线程池数量[0]
org.quartz.threadPool.threadPriority: 5									#工作线程优先级[1-10]
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread: true

# 作业存储
org.quartz.jobStore.misfireThreshold: 60000								

# 插件配置
org.quartz.jobStore.class: org.quartz.simpl.RAMJobStore
```



[3]:https://raw.githubusercontent.com/jionjion/Picture_Space/master/WorkSpace/Java/javaBase/quartz-01.png
[4]:https://raw.githubusercontent.com/jionjion/Picture_Space/master/WorkSpace/Java/javaBase/quartz-02.png
[5]:https://raw.githubusercontent.com/jionjion/Picture_Space/master/WorkSpace/Java/javaBase/quartz-03.png







[1]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/timedTask/timer
[2]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/timedTask/quartz
