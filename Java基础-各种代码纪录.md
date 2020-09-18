---
title: Java基础-各种代码纪录
abbrlink: 8a5da8d1
date: 2020-08-19 19:40:46
categories:
tags:
---



## 集合

```java
import com.google.common.collect.ImmutableSet;
// 创建一个不可变的集合
private static final Set<String> REQUEST_URIS = ImmutableSet.of("/", "/login");
```

