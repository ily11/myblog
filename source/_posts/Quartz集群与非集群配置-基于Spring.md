---
title: Quartz集群与非集群配置--基于Spring
date: 2018-01-29 15:17:35
tags: Quartz
categories: spring
---

### 需求描述
有三个任务，jobA、jobB、jobC需要部署在两台服务器A、B上，
* 其中jobA操作数据库，所以jobA一次只能在一台服务器上运行，不能在服务器A、B上同时运行；
* jobB虽然不操作数据库，但也有相同需求，就是不能同时在两台服务器上运行；
* jobC可以同时在两台服务器上运行；

则jobA和jobB需要用到Quartz集群配置，jobC需要非集群配置。

Quartz的集群是依靠数据库实现的，是依靠数据库来监控到底是哪个quartz去执行了这个任务，然后让别的quartz不再执行这个任务。

我们需要先去[Quartz官网](http://www.quartz-scheduler.org/downloads/)下载相应版本的Quartz，然后解压，在/docs/dbTables文件夹下，会有各个数据库版本的sql文件，我们用的是mySQL数据库，所以我选择的是tables_mysql_innodb.sql文件：

![](/images/mysql1.png)

新建数据库quartz，然后运行tables_mysql_innodb.sql文件，会创建11张表，Quartz就是靠这11张表来实现集群的。

![](/images/mysql2.png)

接下来就是创建Gradle项目，实现Quartz集群配置。
### 创建Gradle项目
首先新建Gradle项目：

![](/images/new1.png)

然后填写GroupId和ArtifactId：

![](/images/new2.png)

下一步需要将前两项勾选上，这样IDEA会帮我们创建相应的项目文件夹：

![](/images/new3.png)

这样一个空的项目就搭建起来了。

### 配置build.gradle

要想使用Quartz需要先添加相应依赖，我们实现的Quartz是要结合Spring的，所以也要引进Spring的相应依赖：
```
dependencies {
    def springframeworkVersion = '4.3.7.RELEASE'
    def springSecurityVersion = '4.2.1.RELEASE'
    def typeSafeVersion = '1.3.1'
    def logbackVersion = '1.2.1'
    def groovyVersion = '2.4.9'

    // The production code uses the SLF4J logging API at compile time
    compile 'org.slf4j:slf4j-api:1.7.13'
    compile "com.typesafe:config:${typeSafeVersion}"
    compile "ch.qos.logback:logback-core:${logbackVersion}"
    compile "ch.qos.logback:logback-access:${logbackVersion}"
    compile "ch.qos.logback:logback-classic:${logbackVersion}"
    compile "org.codehaus.groovy:groovy-all:${groovyVersion}"
    //spring4
    compile "org.springframework:spring-test:${springframeworkVersion}"
    compile "org.springframework:spring-context:${springframeworkVersion}"
    compile "org.springframework:spring-context-support:${springframeworkVersion}"
    compile "org.springframework:spring-tx:${springframeworkVersion}"
    compile "org.springframework:spring-webmvc:${springframeworkVersion}"
    compile "org.springframework:spring-jdbc:${springframeworkVersion}"
    compile "org.springframework.security:spring-security-taglibs:${springSecurityVersion}"
    compile "org.springframework.security:spring-security-config:${springSecurityVersion}"
    compile "org.springframework.security:spring-security-cas:${springSecurityVersion}"
    //mysql connect
    compile group: 'mysql', name: 'mysql-connector-java', version: '6.0.6'
    compile "org.quartz-scheduler:quartz:2.2.3"
    testCompile 'junit:junit:4.12'
    testCompile group: 'javax.servlet', name: 'javax.servlet-api', version: '3.1.0'
}
```
### quartz集群与非集群配置
在同一个项目同时实现集群和非集群，就需要写两个调度配置，然后配置调度器，设置特定时间启动，然后再配置相应的job。具体配置如下：

#### 非集群配置
```
<!-- 非集群配置调度器\触发器\job -->
    <bean name="quartzScheduler2" lazy-init="false" autowire="no"
          class="org.springframework.scheduling.quartz.SchedulerFactoryBean ">
        <property name="applicationContextSchedulerContextKey" value="applicationContextKey"/>
        <property name="triggers">
            <list>
                <ref bean="jobCTaskTrigger"/>
            </list>
        </property>
    </bean>
```
将使用非集群的任务调度器放在`<property name="triggers">`的`<list>`里，然后再配置触发器和相应的job：

![](/images/quartz1.png)

首先触发器的`cronExpression`定义job的触发时间，`jobDetail`是job的实例，JobDetail是在Job被添加到Scheduler时由应用程序创建的，它包含了关于Job的各种属性信息。当触发器激活时，与他关联的JobDetail 被加载，然后就会找到本例中的jobC类的doJob方法并执行。

#### 集群配置

本例中是将Quartz的一些集群配置直接整合到xml文件里了，也可以在quartz.properties里配置。
```
<!-- 配置调度器\触发器\job -->
    <bean id="clusterQuartzScheduler" name="quartzScheduler" lazy-init="false" autowire="no" class="org.springframework.scheduling.quartz.SchedulerFactoryBean ">
        <property name="dataSource" ref="dataSource"/>
        <!--  quartz配置文件路径-->
        <!--<property name="configLocation" value="classpath:quartz.properties" />-->

        <property name="quartzProperties">
            <props>
                <prop key="org.quartz.scheduler.instanceName">TaskManager_Scheduler</prop>
                <prop key="org.quartz.scheduler.instanceId">AUTO</prop>
                <!-- 线程池配置 -->
                <prop key="org.quartz.threadPool.class">org.quartz.simpl.SimpleThreadPool</prop>
                <prop key="org.quartz.threadPool.threadCount">20</prop>
                <prop key="org.quartz.threadPool.threadPriority">5</prop>
                <!-- JobStore 配置 1.如下使用数据库 2.使用内存 org.quartz.simpl.RAMJobStore -->
                <prop key="org.quartz.jobStore.class">org.quartz.impl.jdbcjobstore.JobStoreTX</prop>

                <!-- 集群配置 -->
                <!--通知Scheduler实例要它参与到一个集群当中-->
                <prop key="org.quartz.jobStore.isClustered">true</prop>
                <!--定义了Scheduler实例检入到数据库中的频率(单位：毫秒)-->
                <prop key="org.quartz.jobStore.clusterCheckinInterval">15000</prop>
                <prop key="org.quartz.jobStore.maxMisfiresToHandleAtATime">1</prop>
                <prop key="org.quartz.jobStore.misfireThreshold">120000</prop>
                <prop key="org.quartz.jobStore.tablePrefix">QRTZ_</prop>
            </props>
        </property>
        <property name="schedulerName" value="TaskManagerScheduler"/>

        <!--必须的，QuartzScheduler 延时启动，应用启动完后 QuartzScheduler 再启动 -->
        <property name="startupDelay" value="3"/>

        <!--可选，QuartzScheduler 启动时更新己存在的Job，这样就不用每次修改targetObject后删除qrtz_job_details表对应记录了 -->
        <property name="overwriteExistingJobs" value="true"/>

        <!-- 设置自动启动 -->
        <property name="autoStartup" value="true"/>

        <!--具体Trigger配置-->
        <property name="applicationContextSchedulerContextKey" value="applicationContextKey"/>
        <property name="triggers">
            <list>
                <ref bean="jobAATaskTrigger"/>
                <ref bean="jobBBTaskTrigger"/>
                <!--<ref bean="jobCTaskTrigger"/>-->
            </list>
        </property>
    </bean>
```
![](/images/quartz2.png)

因为我们是用数据库来实现集群的，所以需要将“org.quartz.jobStore.class”值设置为“org.quartz.impl.jdbcjobstore.JobStoreTX”，如果想使用内存存储，只需要将其值设置为“org.quartz.simpl.RAMJobStore”即可。但是内存存储无法实现持久化。

“org.quartz.jobStore.isClustered”的值设置为“true”，即代表该Scheduler下的job参与到一个集群当中。

![](/images/quartz3.png)

触发器的配置跟非集群的配置相同，配置定时时间和job实例。

![](/images/quartz4.png)

在Spring中使用Quartz有两种方式实现：
* 利用JobDetailBean包装QuartzJobBean子类（即Job类）的实例。
* 利用MethodInvokingJobDetailFactoryBean工厂Bean包装普通的Java对象（即Job类），是在配置文件里定义任务类和要执行的方法，类和方法仍然是普通类。

具体说明：

1、采用第一种方法必须要继承QuartzJobBean，实现 executeInternal(JobExecutionContext jobexecutioncontext)方法，此方法就是被调度任务的执行体，然后将此Job类的实例直接配置到JobDetailBean中即可。

2、采用第二种方法 创建Job类，无须继承父类，直接配置MethodInvokingJobDetailFactoryBean即可。但需要指定一下两个属性：

* targetObject：指定包含任务执行体的Bean实例。

* targetMethod:指定将指定Bean实例的该方法包装成任务的执行体。

由于集群需要把定时任务的信息写入表，需要序列化，但MethodInvokingJobDetailFactoryBean 不能序列化，会报错。所以我们采用第一种方式。

首先重写JobDetailBean类：

```
package com.bupt.common;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.scheduling.quartz.QuartzJobBean;

import java.lang.reflect.Method;

public class JobDetailBean extends QuartzJobBean {


    private Logger logger = LoggerFactory.getLogger(JobDetailBean.class);

    private String targetObject;
    private String targetMethod;
    private static ApplicationContext ctx;

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {

        try {
            Object ctxBean = ctx.getBean(this.targetObject);
            try {
                Method m = ctxBean.getClass().getMethod(targetMethod);
                m.invoke(ctxBean);
            } catch (SecurityException e) {
                logger.error(e.toString());
                e.printStackTrace();
            } catch (NoSuchMethodException e) {
                logger.error(e.toString());
                e.printStackTrace();
            }
        } catch (Exception e) {
            throw new JobExecutionException(e);
        }
    }

    public static void setApplicationContext(ApplicationContext applicationContext) {
        ctx = applicationContext;
    }

    public void setTargetObject(String targetObject) {
        this.targetObject = targetObject;
    }

    public void setTargetMethod(String targetMethod) {
        this.targetMethod = targetMethod;
    }
}
```

所以在配置job的时候，需要指定jobClass，指向重写的类。

### 主函数

```
public class MainProgram {
    final static Logger logger = LoggerFactory.getLogger(MainProgram.class);
    public static void main(String[] args){
        try{
            System.out.println("开始执行任务。。。。。");
            AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext("com.bupt.config");
            JobDetailBean.setApplicationContext(ctx);
        }catch (Exception e){
            logger.error(e.getMessage());
        }
    }
}
```

首先指定配置文件，然后在引入到JobDetailBean里，这样程序就会读取配置文件，执行调度器，触发触发器，加载关联的JobDetail。

### job类
#### jobAA
```
@Service("jobAA")
public class jobAA {
    public void doJob() throws Exception{
        //设置日期格式
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        //1.加载驱动程序
        Class.forName("com.mysql.jdbc.Driver");
        String url = "jdbc:mysql://127.0.0.1:3306/quartz?characterEncoding=utf-8&autoReconnect=true&autoReconnectForPools=true&useSSL=false";
        String name = "root";
        String password = "123456";
        //2.获得数据库的连接
        Connection conn = DriverManager.getConnection(url, name, password);
        //3.通过数据库的连接操作数据库，实现增删改查
        Statement stmt = conn.createStatement();
        Boolean result = stmt.execute("UPDATE quartz.Student SET age = '8' WHERE id=2;");
        ResultSet age = stmt.executeQuery("SELECT age FROM quartz.Student WHERE id=2;");
        if(!result){
            System.out.println(df.format(new Date()) + "  执行job1AAAA任务。。。。。");
            while (age.next()){
                System.out.println(df.format(new Date()) + "  job1A任务。。。。。" +age.getString("age"));
            }
//            System.out.println(df.format(new Date()) + "  job1A任务。。。。。" +age.getString("age"));
        }
    }
}
```
#### jobBB
```
@Service("jobBB")
public class jobBB {
    public void doJob() throws Exception{
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");//设置日期格式
        System.out.println(df.format(new Date()) + "  执行jobBBBB任务。。。。。");
    }
}
```
#### jobC
```
@Service("jobC")
public class jobC {
    public void doJob() throws Exception{
        //设置日期格式
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(df.format(new Date()) + "  执行jobCCCC任务。。。。。");
    }
}
```
### 运行结果

再新建一个项目，项目结构和代码跟上面讲的一模一样，然后运行，根据输出结果会发现，jobAA和jobBB每次只会在一台机器上运行，而jobC两台机器同时运行。

![](/images/result1.png) ![](/images/result2.png)

![](/images/result3.png)

从上图也可以看出，当我停掉一台机器时，另一台会马上把jobAA和jobBB的任务结果来继续执行。

完整代码在我的github上[https://github.com/ily11/quartzCluster](https://github.com/ily11/quartzCluster)
