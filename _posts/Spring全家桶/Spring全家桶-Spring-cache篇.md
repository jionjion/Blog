---
title: Spring全家桶-Spring-cache篇
typora-root-url: ../../
categories:
  - Java
  - Spring
tags:
  - Java
  - Spring
abbrlink: 9b688053
date: 2022-07-02 16:39:24
---

> Spring中使用默认的缓存组件

<!-- more -->



## Spring-Cache-Starter  

`SpringBoot` 提供的开箱即用的缓存应用, 默认使用 `AOP` 方法支持自定义 `key` 和各种缓存管理器. 但是不支持自定义过期时间
默认使用 `ConcurrentMap` 实现..



## 引入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```



## 注解

| 注解/接口                                            | 说明                                                         |
| :--------------------------------------------------- | :----------------------------------------------------------- |
| `org.springframework.cache.Cache`                    | 缓存接口, 定义缓存操作. 常用实现 `RedisCache`、`EhCacheCache`、`ConcurrentMapCache` |
| `org.springframework.cache.CacheManager`             | 缓存管理器, 管理各种缓存组件                                 |
| `org.springframework.cache.interceptor.KeyGenerator` | 缓存的key生成策略                                            |
| `serialize`                                          | 缓存的value序列化策略                                        |
|                                                      |                                                              |
| `@EnableCaching`                                     | 开启基于注解的缓存, 配置在启动类配置类上                     |
| `@CacheConfig`                                       | 统一配置本类的缓存注解的属性                                 |
| `@Caching`                                           | 可以通过 `@Caching` 注解组合多个缓存策略在一个方法上         |
| `@Cacheable`                                         | 方法注解, 在执行前检查是否在缓存中, 如果有返回缓存数据; 否则调用方法并将返回值缓存 |
| `@CachePut`                                          | 执行方法将返回值放到缓存中.. 与 `@Cacheable` 区别在于是否每次都调用方法.. 常用于更新 |
| `@CacheEvict`                                        | 将一条或多条数据从缓存中删除                                 |

### `@Cacheable` / `@CachePut` / `@CacheEvict` 主要的参数

| 名称               | 解释                                                         | 示例                                                         |
| :----------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| `value`            | 缓存的名称, 在 `spring` 配置文件中定义, 必须指定至少一个     | `@Cacheable(value=”mycache”)` ;  `@Cacheable(value={”cache1”,”cache2”}` |
| `key`              | 缓存的 key, 可以为空, 如果指定要按照 `SpEL` 表达式编写,  如果不指定, 则缺省按照方法的所有参数进行组合 | ` @Cacheable(value=”testcache”,key=”#id”)`                   |
| `condition`        | 缓存的条件, 可以为空, 使用 `SpEL` 编写, 返回 `true` 或者 `false`,  只有为 `true` 才进行缓存/清除缓存 | `@Cacheable(value=”testcache”,condition=”#userName.length()>2”)` |
| `unless`           | 否定缓存。当条件结果为 `true` 时, 就不会缓存。               | `@Cacheable(value=”testcache”,unless=”#userName.length()>2”)` |
| `allEntries`       | 是否清空所有缓存内容, 缺省为 `false`, 如果指定为 `true`,  则方法调用后将立即清空所有缓存 | `@CachEvict(value=”testcache”,allEntries=true)`              |
| `beforeInvocation` | 是否在方法执行前就清空, 缺省为 `false`, 如果指定为 `true`,  则在方法还没有执行的时候就清空缓存, 缺省情况下, 如果方法 执行抛出异常, 则不会清空缓存 | `@CachEvict(value=”testcache”, beforeInvocation=true)`       |



## `SpEL` 表达式

| 名称            | 位置       | 描述                                                         | 示例                   |
| :-------------- | :--------- | :----------------------------------------------------------- | :--------------------- |
| `methodName`    | root对象   | 当前被调用的方法名                                           | `#root.methodname`     |
| `method`        | root对象   | 当前被调用的方法                                             | `#root.method.name`    |
| `target`        | root对象   | 当前被调用的目标对象实例                                     | `#root.target`         |
| `targetClass`   | root对象   | 当前被调用的目标对象的类                                     | `#root.targetClass`    |
| `args`          | root对象   | 当前被调用的方法的参数列表                                   | `#root.args[0]`        |
| `caches`        | root对象   | 当前方法调用使用的缓存列表                                   | `#root.caches[0].name` |
| `Argument Name` | 执行上下文 | 当前被调用的方法的参数, 如 `findArtisan(Artisan artisan)` 可以通过 `#artsian.id` 获得参数 | `#artsian.id`          |
| `result`        | 执行上下文 | 方法执行后的返回值（仅当方法执行后的判断有效, 如 `@CacheEvict`的 `unless` 和 `beforeInvocation=false`） | `#result`              |

当我们要使用 `root` 对象的属性作为 `key` 时我们也可以将 `#root` 省略, 因为 `Spring` 默认使用的就是root对象的属性。

```
@Cacheable(key = "targetClass + methodName +#p0")
```

使用方法参数时我们可以直接使用“#参数名”或者“#p参数index”。 如：

```
@Cacheable(value="users", key="#id")
@Cacheable(value="users", key="#p0")
```



**SpEL提供了多种运算符**

| **类型**   | **运算符**                                     |
| :--------- | :--------------------------------------------- |
| 关系       | <, >, <=, >=, ==, !=, lt, gt, le, ge, eq, ne   |
| 算术       | +, - , * , /, %, ^                             |
| 逻辑       | &&, \|\|, !, and, or, not, between, instanceof |
| 条件       | ?: (ternary), ?: (elvis)                       |
| 正则表达式 | matches                                        |
| 其他类型   | ?., ?[…], ![…], ^[…], $[…]                     |



## 示例代码

添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

启动类添加 `@EnableCaching` 注解

```java
@EnableCaching
@SpringBootApplication
public class CachingApplication {
    public static void main(String[] args) {
        SpringApplication.run(CachingApplication.class, args);
    }
}
```

带有缓存的 `Service` 类

```java
package top.jionjion.caching.service;


import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.*;
import org.springframework.stereotype.Service;
import top.jionjion.caching.repository.PhotoRepository;
import top.jionjion.caching.vo.Photo;

/**
 * 缓存服务
 *
 * @author Jion
 */
@Service
@CacheConfig(cacheNames = "photos")
public class PhotoService extends AbstractPhotoService {

    private final static Logger LOGGER = LoggerFactory.getLogger(PhotoService.class);

    @Autowired
    PhotoRepository photoRepository;


    /**
     * 配置多个缓存策略
     * Cacheable[] cacheable() default {}; //声明多个@Cacheable
     * CachePut[] put() default {};        //声明多个@CachePut
     * CacheEvict[] evict() default {};    //声明多个@CacheEvict
     * <p>
     * 新增照片
     *
     * @param id    主键
     * @param title 标题
     * @return 返回值.放入缓存中
     */
    @Caching(put = {@CachePut(cacheNames = "photos", key = "'photo:' + #id")})
    public Photo insert(Long id, String title) {
        Photo photo = new Photo(id, title, "JPEG:data//" + super.getRandomString());
        LOGGER.info("保存...{}", photo.getTitle());
        photoRepository.insert(photo);
        return photo;
    }

    /**
     * CacheEvict 的作用 主要针对方法配置，能够根据一定的条件对缓存进行清空
     * 触发缓存清除
     * 默认先执行数据库删除再执行缓存删除
     */
    @CacheEvict(cacheNames = "photos", key = "'photo:' + #id")
    public void delete(Long id) {
        LOGGER.info("删除...{}", id);
        photoRepository.delete(id);
    }

    /**
     * 删除全部以及缓存
     */
    @CacheEvict(cacheNames = "photos", allEntries = true)
    public void deleteAll() {
        System.out.println("删除全部...");
        photoRepository.deleteAll();
    }

    /**
     * CachePut注解的作用 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存，
     * 和 @Cacheable 不同的是，它每次都会触发真实方法的调用
     * 简单来说就是用户更新缓存数据。但需要注意的是该注解的value 和 key 必须与要更新的缓存相同，也就是与@Cacheable 相同
     * 默认先执行数据库更新再执行缓存更新
     * 注意返回值必须是要修改后的数据
     *
     * @param photo 修改数据
     * @return 返回值.放入缓存中
     */
    @CachePut(cacheNames = "photos", key = "'photo:' + #photo.id")
    public Photo update(Photo photo) {
        LOGGER.info("更新...{}", photo.getTitle());
        photoRepository.update(photo);
        return photo;
    }

    /**
     * Cacheable注解会先查询是否已经有缓存，有会使用缓存，没有则会执行方法并缓存
     * 命名空间: @Cacheable 的 value 会替换 @CacheConfig 的 cacheNames(两者必须有一个)
     * key的构成: [命名空间]::[@Cacheable的key或者KeyGenerator生成的key](@Cacheable的key优先级高,KeyGenerator不配置走默认KeyGenerator SimpleKey [])
     * <p>
     * 根据主键查询
     *
     * @param id 主键
     * @return 返回值.放入缓存中
     */
    @Cacheable(value = "photos", key = "'photo:' + #id", unless = "#result == null")
    public Photo findById(Long id) {
        super.simulateSlowService();
        return photoRepository.find(id);
    }
}
```

