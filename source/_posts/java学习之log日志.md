---
title: java学习之logback
date: 2017-11-21 16:57:05
tags: logback
categories: java
---

### 简介
LogBack是由log4j的创始人开发的一个日志组件，用于替代log4j。LogBack的架构设计足够通用，可适用于不同的环境，目前LogBack分为三个模：lobback-core，logback-classic和logback-access。

core模块是其它两个模块的基础，classic是core的扩展，是log4j巨大改进的版本。LogBack-classic本身实现了SL4J的API，因此可以很容易的在logback与其它日志系统之间转换，例如log4j、JDK1.4中的java.util.logging（JUL）。第三个模块access，它集成了Servlet容器，提供了通过HTTP访问日志的功能。

在项目里添加依赖（gradle）：
```
dependencies {
    def logbackVersion = '1.2.1'
    // The production code uses the SLF4J logging API at compile time
    compile 'org.slf4j:slf4j-api:1.7.13'
    compile "ch.qos.logback:logback-core:${logbackVersion}"
    compile "ch.qos.logback:logback-access:${logbackVersion}"
    compile "ch.qos.logback:logback-classic:${logbackVersion}"
}
```

### Logback的核心对象：Logger、Appender、Layout
Logback主要建立于Logger、Appender 和 Layout 这三个类之上：
* Logger：日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别。Logger对象一般多定义为静态变量：
`
public  static final Logger logger = LoggerFactory.getLogger(loggerName);
`
* Appender:用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、 PostreSQL、Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。一般在配置的时候用到。
* Layout:负责把事件转换成字符串，格式化的日志信息的输出。


### Level 有效级别
LogBack的日志级别有trace、debug、info、warn、error，级别依次递增，定义于ch.qos.logback.classic.Level类。程序会打印高于或等于所设置级别的日志，设置的日志等级越高，打印出来的日志就越少。如果设置级别为INFO，则优先级高于等于INFO级别（如：INFO、 WARN、ERROR）的日志信息将可以被输出,小于该级别的如DEBUG将不会被输出。默认情况下，根logger级别是DEBUG。
### 过滤器
过滤器的返回值只能是ACCEPT、DENY和NEUTRAL的其中一个：
* 如果返回DENY，那么记录事件立即被抛弃，不再经过剩余过滤器；
* 如果返回NEUTRAL，那么有序列表里的下一个过滤器会接着处理记录事件；
* 如果返回ACCEPT，那么记录事件被立即处理，不再经过剩余过滤器。

### 配置
LogBack可以通过编程式或以XML、Groovy格式配置，LogBack读取配置或属性文件的步骤：

1.logback首先会试着查找logback.groovy文件;

2.当没有找到时，继续试着查找logback-test.xml文件;

3.当没有找到时，继续试着查找logback.xml文件;

4.如果仍然没有找到，则使用默认配置（打印到控制台）。

本人项目的配置是在logback.groovy里的，示例如下：
```
import ch.qos.logback.core.filter.EvaluatorFilter
import ch.qos.logback.classic.boolex.GEventEvaluator
import ch.qos.logback.core.util.FileSize
import static ch.qos.logback.classic.Level.INFO
import static ch.qos.logback.core.spi.FilterReply.DENY
import static ch.qos.logback.core.spi.FilterReply.NEUTRAL


def userDir = System.getProperty("user.dir")
//配置输出info信息
appender("FILE", RollingFileAppender) {
    append =true
    rollingPolicy(TimeBasedRollingPolicy) {
      //输出文件路径
        fileNamePattern = userDir+"/logs/%d{yyyy-MM-dd}_info.log"
        maxHistory = 30 // controls the maximum number of archive files to keep, asynchronously deleting older files
        totalSizeCap = FileSize.valueOf("3 gb")
    }
    encoder(PatternLayoutEncoder) {
        pattern = '%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level  %logger - %msg%n'
    }
    filter(EvaluatorFilter) {
        evaluator(GEventEvaluator) {
            expression = 'e.level.toInt() <= INFO.toInt()'
        }
        onMatch = NEUTRAL
        onMismatch = DENY
    }
}

appender("ERROR_FILE", RollingFileAppender) {
    append =true
    rollingPolicy(TimeBasedRollingPolicy) {
        fileNamePattern =  userDir+"/logs/%d{yyyy-MM-dd}_error.log"
        maxHistory = 30 // controls the maximum number of archive files to keep, asynchronously deleting older files
        totalSizeCap = FileSize.valueOf("3 gb")
    }
    encoder(PatternLayoutEncoder) {
        pattern = '%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level  %logger - %msg%n'
    }
    filter(EvaluatorFilter) {
        evaluator(GEventEvaluator) {
            expression = 'e.level.toInt() > INFO.toInt()'
        }
        onMatch = NEUTRAL
        onMismatch = DENY
    }
}

appender("CONSOLE", ConsoleAppender) {
    encoder(PatternLayoutEncoder) {
        pattern = '%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level  %logger - %msg%n'
    }
    filter(EvaluatorFilter) {
        evaluator(GEventEvaluator) {
            expression = 'e.level.toInt() >= INFO.toInt()'
        }
        onMatch = NEUTRAL
        onMismatch = DENY
    }
}

root(INFO, ["CONSOLE", "ERROR_FILE", "FILE"])
```

下面贴出的是参考网上来自[LogBack——LogBack在项目（Web或Java）中的应用](http://blog.csdn.net/wangdongsong1229/article/details/17463113)的logback.xml的配置:
```
<?xml version="1.0" encoding="UTF-8"?>  
<!--   
    说明：  
    1、日志级别及文件  
        日志记录采用分级记录，级别与日志文件名相对应，不同级别的日志信息记录到不同的日志文件中  
        例如：error级别记录到log_error_xxx.log或log_error.log（该文件为当前记录的日志文件），而log_error_xxx.log为归档日志，  
        日志文件按日期记录，同一天内，若日志文件大小等于或大于2M，则按0、1、2...顺序分别命名  
        例如log-level-2013-12-21.0.log  
        其它级别的日志也是如此。  
    2、文件路径  
        若开发、测试用，在Eclipse中运行项目，则到Eclipse的安装路径查找logs文件夹，以相对路径../logs。  
        若部署到Tomcat下，则在Tomcat下的logs文件中  
    3、Appender  
        FILEERROR对应error级别，文件名以log-error-xxx.log形式命名  
        FILEWARN对应warn级别，文件名以log-warn-xxx.log形式命名  
        FILEINFO对应info级别，文件名以log-info-xxx.log形式命名  
        FILEDEBUG对应debug级别，文件名以log-debug-xxx.log形式命名  
        stdout将日志信息输出到控制上，为方便开发测试使用  
 -->  
<configuration>  

    <!-- 在Eclipse中运行，请到Eclipse的安装目录中找log文件，Tomcat下，请到Tomcat目录下找 -->  
    <property name="LOG_PATH" value="../logs" />  

    <!-- 日志记录器，日期滚动记录 -->  
    <appender name="FILEERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <!-- 正在记录的日志文件的路径及文件名 -->  
        <file>${LOG_PATH}/log_error.log</file>  
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->  
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <!-- 归档的日志文件的路径，例如今天是2013-12-21日志，当前写的日志文件路径为file节点指定，可以将此文件与file指定文件路径设置为不同路径，从而将当前日志文件或归档日志文件置不同的目录。  
            而2013-12-21的日志文件在由fileNamePattern指定。%d{yyyy-MM-dd}指定日期格式，%i指定索引 -->  
            <fileNamePattern>${LOG_PATH}/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>  
            <!-- 除按日志记录之外，还配置了日志文件不能超过2M，若超过2M，日志文件会以索引0开始，  
            命名日志文件，例如log-error-2013-12-21.0.log -->  
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">  
                <maxFileSize>2MB</maxFileSize>  
            </timeBasedFileNamingAndTriggeringPolicy>  
        </rollingPolicy>  
        <!-- 追加方式记录日志 -->  
        <append>true</append>  
        <!-- 日志文件的格式 -->  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <pattern>%-5p [%d] %C:%L - %m %n</pattern>  
            <charset>utf-8</charset>  
        </encoder>  
        <!-- 此日志文件只记录error级别的 -->  
        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>error</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>  
    </appender>  

    <appender name="FILEWARN" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${LOG_PATH}/log_warn.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <fileNamePattern>${LOG_PATH}/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>  
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">  
                <maxFileSize>2MB</maxFileSize>  
            </timeBasedFileNamingAndTriggeringPolicy>  
        </rollingPolicy>  
        <append>true</append>  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <pattern>%-5p [%d] %C:%L - %m %n</pattern>  
            <charset>utf-8</charset>  
        </encoder>  
        <!-- 此日志文件只记录warn级别，不记录大于warn级别的日志 -->  
        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>WARN</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>  
    </appender>  

    <appender name="FILEINFO" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${LOG_PATH}/log_info.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <fileNamePattern>${LOG_PATH}/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>  
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">  
                <maxFileSize>2MB</maxFileSize>  
            </timeBasedFileNamingAndTriggeringPolicy>  
        </rollingPolicy>  
        <append>true</append>  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <pattern>%-5p [%d] %C:%L - %m %n</pattern>  
            <charset>utf-8</charset>  
        </encoder>  
        <!-- 此日志文件只记录info级别，不记录大于info级别的日志 -->  
        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>INFO</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>  
    </appender>  


    <appender name="FILEDEBUG" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${LOG_PATH}/log_debug.log</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <fileNamePattern>${LOG_PATH}/log-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>  
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">  
                <maxFileSize>2MB</maxFileSize>  
            </timeBasedFileNamingAndTriggeringPolicy>  
        </rollingPolicy>  
        <append>true</append>  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <pattern>%-5p [%d] %C:%L - %m %n</pattern>  
            <charset>utf-8</charset>  
        </encoder>  
        <!-- 此日志文件只记录debug级别，不记录大于debug级别的日志 -->  
        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>DEBUG</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>  
    </appender>  

    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">  
        <Target>System.out</Target>  
        <encoder>  
            <pattern>%-5p [%d] %C:%L - %m %n</pattern>  
            <charset>utf-8</charset>  
        </encoder>  
        <!-- 此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->  
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">  
            <level>DEBUG</level>  
        </filter>  
    </appender>  
    <!-- 为单独的包配置日志级别，若root的级别大于此级别， 此处级别也会输出  
        应用场景：生产环境一般不会将日志级别设置为trace或debug，但是为详细的记录SQL语句的情况，  
        可将hibernate的级别设置为debug，如此一来，日志文件中就会出现hibernate的debug级别日志，  
        而其它包则会按root的级别输出日志  
    -->  
    <logger name="org.hibernate.SQL" level="DEBUG" />  
    <logger name="org.hibernate.jdbc" level="DEBUG"/>  
    <logger name="org.springframework" level="DEBUG"/>  


    <!-- 生产环境，将此级别配置为适合的级别，以名日志文件太多或影响程序性能 -->  
    <root level="INFO">  
        <appender-ref ref="FILEDEBUG" />  
        <appender-ref ref="FILEINFO" />  
        <appender-ref ref="FILEWARN" />  
        <appender-ref ref="FILEERROR" />  
        <!-- 生产环境将请stdout去掉 -->  
        <appender-ref ref="stdout" />  
    </root>  
</configuration>  
```
