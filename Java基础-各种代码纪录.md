---
title: Java基础-各种代码纪录
abbrlink: 8a5da8d1
date: 2020-08-19 19:40:46
categories:
  - Java
tags: [Java]
---

> Java 的一些常用代码记录

<!--more-->



## 集合

```java
import com.google.common.collect.ImmutableSet;
// 创建一个不可变的集合
private static final Set<String> REQUEST_URIS = ImmutableSet.of("/", "/login");
```



## 环境信息

```java
// 判断当前系统中是否含有这个环境变量
Boolean.getBoolean("user.name")
```

