---
title: Spring全家桶-Spring-Data-Redis篇
date: 2020-06-19 23:20:30
categories:
  - Java
  - Spring
tags: [Java, Spring, JPA]
---

 

# 基础语法

## `RedisTemplate` 成员接口

### 常用接口

| 接口 | 操作/绑定对象 | 获取方法 |
| ---- | ------------- | -------- |
|`GeoOperations`             |  空间地理 | `redisTemplate.opsForGeo()` |
|`HashOperations`             | `Hash` 操作 | `redisTemplate.opsForHash()` |
|`HyperLogLogOperations` |      `HyperLogLog` 操作| `redisTemplate.opsForHyperLogLog()` |
|`ListOperations`             | `List` 操作| `redisTemplate.opsForList()` |
|`SetOperations`              | `Set` 操作| `redisTemplate.opsForSet()` |
|`ValueOperations`    |         `Value` 操作| `redisTemplate.opsForValue()` |
|`ZSetOperations`        |      `Zset` 操作| `redisTemplate.opsForZSet()` |
| | | |
|`BoundGeoOperations`        |  空间地理| `redisTemplate.boundGeoOps("key")` |
|`BoundHashOperations`      |   `Hash` 绑定操作| `redisTemplate.boundHashOps("key")` |
|`BoundKeyOperations`         | `Key` 绑定操作| `redisTemplate.boundValueOps("key")` |
|`BoundListOperations`    |     `List` 绑定操作| `redisTemplate.boundListOps("key")` |
|`BoundSetOperations`       |   `Set` 绑定操作| `redisTemplate.boundSetOps("key")` |
|`BoundValueOperations`     |   `Value` 绑定操作| `redisTemplate.boundValueOps("key")` |
|`BoundZSetOperations`        | `Zset` 绑定操作| `redisTemplate.boundZSetOps("key")` |



### 事物

| 方法                                              | 说明         |
| ------------------------------------------------- | ------------ |
| `redisTemplate.setEnableTransactionSupport(true)` | 开启事物支持 |
| `redisTemplate.multi()`                           | 事物开始     |
| `redisTemplate.exec()`                            | 提交事物     |
| `redisTemplate.discard()`                         | 回滚事物     |
| `SessionCallback<Object>`                         | 事物回调接口 |

### 子接口获取示例

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.data.redis.core.*;
import top.jionjion.DataRedisApplicationTest;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 *  RedisTemplate的测试
 */
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisTemplateTest{

/*
接口	                    描述,操作对象
GeoOperations               空间地理
HashOperations              Hash 操作
HyperLogLogOperations       HyperLogLog 操作
ListOperations              List 操作
SetOperations               Set 操作
ValueOperations             Value 操作
ZSetOperations              Zset 操作

                            绑定操作
BoundGeoOperations          空间地理
BoundHashOperations         Hash 绑定操作
BoundKeyOperations          Key 绑定操作
BoundListOperations         List 绑定操作
BoundSetOperations          Set 绑定操作
BoundValueOperations        Value 绑定操作
BoundZSetOperations         Zset 绑定操作

 */

    /** Redis操作模板,框架提供 */
    @Autowired
    RedisTemplate<String, String> redisTemplate;

    /** 字符串操作模板,框架提供 */
    @Autowired
    StringRedisTemplate stringRedisTemplate;

    /** 测试注入 */
    @Test
    public void testAutowiredBean(){
        assertNotNull(redisTemplate);
        assertNotNull(stringRedisTemplate);
    }

    @Test
    public void testRedisTemplateOpsX(){
        //空间地理
        GeoOperations  geoOperations = redisTemplate.opsForGeo();
        assertNotNull(geoOperations);

        //Hash 操作
        HashOperations hashOperations = redisTemplate.opsForHash();
        assertNotNull(hashOperations);

        //HyperLogLog 操作
        HyperLogLogOperations hyperLogLogOperations = redisTemplate.opsForHyperLogLog();
        assertNotNull(hyperLogLogOperations);

        //List 操作
        ListOperations listOperations = redisTemplate.opsForList();
        assertNotNull(listOperations);

        //Set 操作
        SetOperations setOperations = redisTemplate.opsForSet();
        assertNotNull(setOperations);

        //Value 操作
        ValueOperations valueOperations = redisTemplate.opsForValue();
        assertNotNull(valueOperations);

        //Zset 操作
        ZSetOperations zSetOperations = redisTemplate.opsForZSet();
        assertNotNull(zSetOperations);

        // 绑定已有的key进行操作

        //空间地理
        BoundGeoOperations boundGeoOperations = redisTemplate.boundGeoOps("key");
        assertNotNull(boundGeoOperations);

        //Hash 绑定操作
        BoundHashOperations boundHashOperations = redisTemplate.boundHashOps("key");
        assertNotNull(boundHashOperations);

        //Key 绑定操作
        BoundKeyOperations boundKeyOperations = redisTemplate.boundValueOps("key");
        assertNotNull(boundKeyOperations);

        //List 绑定操作
        BoundListOperations boundListOperations = redisTemplate.boundListOps("key");
        assertNotNull(boundListOperations);

        //Set 绑定操作
        BoundSetOperations boundSetOperations = redisTemplate.boundSetOps("key");
        assertNotNull(boundSetOperations);

        //Value 绑定操作
        BoundValueOperations boundValueOperations = redisTemplate.boundValueOps("key");
        assertNotNull(boundValueOperations);

        //Zset 绑定操作
        BoundZSetOperations boundZSetOperations = redisTemplate.boundZSetOps("key");
        assertNotNull(boundZSetOperations);
    }

    /** 开启事物 */
    @Test
    public void testTransactional(){
        redisTemplate.setEnableTransactionSupport(true);
        // 开始事物
        redisTemplate.multi();
        // 提交事物
        redisTemplate.exec();
        // 回归事物
        redisTemplate.discard();
    }


    /** 使用回调进行事物 */
    @Test
    public void testTransactionalWithCallBack() {
        SessionCallback<Object> callback = new SessionCallback<>() {
            @Override
            public Object execute(RedisOperations operations) throws DataAccessException {
                operations.multi();
                /* ... */
                return operations.exec();
            }
        };
        redisTemplate.execute(callback);
    }
}
```



## `ValueOperations` 接口
`ValueOperations` 对 `Values` 的操作

### 常用方法

