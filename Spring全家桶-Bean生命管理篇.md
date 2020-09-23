---
title: Spring全家桶-Bean生命管理篇
abbrlink: 7f310c97
date: 2020-08-11 21:01:46
categories:
tags:
---

## Bean生命周期处理

### `@Bean` 注解

通过 `@Bean` 配合 `@Configuration` 注解对其进行声明, 在其中 `initMethod` 和 `destroyMethod` 分别指定其初始化和销毁方法

```java
/** 配置类 */
@Configuration
public class AppConfig {

    @Bean(name = {"user","USER"}, initMethod = "init", destroyMethod = "destroy")
    public User getUser(){
        return new User();
    }
}
```

