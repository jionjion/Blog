---
title: Java基础- Apache-Commons篇
abbrlink: 6740b75
date: 2021-03-08 11:22:14
categories:
  - Java
  - Commons
tags: [Java, Commons]
---

> Apache-Commons 的使用

<!-- more -->



# Collections

简化Java中集合框架的操作. 参考[官网](http://commons.apache.org/proper/commons-collections/)

其中

- `ListUtils` 提供了对 `List` 的扩展支持

- `SetUtils` 提供了对 `Set` 集合的扩展工具类
- `SetView` 内部类为 `SetUtils` 下的不可变的 `Set` 集合
- `Bag` 提供了对 `Set` 集合的统计扩展
- `MapUtils` 提供了对 `Map` 字典的扩展工具
- `BidiMap` 扩展了 `Map` 字典,可以通过 `value` 找到 `key`
- `IterableMap`  继承 `Map` 字典,可以进行迭代操作
- `OrderedMap` 扩展了 `Map` 字典可以



## 示例代码

- [*BagsExample*][1-1] 				额外提供了对 `set` 的统计,计算类
- [*BidiMapExample*][1-2]           可以通过 `key` 寻找 `value` ,也可以通过 `value` 寻找 `key` 要求 `Key-Value` 一一对应,否则如果 `key` 或者 `value` 重复,只会保存最后一个结果
- [*IterableMapExample*][1-3]     `IterableMap` 类的例子
- [*ListUtilsExample*][1-4]            测试 `ListUtils` 类的常用方法
- [*MapUtilsExample*][1-5]          `MapUtils` 类的例子
- [*OrderedMapExample*][1-6]   `OrderedMap` 类的例子
- [*SetUtilsExample*][1-7]            `SetUtils` 类常用的方法
- [*SetViewExample*][1-8]            `SetView` 类,只读的 `set` 集合



[1-1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/BagsExample.java
[1-2]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/BidiMapExample.java
[1-3]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/IterableMapExample.java
[1-4]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/ListUtilsExample.java
[1-5]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/MapUtilsExample.java
[1-6]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/OrderedMapExample.java
[1-7]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/SetUtilsExample.java
[1-8]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/SetViewExample.java



# Codec

参考 [官网](http://commons.apache.org/proper/commons-codec/)



## 示例代码

* [*Base64Demo*][2-1]     Base64编码与解码测试
* [*HexDemo*][2-2]           Hex十六进制编码与解码测试
* [*MD5Demo*][2-3]         MD5摘要测试
* [*URLDemo*][2-4]          URL编码与解码测试



[2-1]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/codec/Base64Demo/Base64Example.java
[2-2]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/codec/HexDemo/HexExample.java
[2-3]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/codec/MD5Demo/MD5Example.java
[2-4]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/codec/URLDemo/URLExample.java



# Dbcp

通过使用 `commons-dbcp2` 扩展程序,连接 `MySQL` 数据库


## 代码示例


- [*MysqlDbcpExample*][3-1] 						通过数据源连接 `MySQL` 数据库


[3-1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/dbcp/MysqlDbcpExample.java



# IO

`Commons-IO` 提供了对文件,目录的IO操作,其中常用的工具类有

- `XXXComparator` 文件类,对当前文件列表进行各种排序
- `XXXFilter` 文件过滤,查找符合要求的文件对象
- 目录文件内文件变化监听 
- `FilenameUtils` 文件名工具类
- `FileSystemUtils` 当前文件系统工具类
- `FileUtils` 文件对象类
- `IOCase` 通过IO流比较对象类


## 示例代码

- [*ComparatorExample*][4-1] 							轻松地比较和排序文件和目录,根据文件名,大小,最后修改日比较
- [*FiltersExample*][4-2]                                        文件过滤器,通过不同的筛选条件,进行文件查询
- [*InputExample*][4-3]                                         自动将读取的字节从输入复制到输出
- [*FileMonitorExample*][4-4]                               监视文档的状态变化
- [*OutputExample*][4-5]                                      可以将输入流发送到2个不同的输出
- [*FilenameUtilsExample*][4-7]                           与文件名相关的操作
- [*FileSystemUtilsExample*][4-6]                         返回指定储存介质的可用空间
- [*FileUntilsExample*][4-8]                                   文件操作（移动，打开和读取文件，检查文件是否存在等）的方法
- [*IOCaseExample*][4-9]                                       通过 `IOCase` 枚举类, 进行IO之间的比较操作


[4-1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/comparatorDemo/ComparatorExample.java
[4-2]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/filefilterDemo/FiltersExample.java
[4-3]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/inputDemo/InputExample.java
[4-4]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/monitorDemo/FileMonitorExample.java
[4-5]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/outputDemo/OutputExample.java
[4-6]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/utilityTool/FileSystemUtilsExample.java
[4-7]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/utilityTool/FilenameUtilsExample.java
[4-8]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/utilityTool/FileUntilsExample.java
[4-9]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/utilityTool/IOCaseExample.java



# Lang2

`Commons-Lang2` 封装了一些关于JAVA的基础操作.如数组操作, 数字操作, 字符串操作, 日期时间, 反射等



## 示例代码

[*ArrayExample*][5-1] 								数组操作

[*NumberExample*][5-2]							数字操作

[*StringExample*][5-3]								字符串操作,不会抛出空指针

[*RandomStringExample*][5-4]				  随机字符串操作

[*DateExample*][5-5] 							 	日期操作

[*ClassReflectExample*][5-6]					  反射-类信息的获取	

[*FieldReflectExample*][5-7]					   反射-属性信息的获取

[*MethodReflectExample*][5-8]				  反射-方法调用




[5-1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/lang/arrayDemo/ArrayExample.java
[5-2]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/lang/mathDemo/NumberExample.java
[5-3]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/lang/stringDemo/StringExample.java
[5-4]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/lang/stringDemo/RandomStringExample.java
[5-5]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/lang/timeDemo/DateExample.java
[5-6]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/lang/reflectDemo/ClassReflectExample.java
[5-7]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/lang/reflectDemo/FieldReflectExample.java
[5-8]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/lang/reflectDemo/MethodReflectExample.java



# Log4j2

参考 [官网](http://www.apache.org/)

`Log4j2` 是日志接口 `slf4j` 的实现类,类似的实现还有 `logback` 和 `log4j`, 其核心jar为 `log4j-api` , `log4j-core`

## 示例代码

- [*Log4j2Example*][6-1]



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



[6-1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/log4j2/Log4j2Example.java





# Email

通过 `commons-email` 扩展程序包,发送邮件



## 代码示例

- [*SendQQEmailExample*][7-1]  				通过QQ邮箱发送邮件



[7-1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/email/SendQQEmailExample.java

