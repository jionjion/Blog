---
title: Java基础-Commons-Log4j2篇
abbrlink: 95377e54
date: 2020-01-28 16:55:04
categories:
  - Java
  - Commons
tags: [Java, Commons, Commons-Log4j2]
---

# 简介
参考 [官网](http://www.apache.org/)

`Log4j2` 是日志接口 `slf4j` 的实现类,类似的实现还有 `logback` 和 `log4j`, 其核心jar为 `log4j-api` , `log4j-core`

## 示例代码

- [*Log4j2Example*][1]



## 介绍

### 默认级别

JDK的 `logging` 方式,最多支持到 `info` 级别

```verilog
四月 09, 2018 7:48:29 下午 commons.logging.LoggingExample main
信息: info级别....
四月 09, 2018 7:48:29 下午 commons.logging.LoggingExample main
严重: error级别....
```

在引入日志框架时,如果不指定配置文件,默认提供的日志级别为 `Error` 级别并输出到控制台中



### 级别关系	

all < trace < debug < info < warn < error < fatal < off



### 配置文件位置

配置文件,一般放置在src/main/resources根目录,以log4j2.xml、log4j.json、log4j.jsn等名称,如果没有找到,则使用默认配置.

Web工程中配置 `log4j2.xml` 位置

```xml
 <context-param>  
    <param-name>log4jConfiguration</param-name>  
    <param-value>/WEB-INF/conf/log4j2.xml</param-value>  
 </context-param> 
```




### 配置文件格式

| 根/子节点       | 子子节点                  | 子子子节点      | 属性              | 说明                                                         |
| --------------- | ------------------------- | --------------- | ----------------- | ------------------------------------------------------------ |
| `Configuration` |                           |                 |                   | 根节点                                                       |
|                 |                           |                 | `status`          | 控制log4j2日志框架本身的日志级别                             |
|                 |                           |                 | `monitorInterval` | 每隔多少秒重新读取配置文件                                   |
| `properties`    |                           |                 |                   | 定义常量                                                     |
| `Appenders`     |                           |                 |                   | 输出源,指定输出的位置,有很多,且支持扩展                      |
|                 | `Console`                 |                 |                   | 控制台输出                                                   |
|                 |                           | `PatternLayout` |                   | 控制台或文件输出源中必须含有,指定输出的文件格式              |
|                 | `File`                    |                 |                   | 文件输出                                                     |
|                 | `RollingRandomAccessFile` |                 |                   | 文件输出,支持拆分文件                                        |
|                 | `NoSql`                   |                 |                   | 输出到非关系型数据库中                                       |
|                 | `Flume`                   |                 |                   | 输出到Apache Flume                                           |
|                 | `Async`                   |                 |                   | 异步输出                                                     |
| `Loggers`       |                           |                 |                   | 日志器 根日志器root和自定义日志器                            |
|                 | `Logger`                  |                 |                   | 自定义日志器  name属性为日志器命名,多以包名命名,配置不同的输出等级 |
|                 | `Root`                    |                 |                   | 默认配置信息                                                 |
|                 |                           | `AppenderRef`   |                   | 输出位置指定                                                 |

**参考配置文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 
	最简单的配置
	
		%d{HH:mm:ss.SSS} 表示输出到毫秒的时间
		%t 输出当前线程名称
		%-5level 输出日志级别，-5表示左对齐并且固定输出5个字符，如果不足在右边补0
		%logger 输出logger名称，因为Root Logger没有名称，所以没有输出
		%msg 日志文本
		%n 换行
		
		其他常用的占位符有：
		%F 输出所在的类文件名，如Log4j2Test.java
		%L 输出行号
		%M 输出所在方法名
		%l 输出语句所在的行数, 包括类名、方法名、文件名、行数	
-->
<Configuration status="WARN">  
	<!-- 输出源配置 -->
    <Appenders>  
    	<!-- 控制台配置 -->
        <Console name="Console" target="SYSTEM_OUT">
        	<!-- 日志输出格式 -->  
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />  
        </Console>  

		<!-- 文件日志配置 -->
        <File name="FileAppender" fileName="F:/JAVA_WorkSpace/JavaBase/src/commons/log4j2/log4j2-01.log">  
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />  
        </File>  

        <!-- 异步输出 -->
        <Async name="AsyncAppender">
        	<!-- 将文件日志异步输出 -->
        	<AppenderRef ref="FileAppender"/>
        </Async>        
    </Appenders>  

	<!-- 日志器配置 -->
    <Loggers>  
    	<!-- root,默认logger配置 日志级别 info -->
        <Root level="info">
        	<!-- 输出位置为控制台 -->  
            <AppenderRef ref="Console" />  
            <!-- 输出位置为文件日志 -->
            <AppenderRef ref="FileAppender" />
        </Root>  
        
        <!-- 自定义logger配置 , name一般与包名一致 -->
        <Logger name="mylog" level="all" additivity="true">
        	<!-- 采用异步日志输出 -->  
            <AppenderRef ref="AsyncAppender" />  
        </Logger>        
    </Loggers>  
</Configuration>
```











[1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/log4j2/Log4j2Example.java