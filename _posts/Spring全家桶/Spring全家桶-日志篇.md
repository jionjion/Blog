---
title: Spring全家桶-日志篇
typora-root-url: ../../
date: 2022-02-15 10:51:31
categories:
  - Java
  - Spring
tags: [Java, Spring]
---

> Spring 日志框架配置...这里使用 `SL4J` 配合 `Logback`  进行日志记录

<!-- more -->

# 简介

`SpringBoot` 对日志的使用...这里使用 `SL4J` 配合 `Logback`  进行日志记录



# 配置策略

在 `pom` 文件中, 使用场景器引入依赖.. 

```xml
<!-- 日志处理 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```



## 基本使用

```java
@SpringBootTest
class LogbackApplicationTest {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @Test
    void logs() {
        logger.error("日志级别: {}, {}", "错误", "error");
        logger.warn("日志级别: {}, {}", "警告", "warn");
        logger.info("日志级别: {}, {}", "信息", "info");
        logger.debug("日志级别: {}, {}", "调试", "debug");
        logger.trace("日志级别: {}, {}", "堆栈", "trace");
    }
}
```



## 配置文件

在 `resources` 文件夹下创建  `logback-spring.xml`  文件..进行配置

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!--
  logback 配置文件.
    scan:当设置为true时配置文件若发生改变 将会重新加载;
    scanPeriod:扫描时间间隔, 若没给出时间单位默认为毫秒;
    debug:若设置 为true,将打印出logback内部日志信息
-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 上下文名称, 用来区分不同应用程序的记录默认为 default -->
    <contextName>Logger Demo</contextName>
    <!-- 属性配置 -->
    <property resource="application.properties"/>
    <!-- name:变量名称; value:变量值. 通过 :- 表示默认值 -->
    <!-- 文件路径 C:\Users\14345\springboot\application\logs -->
    <property name="LOG_PATH" value="${logging.path:-${user.home}/springboot/${spring.application.name:-application}/logs}"/>
    <!-- 文件名  C:\Users\14345\springboot\application\logs\application.log -->
    <property name="LOG_FILE" value="${logging.file:-${LOG_PATH}/application.log}"/>
    <!--
        logger{length}      输出日志的logger名,可有一个整形参数,可能能是缩短logger名
        contextName|cn      上下文名称
        date{pattern}       输出日志的打印时间,模式语法与java.text.SimpleDateFormat 兼容
        p/le/level          日志级别
        M/method            输出日志的方法名
        r/relative          从程序启动到创建日志记录的时间
        m/msg/message       程序提供的信息
        n                   换行符
     -->
    <!-- 格式化日志输出 -->
    <appender name="APPLICATION" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 输出文件名 -->
        <file>${LOG_FILE}</file>
        <!-- 格式 -->
        <encoder>
            <pattern>%date{HH:mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
        </encoder>
        <!-- 文件策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 文件名 -->
            <fileNamePattern>${LOG_FILE}.%d{yyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 最大文件保存天数 -->
            <maxHistory>7</maxHistory>
            <!-- 单个文件最大大小 -->
            <maxFileSize>50MB</maxFileSize>
            <!-- 总使用大小 -->
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!-- 格式化日志输出, 标准输出到控制台 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- REQUEST_ID 为 MDC注入 -->
            <pattern>[%X{REQUEST_ID}] %date{HH: mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- root: 全局,日志输出设置 -->
    <root level="info">
        <!-- 输出到: 控制台 -->
        <appender-ref ref="STDOUT"/>
        <!-- 输出到: 日志文件 -->
		<appender-ref ref="APPLICATION"/>
    </root>

    <!-- logger 具体包或子类输出设置 -->
    <logger name="top.jionjion.logging.service.SomeService" level="error" additivity="true">
        <appender-ref ref="STDOUT"/>
    </logger>
</configuration>
```



## 自定义日志拆分

有时候,需要根据业务不同拆分为不同的文件..因此

日志配置, 配置 `SiftingAppender` 根据业务关键字, 拆分日志文件. 通过 `<discriminator>` 标签设置关键字.  `bizType`

```xml
<!-- 选填: 根据日志级别分发到不同的日志级别目录下 -->
<appender name="SIFT" class="ch.qos.logback.classic.sift.SiftingAppender">
    <discriminator>
        <!-- 设置分割关键字的默认值,其他日志在其中输出 -->
        <key>bizType</key>
        <defaultValue>other</defaultValue>
    </discriminator>
    <sift>
        <!-- 根据不同的日志类型,输出到不同的文件下 -->
        <property name="BIZ_FILE" value="${logging.file:-${LOG_PATH}/application-${bizType}.log}"/>
        <!-- 选填的日志输出格式 -->
        <appender name="APPLICATION-${bizType}" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <!-- 输出文件名 -->
            <file>${BIZ_FILE}</file>
            <!-- 格式 -->
            <encoder>
                <pattern>%date{HH:mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
            </encoder>
            <!-- 文件策略 -->
            <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
                <!-- 文件名 -->
                <fileNamePattern>${BIZ_FILE}.%d{yyy-MM-dd}.%i.log</fileNamePattern>
                <!-- 最大文件保存天数 -->
                <maxHistory>7</maxHistory>
                <!-- 单个文件最大大小 -->
                <maxFileSize>50MB</maxFileSize>
                <!-- 总使用大小 -->
                <totalSizeCap>10GB</totalSizeCap>
            </rollingPolicy>
        </appender>
    </sift>
</appender>
```

在程序中,指定业务类型, 通过 `org.slf4j.MDC1` 为当前线程中放入关键字 `bizType` 随后日志会根据关键字进行拆分..

```java
@Service
public class SomeService {

    private final Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 抛出自义定异常,并指定关键字
     */
    public void throwExceptionForSift() {
        // 关键字, 根据关键字分发到不同的日志级别
        MDC.put("bizType", "service");
        throw new SomeException("自定义异常");
    }
}
```



## 动态配置日志实例

通过动态配置生成 `Logger` 对象, 实现对日志对象的使用...

```java

/**
 * 动态配置 Logback 的 Logger 实例
 *
 * @author Jion
 */
public class LogbackHolder {

    /**
     * @param name logger的名字, 作为日志前缀
     * @return logger实例
     */
    public static Logger getLogger(String name) {
        // 上下文
        LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
        // 是否已经创建过 logger 对象
        if (loggerContext.exists(name) == null) {
            // 不存在自行创建
            return buildLogger(name);
        }
        return loggerContext.getLogger(name);
    }

    /**
     * 自行创建 logger 对象
     *
     * @param name logger的名字, 作为日志前缀
     * @return logger实例
     */
    private static Logger buildLogger(String name) {
        LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
        // 初始化环境,不存在则创建
        Logger logger = loggerContext.getLogger(name);

        // 配置 RollingFileAppender
        RollingFileAppender<ILoggingEvent> rollingFileAppender = new RollingFileAppender<>();
        rollingFileAppender.setName("CUSTOM");
        rollingFileAppender.setContext(loggerContext);

        // 配置 RollingPolicy
        TimeBasedRollingPolicy<?> rollingPolicy = new TimeBasedRollingPolicy<>();
        rollingPolicy.setFileNamePattern("temp/log" + name + ".%d{yyyyMM}.log");
        rollingPolicy.setParent(rollingFileAppender);
        rollingPolicy.setContext(loggerContext);
        rollingPolicy.start();

        // 配置 Encoder
        PatternLayoutEncoder encoder = new PatternLayoutEncoder();
        encoder.setCharset(StandardCharsets.UTF_8);
        encoder.setPattern("%msg%n");
        encoder.setContext(loggerContext);
        encoder.start();

        // 关联
        rollingFileAppender.setRollingPolicy(rollingPolicy);
        rollingFileAppender.setEncoder(encoder);
        rollingFileAppender.start();

        logger.addAppender(rollingFileAppender);
        return logger;
    }
}
```



测试

```java
@Test
public void test(){
    Logger logger = LogbackHolder.getLogger("jion");
    logger.error("这是错误消息....");
}
```



## 日志切面

在启动类中添加 `@EnableAspectJAutoProxy` 开启切面.

自定义切面类, 在 `service` 层中对所有方法进行日志记录..

```java

/**
 * 自定义异常切面
 *
 * @author Jion
 */
@Component
@Aspect
public class ExceptionAspect {

    private final Logger logger = LoggerFactory.getLogger(ExceptionAspect.class);


    @AfterThrowing(pointcut = "within(top.jionjion.logback.service.*)", throwing = "ex")
    public void handleException(JoinPoint joinPoint, Exception ex) throws Exception {
        // 类名
        String clazz = joinPoint.getSignature().getDeclaringType().getCanonicalName();
        // 方法名
        String method = joinPoint.getSignature().getName();

        // 异常判断,自定义异常.日志处理...
        if (ex instanceof SomeException) {
            logger.warn("class: {}, name:{}", clazz, method, ex);
        }

        // 结束抛出,交给下文处理
        throw ex;
    }
}
```



## 日志链路追踪

在日志配置文件中, 通过 `%X{REQUEST_ID}` 指定变量 `REQUEST_ID` ,尝试获取. 随后在请求和拦截器中,添加请求头 `REQUEST_ID`. 并在拦截器中将其写入到 `MC` 中. 完成同一个请求的链路追踪.

```xml
<!-- 格式化日志输出, 标准输出到控制台 -->
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <!-- REQUEST_ID 为 MDC注入 -->
        <pattern>[%X{REQUEST_ID}] %date{HH: mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

