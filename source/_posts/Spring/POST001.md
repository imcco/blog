---
title: Spring定时任务之Quartz
tags: Quartz
category: Quartz
abbrlink: 6441
date: 2018-04-06 20:30:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0169.jpg)
Spring中的定时任务功能，最好的办法当然是使用Quartz来实现。我使用的是Maven来管理项目
<!--more-->

### 需要的Jar包  

`quartz-1.8.5.jar  commons-logging.jar  spring-core-3.0.5.RELEASE.jar  spring-beans-3.0.5.RELEASE.jar  spring-context-3.0.5.RELEASE.jar  spring-context-support-3.0.5.RELEASE.jar  spring-asm-3.0.5.RELEASE.jar  spring-expression-3.0.5.RELEASE.jar  spring.transaction-3.0.5.RELEASE.jar  spring-web-3.0.5.RELEASE.jar` 

### Maven的pom.xml的配置  

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>QtzTest</groupId>  
    <artifactId>QtzTest</artifactId>  
    <version>1.0</version>  
  
    <properties>  
        <springframework.version>3.0.5.RELEASE</springframework.version>  
    </properties>  
  
    <dependencies>  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-context</artifactId>  
            <version>${springframework.version}</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-context-support</artifactId>  
            <version>${springframework.version}</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-tx</artifactId>  
            <version>${springframework.version}</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-web</artifactId>  
            <version>${springframework.version}</version>  
        </dependency>  
  
        <dependency>  
            <groupId>org.quartz-scheduler</groupId>  
            <artifactId>quartz</artifactId>  
            <version>1.8.5</version>  
        </dependency>  
    </dependencies>  
  
    <build>  
        <finalName>${project.artifactId}</finalName>  
        <plugins>  
            <plugin>  
                <groupId>org.mortbay.jetty</groupId>  
                <artifactId>jetty-maven-plugin</artifactId>  
                <version>7.5.4.v20111024</version>  
                <configuration>  
                    <scanIntervalSeconds>10</scanIntervalSeconds>  
                    <webApp>  
                        <contextPath>/${project.artifactId}</contextPath>  
                    </webApp>  
                </configuration>  
            </plugin>  
        </plugins>  
    </build>  
</project>  
```

### 在Spring中使用Quartz有两种方式实现

第一种是任务类继承QuartzJobBean

第二种则是在配置文件里定义任务类和要执行的方法，类和方法仍然是普通类。

很显然，第二种方式远比第一种方式来的灵活。

#### 第一种方式的JAVA代码： 

```java
package com.ncs.hj;  
  
import org.quartz.JobExecutionContext;  
import org.quartz.JobExecutionException;  
import org.springframework.scheduling.quartz.QuartzJobBean;  
  
public class SpringQtz extends QuartzJobBean{  
    private static int counter = 0;  
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {  
        System.out.println();  
        long ms = System.currentTimeMillis();  
        System.out.println("\t\t" + new Date(ms));  
        System.out.println(ms);  
        System.out.println("(" + counter++ + ")");  
        String s = (String) context.getMergedJobDataMap().get("service");  
        System.out.println(s);  
        System.out.println();  
    }  
}  
```

#### 第二种方式的JAVA代码： 

```java
package com.ncs.hj;  
  
import org.quartz.JobExecutionContext;  
import org.quartz.JobExecutionException;  
import org.springframework.scheduling.quartz.QuartzJobBean;  
  
import java.util.Date;  
  
public class SpringQtz {  
    private static int counter = 0;  
    protected void execute()  {  
        long ms = System.currentTimeMillis();  
        System.out.println("\t\t" + new Date(ms));  
        System.out.println("(" + counter++ + ")");  
    }  
}  
```

### Spring的配置文件 

```xml
<!------------ 配置调度程序quartz ，其中配置JobDetail有两种方式-------------->    
    <!--方式一：使用JobDetailBean，任务类必须实现Job接口 -->     
    <bean id="myjob" class="org.springframework.scheduling.quartz.JobDetailBean">    
     <property name="name" value="exampleJob"></property>    
     <property name="jobClass" value="com.ncs.hj.SpringQtz"></property>   
     <property name="jobDataAsMap">  
<map>  
    <entry key="service"><value>simple is the beat</value></entry>  
</map>  
;/property>  
    </bean>   
    <!--运行时请将方式一注释掉！ -->    
    <!-- 方式二：使用MethodInvokingJobDetailFactoryBean，任务类可以不实现Job接口，通过targetMethod指定调用方法-->    
    <!-- 定义目标bean和bean中的方法 -->  
    <bean id="SpringQtzJob" class="com.ncs.hj.SpringQtz"/>  
    <bean id="SpringQtzJobMethod" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">  
    <property name="targetObject">  
        <ref bean="SpringQtzJob"/>  
    </property>  
    <property name="targetMethod">  <!-- 要执行的方法名称 -->  
        <value>execute</value>  
    </property>  
</bean>  
  
<!-- ======================== 调度触发器 ======================== -->  
<bean id="CronTriggerBean" class="org.springframework.scheduling.quartz.CronTriggerBean">  
    <property name="jobDetail" ref="SpringQtzJobMethod"></property>  
    <property name="cronExpression" value="0/5 * * * * ?"></property>  
</bean>  
  
<!-- ======================== 调度工厂 ======================== -->  
<bean id="SpringJobSchedulerFactoryBean" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">  
    <property name="triggers">  
        <list>  
            <ref bean="CronTriggerBean"/>  
        </list>  
    </property>  
</bean>
```

### 关于cronExpression表达式 

一个cron表达式由6或7个时间元素组成。它们之间用空格分隔，依次为：[秒][分] [小时][日] [月][星期] [年]

| 序号 | 说明 | 是否必填 | 允许填写的值     | 允许的符号    |
| ---- | ---- | -------- | ---------------- | ------------- |
| 1    | 秒   | 是       | 0－59            | , - * /       |
| 2    | 分   | 是       | 0－59            | , - * /       |
| 3    | 小时 | 是       | 0－23            | , - * /       |
| 4    | 日   | 是       | 1－31            | , - * ? / L W |
| 5    | 月   | 是       | 1－12 or JAN-DEC | , - * /       |
| 6    | 星期 | 是       | 1-7 or SUN-SAT   | , - * ? / L # |
| 7    | 年   | 否       | 1970-2099        | , - * /       |

> 其中每个元素值可以是一个确定值(6)，一个连续区间(9-12)，一个间隔时间(0/5)，一个列表(1，3，5)或通配符。

“-”表示可选值范围，如在“小时”上设置“10-12”，表示10点、11点和12点触发。 
“，”表示可选的多个值，例如在“星期”上设置“MON，WED，FRI”，表示周一，周三和周五触发。 
“/”用于递增触发，如在“秒”上面设置“5/15”表示从第5秒开始，每15秒触发一次(5，20，35，50)；在“日”上设置“1/3”表示每月1号开始，每三天触发一次。

*表示所有值. 如在“分”上设置“*”，表示每分钟触发。 
“？”字符仅出现在“日”和“星期”两个元素上，表示不指定值。当这两个元素之一被指定了值以后，为了避免冲突，需要将另一个元素的值设为“？”

“月”和“星期”元素上若使用英文字母是不区分大小写的，即MON与mon相同

“L” 字符仅出现在“日”和“星期”两个元素上，它是单词“last”的缩写。 
“L”在“日”元素上出现，表示每个月的最后一天；在“星期”元素上出现，表示每个月最后一个星期六。 
如果在“L”前有具体的内容，它就具有其他的含义了。例如：“6L”在“日”上出现，表示每月的倒数第６天；“5L”在“星期”上出现，表示每月的最后一个星期四

> 注意：在使用“L”参数时，不要指定列表或范围，因为这会导致问题

W表示离指定日期的最近那个工作日(周一至周五). 
例如在日字段上设置“15W”，表示离每月15号最近的那个工作日触发。 
如果15号正好是周六，则找最近的周五(14号)触发；如果15号是周未，则找最近的下周一(16号)触发；如果15号正好在工作日(周一至周五)，则就在该天触发。 
如果指定格式为“1W”，它则表示每月1号往后最近的工作日触发。 
如果1号正是周六，则将在3号下周一触发。(注，“W”前只能设置具体的数字，不允许区间“-”).

> 小提示：“L”和 “W”可以一组合使用。如果在“日”上设置“LW”，则表示在本月的最后一个工作日触发；

常用示例: 

- 0 0 12 * * ? 每天12点触发

- 0 15 10 ? * * 每天10点15分触发
- 0 15 10 * * ? 每天10点15分触发
- 0 15 10 * * ? * 每天10点15分触发
- 0 15 10 * * ? 2005 2005年每天10点15分触发
- 0 * 14 * * ? 每天14点到14点59分之间，每分钟触发一次
- 0 0/5 14 * * ? 每天14点到14点59分之间，每5分钟触发一次（从14点开始触发）
- 0 0/5 14，18 * * ? 每天14点到14点59分及18点到18点59分，每5分钟触发一次（分别从14点、18点开始触发）
- 0 0-5 14 * * ? 每天14点到14点05分之间，每分钟触发
- 0 10，44 14 ? 3 WED 3月份每周三14点10分和14点44分触发
- 0 15 10 ? * MON-FRI 周一到周五每天10点15分触发
- 0 15 10 15 * ? 每月15号10点15分触发
- 0 15 10 L * ? 每月最后一天的10点15分触发
- 0 15 10 ? * 6L 每月最后一个周五的10点15分触发
- 0 15 10 ? * 6L 2002-2005 从2002年到2005年每月一个周五的10点15分触发
- 0 15 10 ? * 6#3 每月第三个周五的10点15分触发
- 0 0 12 1/5 * ? 每月1号的12点开始触发，每隔5天触发一次

### web.xml里面配置Spring

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app xmlns="http://java.sun.com/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee  
          http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"  
         version="2.5">  
    <welcome-file-list>  
        <welcome-file>index.html</welcome-file>  
    </welcome-file-list>  
  
    <context-param>  
        <param-name>contextConfigLocation</param-name>  
        <param-value>/WEB-INF/spring-config.xml</param-value>  
    </context-param>  
  
    <listener>  
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener>  
</web-app>  
```

### 其他

#### 加载任务loadTask.java

```java
package com.imc.task;

import java.text.ParseException;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.UUID;

import org.apache.commons.lang.StringUtils;
import org.quartz.JobDataMap;
import org.quartz.Scheduler;
import org.quartz.SchedulerException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.CronTriggerFactoryBean;
import org.springframework.scheduling.quartz.JobDetailFactoryBean;

import com.imc.entity.assist.CmsTask;
import com.imc.manager.assist.CmsTaskMng;

/**
 * @author Tom
 */
public class LoadTask{
	/**
	 * 系统初始加载任务
	 */
	public void loadTask(){
		List<CmsTask>tasks=taskMng.getList();
		if(tasks.size()>0){
			for (int i = 0; i < tasks.size(); i++) {
				CmsTask task=tasks.get(i);
				//任务开启状态 执行任务调度
				if(task.getEnable()){
					try {
						JobDetailFactoryBean jobDetailFactoryBean = new JobDetailFactoryBean();
						//设置任务名称
						if(StringUtils.isNotBlank(task.getTaskCode())){
							jobDetailFactoryBean.setName(task.getTaskCode());
						}else{
							UUID uuid=UUID.randomUUID();
							jobDetailFactoryBean.setName(uuid.toString());
							task.setTaskCode(uuid.toString());
							taskMng.update(task, task.getAttr());
						}
						jobDetailFactoryBean.setJobClass(getClassByTask(task.getJobClass()));
						//任务需要参数attr属性 
						jobDetailFactoryBean.setJobDataMap(getJobDataMap(task.getAttr()));
						jobDetailFactoryBean.setGroup(Scheduler.DEFAULT_GROUP);
						jobDetailFactoryBean.afterPropertiesSet();
						
						CronTriggerFactoryBean cronTriggerFactoryBean=new CronTriggerFactoryBean();
						cronTriggerFactoryBean.setBeanName("cron_" + i);
						cronTriggerFactoryBean.setCronExpression(taskMng.getCronExpressionFromDB(task.getId()));
						cronTriggerFactoryBean.setGroup(Scheduler.DEFAULT_GROUP);
						cronTriggerFactoryBean.setName("cron_" + i);
						cronTriggerFactoryBean.setDescription("");
						cronTriggerFactoryBean.afterPropertiesSet();
						//调度任务
						scheduler.scheduleJob(jobDetailFactoryBean.getObject(), cronTriggerFactoryBean.getObject()); 
					} catch (SchedulerException e) {
						e.printStackTrace();
					} catch (ClassNotFoundException e) {
						e.printStackTrace();
					} catch (ParseException e) {
						e.printStackTrace();
					}
				}
			}
		}
	}
	
	/**
	 * 
	 * @param params 任务参数
	 * @return
	 */
	private JobDataMap getJobDataMap(Map<String,String> params){
		JobDataMap jdm=new JobDataMap();
		Set<String>keySet=params.keySet();
		Iterator<String>it=keySet.iterator();
		while(it.hasNext()){
			String key=it.next();
			jdm.put(key, params.get(key));
		}
		return jdm;
	}
	
	/**
	 * 
	 * @param taskClassName 任务执行类名
	 * @return
	 * @throws ClassNotFoundException
	 */
	private Class<?> getClassByTask(String taskClassName) throws ClassNotFoundException{
		return Class.forName(taskClassName);
	}
	@Autowired
	private CmsTaskMng taskMng;
	@Autowired
	private Scheduler scheduler;
}
```

#### 用户同步任务

```java
import java.io.File;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;

import org.apache.commons.lang.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

import imc.c6.api.sysuser.SysUserAPI;
import imc.c6.api.sysuser.SysUserRoleAPI;
import imc.c6.api.sysuser.dto.SysDept;
import imc.c6.api.sysuser.dto.SysUser;

import com.imc.common.util.PropertyUtils;
import com.imc.common.web.springmvc.RealPathResolver;
import com.imc.core.entity.CmsGroup;
import com.imc.core.entity.CmsUser;
import com.imc.core.entity.CmsUserExt;
import com.imc.core.manager.CmsUserExtMng;
import com.imc.core.manager.CmsUserMng;
import com.imc.core.manager.UnifiedUserMng;

import java.util.Map;
public class UserDayJob{
	private static final Logger log = LoggerFactory.getLogger(UserDayJob.class);
	
	public void execute() {
		String a =PropertyUtils.getPropertyValue(
				new File(realPathResolver.get(com.imc.cms.Constants.imc_CONFIG)),"cms.usetype");
		String usetype=propertyUtils.getPropertiesString("cms.usetype");	
		if ((usetype.equals("3"))||(usetype.equals("2"))){			
		manager.getMember();
		manager.getAdmin();	
		}
	}
	
	@Autowired
	private CmsUserMng manager;
	@Autowired
	private CmsUserExtMng extManager;	
	@Autowired
	protected PropertyUtils propertyUtils;
	@Autowired
	private RealPathResolver realPathResolver;
}
```

#### 配置文件quartz-task.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd"
	default-lazy-init="false">
	
	<bean id="executor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor"> 
		<property name="corePoolSize" value="10" /> 
		<property name="maxPoolSize" value="100" /> 
		<property name="queueCapacity" value="500" />
	</bean>
	<!--每日任务(内容相关) -->
	<bean id="contentDayJob" class="com.imc.task.job.ContentDayJob"></bean>
	<bean id="contentDayJobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
		<property name="targetObject" ref="contentDayJob" />
		<property name="targetMethod" value="execute" />
	</bean>
	<bean id="contentDayJobTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
		<property name="jobDetail" ref="contentDayJobDetail" />
		<property name="cronExpression" value="0 0 0 * * ?" />
	</bean>
	<bean id="siteDayJob" class="com.imc.task.job.SiteDayJob"></bean>
	<bean id="siteDayJobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
		<property name="targetObject" ref="siteDayJob" />
		<property name="targetMethod" value="execute" />
	</bean>
	<bean id="siteDayJobTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
		<property name="jobDetail" ref="siteDayJobDetail" />
		<property name="cronExpression" value="0 0 0 * * ?" />
	</bean>
	<!-- 用户同步任务 -->
	<bean id="userDayJob" class="com.imc.task.job.UserDayJob"></bean>
	<bean id="userDayJobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
		<property name="targetObject" ref="userDayJob" />
		<property name="targetMethod" value="execute" />
	</bean>
	<bean id="userDayJobTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
		<property name="jobDetail" ref="userDayJobDetail" />
		<property name="cronExpression" value="*/5 * * * * ?" />
	</bean>
	<!-- 调度器 -->
    <bean name="scheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean"> 
       <!-- 通过applicationContextSchedulerContextKey属性配置spring上下文 -->    
        <property name="applicationContextSchedulerContextKey">    
            <value>applicationContext</value>    
        </property>   
        <property name="triggers">  
			<list>   
				<ref bean="contentDayJobTrigger" /> 
				<ref bean="siteDayJobTrigger" /> 
				<ref bean="userDayJobTrigger" />    
			</list> 
		</property> 
    	<property name="taskExecutor" ref="executor" /> 
   	</bean> 
    <!--加载数据库任务-->
    <bean id="loadTask" class="com.imc.task.LoadTask" init-method="loadTask" />
    
</beans>
```
