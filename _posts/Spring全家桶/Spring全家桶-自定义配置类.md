---
title: Spring全家桶-自定义配置类
typora-root-url: ../../
abbrlink: 51eca88c
date: 2019-05-29 14:46:09
categories:
  - Java
  - Spring
tags: [Java, Spring]
---

> Spring 中使用配置类

<!-- more -->



# 介绍

介绍自定义配置Bean注入

## 注解描述

`@Configuration` VS `@Component`
共同点：都可以用于创建Bean；
不同点：实现原理不同，`@Configuration`基于CGlib代理实现，`@Component`基于反射实现；
使用场景：`@Configuration`用于全局配置，比如数据库相关配置，MVC相关配置等；业务Bean的配置使用注解配置(`@Component`,`@Service`,`@Repository`,`@Controller`)。

`@EnableCaching`
对于一些配置文件,数据库连接等对象,通过`@EnableCaching`注解将其缓存中.



## 依赖管理

对于非直接继承自 `spring-boot-starter-parent` 的项目, 通过 `<dependencyManagement>`  标签配置依赖关系.

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.6.3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

