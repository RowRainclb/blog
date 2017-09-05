---
title: springboot中集成quartz任务调度
date: 2017-05-15 17:12:21
tags:
    - java
    - quartz
    - springboot
---
# quartz的使用
# 介绍
  Quartz是一个完全由Java编写的开源作业调度框架，为在Java应用程序中进行作业调度提供了简单却强大的机制。Quartz允许开发人员根据时间间隔来调度作业。它实现了作业和触发器的多对多的关系，还能把多个作业与不同的触发器关联。

# 使用
   最近项目需求需要用户设定自动执行的定时任务，因为用的springboot框架，所以结合springboot,
进行quartz在项目中的使用
   所需依赖：
``` javascript
<!-- 要用最新版本，4.2.7.RELEASE -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>4.2.7.RELEASE</version>
		</dependency>
		<!-- tools -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<optional>true</optional>
		</dependency>
		<!-- quartz  版本用：2.2.1以上-->
		<dependency>
			<groupId>org.quartz-scheduler</groupId>
			<artifactId>quartz</artifactId>
			<version>2.2.1</version>
		</dependency>
		<!-- 该依赖必加，里面有sping对schedule的支持 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>4.2.7.RELEASE</version>
		</dependency>
```
在Spring中使用Quartz有两种方式实现：第一种是任务类继承QuartzJobBean，第二种则是在配置文件里定义任务类和要执行的方法，类和方法可以是普通类。很显然，第二种方式远比第一种方式来的灵活。

这里采用的就是第二种方式。

``` javascript
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:task="http://www.springframework.org/schema/task"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-3.0.xsd">
	
	<!-- 使用MethodInvokingJobDetailFactoryBean，任务类可以不实现Job接口，通过targetMethod指定调用方法-->
	<bean id="taskJob" class="com.example.demo.utils.TestTask"/>
	<bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    	<property name="group" value="job_work"/>
    	<property name="name" value="job_work_name"/>
    	<!--false表示等上一个任务执行完后再开启新的任务-->
    	<property name="concurrent" value="false"/>
    	<property name="targetObject">
        	<ref bean="taskJob"/>
   	 	</property>
    	<property name="targetMethod">
        	<value>run</value>
    	</property>
	</bean>
	<!--  调度触发器 -->
	<bean id="myTrigger" lazy-init="false"
      	class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    	<property name="name" value="work_default_name"/>
    	<property name="group" value="work_default"/>
    	<property name="jobDetail">
        	<ref bean="jobDetail" />
    	</property>
    	<property name="cronExpression">
        	<value>0 0 10 * * ?</value>
    	</property>
	</bean>
	<!-- 调度工厂 -->
	<bean id="scheduler" lazy-init="false" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    	<property name="triggers">
        	<list>
            	<ref bean="myTrigger"/>
        	</list>
    	</property>
	</bean>
	
	<bean id="springContextUtil" class="com.example.demo.utils.MyApplicationContextUtil"/>

</beans>

```

TestTask类则是一个普通的Java类，没有继承任何类和实现任何接口(当然可以用注解方式来声明bean)：


``` javascript
package com.example.demo.utils;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * Created by clb on 17-7-14.
 */

@Configurable
@EnableScheduling
public class TestTask {
	@Autowired
	ScheduleJobService scheduleJobService;
	/**
	 * 检查任务启动情况
	 */
	public void run() {
		System.out.println("测试任务线程开始执行");
	}
}

```
读取配置类
``` javascript
package com.example.demo.utils;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * <p>Note: 获取上下文的bean
 */

@Component
public class MyApplicationContextUtil implements ApplicationContextAware {
	
	// 声明一个静态变量保存   
	private static ApplicationContext applicationContext;

	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		// TODO Auto-generated method stub
		MyApplicationContextUtil.applicationContext=applicationContext;   
	}
	
	public static ApplicationContext getContext(){
		return applicationContext;
	}  
	
	@SuppressWarnings("unchecked")
    public static <T> T getBean(String name) throws BeansException {
               return (T) applicationContext.getBean(name);
     }

}
```

在Application添加配置
``` javascript

	@Configuration
	@ImportResource("classpath:/spring/applicationContext.xml")
	static class XmlImportingConfiguration {
	}
```
项目启动自运行类
``` javascript
package com.example.demo;

/**
 * Created by clb on 17-7-17.
 */

import com.example.demo.utils.ScheduleJobService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

/**
 * 服务启动执行
 */
@Component
public class MyStartupRunner1 implements CommandLineRunner {
	@Autowired
	ScheduleJobService scheduleJobService;
	@Override
	public void run(String... args) throws Exception {
		System.out.println(">>>>>>>>>>>>>>>服务启动执行，执行加载数据等操作<<<<<<<<<<<<<");
		init();
	}

	/**
	 * 在项目启动时运行以下代码
	 * @throws Exception
	 */
	public void init() throws Exception {
		scheduleJobService.getAllSchedules();
	}
}

```
任务操作类
``` javascript
package com.example.demo.utils;

import org.quartz.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by clb on 17-7-17.
 */
@Service
public class ScheduleJobService {
	@Autowired
	SchedulerFactoryBean schedulerFactoryBean;
	private static String JOB_GROUP_NAME = "JOBGROUP_NAME";
	private static String TRIGGER_GROUP_NAME = "EXTJWEB_TRIGGERGROUP_NAME";



	public void getAllSchedules() throws Exception {
		// 这里从数据库中获取任务信息数据
		List<ScheduleJob> jobList = new ArrayList<>();
		ScheduleJob scheduleJob = new ScheduleJob();
		String cron = String.format("0 0/%s * * * ? *",1);
		scheduleJob.setJobDesc(String.format("task(id:%s) is running","taskId"));
		scheduleJob.setJobName(String.valueOf(11));
		scheduleJob.setJobGroup("test_group");
		scheduleJob.setCronExpression(cron);
		jobList.add(scheduleJob);
		for (ScheduleJob job : jobList) {
			addScheduleJob(job);
		}
	}
	/**
	 * @Description: 添加一个定时任务
	 * @param job
	 * @throws SchedulerException
	 */

	public void addScheduleJob(ScheduleJob job) throws SchedulerException {
		Scheduler scheduler = schedulerFactoryBean.getScheduler();
		TriggerKey triggerKey = TriggerKey.triggerKey(job.getJobName(), job.getJobGroup());
		//获取trigger，即在spring配置文件中定义的 bean id="myTrigger"
		CronTrigger trigger = (CronTrigger) scheduler.getTrigger(triggerKey);
		//不存在，创建一个
		if (null == trigger) {
			JobDetail jobDetail = JobBuilder.newJob(QuartzJobFactory.class)
					.withIdentity(job.getJobName(), job.getJobGroup()).build();
			jobDetail.getJobDataMap().put("scheduleJob", job);
			//表达式调度构建器
			CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(job
					.getCronExpression()).withMisfireHandlingInstructionFireAndProceed();
			//按新的cronExpression表达式构建一个新的trigger
			trigger = TriggerBuilder.newTrigger().startNow().withIdentity(job.getJobName(), job.getJobGroup()).withSchedule(scheduleBuilder).build();
			scheduler.scheduleJob(jobDetail, trigger);
		} else {
			// Trigger已存在，那么更新相应的定时设置
			//表达式调度构建器
			CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(job
					.getCronExpression()).withMisfireHandlingInstructionFireAndProceed();
			//按新的cronExpression表达式重新构建trigger
			trigger = trigger.getTriggerBuilder().startNow().withIdentity(triggerKey)
					.withSchedule(scheduleBuilder).build();
			//按新的trigger重新设置job执行
			scheduler.rescheduleJob(triggerKey, trigger);
		}
		System.out.println("触发器执行完毕！");
		// 启动
		if (!scheduler.isShutdown()) {
			scheduler.start();
		}
	}
	/**
	 * @Description: 移除一个任务
	 * @param job
	 * @throws SchedulerException
	 */
	public void removeJob(ScheduleJob job) throws SchedulerException {
		Scheduler sched = schedulerFactoryBean.getScheduler();
		TriggerKey triggerKey = TriggerKey.triggerKey(job.getJobName(), job.getJobGroup());
		sched.pauseTrigger(triggerKey);// 停止触发器
		sched.unscheduleJob(triggerKey);// 移除触发器
		JobKey jobKey = new JobKey(job.getJobName(),job.getJobGroup());
		sched.deleteJob(jobKey );// 删除任务
	}
}

```
作业调度执行类
``` javascript
package com.example.demo.utils;

import com.example.demo.service.TaskService;
import org.quartz.*;
import org.springframework.stereotype.Component;

/**
 *
 * @Description: 若一个方法一次执行不完下次轮转时则等待改方法执行完后才执行下一次操作
 */
@Component
@DisallowConcurrentExecution
public class QuartzJobFactory implements Job {
	@Override
	public void execute(JobExecutionContext context) throws JobExecutionException {
		System.out.println("任务成功运行");
		ScheduleJob scheduleJob = (ScheduleJob)context.getMergedJobDataMap().get("scheduleJob");
		System.out.println("任务名称 = [" + scheduleJob.getJobName() + "]");
		TaskService taskService = MyApplicationContextUtil.getBean("taskService");
		try {
			taskService.run(Long.parseLong(scheduleJob.getJobName()));
		} catch (Exception e) {
//			log.error("定时任务运行出错！",e);
			System.out.println("定时任务运行出错"+e.getMessage());
			ScheduleJobService scheduleJobService = MyApplicationContextUtil.getBean("scheduleJobService");
			try {
//				log.debug("删除定时任务");
				System.out.println("删除定时任务");
				scheduleJobService.removeJob(scheduleJob);
				System.out.println("删除数据库任务记录");
				//删除数据库中记录
//				TaskQuartzMapper taskQuartzMapper= MyApplicationContextUtil.getBean("taskQuartzMapper");
//				taskQuartzMapper.delete(scheduleJob);
			} catch (SchedulerException e1) {
//				log.error("删除定时任务出错！",e);
				System.out.println("删除定时任务出错！"+e.getMessage());
			}
		}
	}
}

```

实体类
``` javascript
package com.example.demo.utils;

import lombok.Data;

import javax.persistence.Table;

@Data
@Table(name = "task_quartz")
public class ScheduleJob {
	private Long id;
	private String jobId;
	private String jobName;
	private String jobGroup;
	private String jobDesc;
	private String jobStatus;
	private String cronExpression;
}

```

启动程序，效果
``` javascript
2017-09-03 12:58:26.555  INFO 4040 --- [  restartedMain] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 2147483647
2017-09-03 12:58:26.556  INFO 4040 --- [  restartedMain] o.s.s.quartz.SchedulerFactoryBean        : Starting Quartz Scheduler now
2017-09-03 12:58:26.556  INFO 4040 --- [  restartedMain] org.quartz.core.QuartzScheduler          : Scheduler scheduler_$_NON_CLUSTERED started.
2017-09-03 12:58:26.626  INFO 4040 --- [  restartedMain] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
>>>>>>>>>>>>>>>服务启动执行，执行加载数据等操作<<<<<<<<<<<<<
触发器执行完毕！
2017-09-03 12:58:26.631  INFO 4040 --- [  restartedMain] org.quartz.core.QuartzScheduler          : Scheduler scheduler_$_NON_CLUSTERED started.
2017-09-03 12:58:26.632  INFO 4040 --- [  restartedMain] com.example.demo.QuartzApplication       : Started QuartzApplication in 4.449 seconds (JVM running for 4.988)
任务成功运行
任务名称 = [11]
进入定时任务－－－
------
本次任务执行完毕－－－
任务成功运行
任务名称 = [11]
进入定时任务－－－
------

```

源码地址：
https://github.com/RowRainclb/springboot_quartz




