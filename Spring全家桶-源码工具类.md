---
title: Spring全家桶-源码工具类
abbrlink: 62e68549
date: 2020-09-23 10:21:30
categories:
  - Java
tags: [Java]
---



# 语法类

```java
// switch语法
switch (this.webApplicationType) {
    case SERVLET:
        contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
        break;
    case REACTIVE:
        contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
        break;
    default:
        contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
}
```





# 工具类

`Bean` 管理工具类

```java
// 根据类全名创建示例
BeanUtils.instantiateClass("org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext");

```



## 字符工具类

```java
// 将集合转为字符串数据
StringUtils.toStringArray(new HashSet())
```

