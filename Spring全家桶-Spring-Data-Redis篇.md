---
title: Spring全家桶-Spring-Data-Redis篇
abbrlink: 2dcb30a0
date: 2020-06-19 23:20:30
categories:
  - Java
  - Spring
tags: [Java, Spring, JPA]
---

# 基础语法

默认, 在`SpringBoot 2.x` 中, 使用 `lettuce` 进行数据库操作

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
public class RedisTemplateTest {
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

| 声明方法                                                     | Redis操作                                                | 说明                                                         |
| ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
| `void set(K key, V value)`                                   | `SET key value [EX seconds] [PX milliseconds] [NX / XX]` | 将数据以键值对的形式存入                                     |
| `Boolean setIfAbsent(K key, V value)`                        | `SET key value [EX seconds] [PX milliseconds] [NX]`      | 当key不存在时,存入;否则报错.类似新增操作(set if not exists)  |
| `Boolean setIfPresent(K key, V value)`                       | `SET key value [EX seconds] [PX milliseconds] [XX]`      | 当key存在是,更新;否则不做处理.类似更新操作(set exists)       |
| `V get(Object key)`                                          | `GET key`                                                | 通过键,获得以键值对形式存在的值.如果值不存在,则返回nil;另外,键key只能为字符串 |
| `V getAndSet(K key, V value)`                                | `GETSET key value`                                       | 将key对应的值替换为value,并将替换之前的value返回.如果是新建,则返回nil |
| `Long size(K key)`                                           | `STRLEN key`                                             | 获得指定key对应的value的长度,仅限在value类型为字符类型时才有长度,否则返回0 |
| `Integer append(K key, String value)`                        | `APPEND key value`                                       | 如果value存在,则将其拼在原有的值后面,不存在则存入新值.并返回追加后的长度. |
| `void set(K key, V value, long offset)`                      | `SETRANGE key offset value`                              | key为字符类型,offset为非负数,value为字符类型.其目的是将key所对应的值的offset位置以后的内容替换为value所对应的值,类似于截取替换. |
| `String get(K key, long start, long end)`                    | `GETRANGE key start end`                                 | key为字符类型,start和end类型为数字,正数表示位置从左往右,负数表示从右往左,-1为最后一个字符的位置.start和end的过大或者过小只会返回到极限位置对应的字符串 |
| `Long increment(K key)`                                      | `INCR key`                                               | 将key对应的值进行自增                                        |
| `Long increment(K key, long delta)`                          | `INCRBY key increment`                                   | key对应的值必须为整数类型,increment为整数,表示将key对应的值其加上increment增量 |
| `INCRBYFLOAT key increment`                                  | `INCRBYFLOAT key increment`                              | key对应的值必须为数字,但不限制精度.increment为数字类型,不限精度.表示对key对应的相加increment增量 |
| `Long decrement(K key)`                                      | `DECR key`                                               | 将key对应的值进行自减                                        |
| `Long decrement(K key, long delta);`                         | `DECRBY key decrement`                                   | key对应的值必须为整数,decrement必须也为整数.表示key对应的值减去decrement增量 |
| `void multiSet(Map<? extends K, ? extends V> map)`           | `MSET key value [key value ...]`                         | key为字符类型,value不做限制,表示对一个或者多对键值对进行赋值,如果key重复,则更新对应的value |
| `Boolean multiSetIfAbsent(Map<? extends K, ? extends V> map)` | `MSETNX key value [key value ...]`                       | key为字符类型,value不做限制.当key不存在时,执行赋值操作.其中,如果任意一个key存在,则整个赋值操作执行失败.事务回滚 |
| `List<V> multiGet(Collection<K> keys)`                       | `MGET key [key …]`                                       | key为字符类型.获得一个或者多个key对应的值.不存在返回nil      |



### 方法调用示例

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.Before;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import top.jionjion.DataRedisApplicationTest;

import java.time.Duration;
import java.util.*;
import java.util.concurrent.TimeUnit;

import static org.junit.Assert.*;

/**
 * @author Jion
 *  测试 Value
 *      ValueOperations 对 Values 的操作
 */
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class ValueOperationsTest {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    private ValueOperations<String,String> valueOperations;

    @Before
    public void initOperations(){
        if (Objects.isNull(valueOperations)){
            valueOperations = redisTemplate.opsForValue();
        }
        // 初始化数据,启用事物,并提交
        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.multi();
        redisTemplate.delete("KeyA");
        valueOperations.set("KeyA","Aa");
        valueOperations.set("KeyZero","0");
        redisTemplate.exec();

//        valueOperations.set();
//        valueOperations.setIfAbsent();
//        valueOperations.setIfPresent();
//        valueOperations.get();
//        valueOperations.getAndSet();
//        valueOperations.size();
//        valueOperations.append();
//        valueOperations.increment();
//        valueOperations.decrement();
//        valueOperations.multiSet();
//        valueOperations.multiSetIfAbsent();
//        valueOperations.multiGet();
    }

    /**
     *  语法 SET key value [EX seconds] [PX milliseconds] [NX|XX]
     *  将数据以键值对的形式存入
     *
     *  key为字符型,value格式不限制
     *  例如:set name "jionjion" 将jionjion存入,key为name
     *  [EX seconds]过期时间,单位秒
     *  例如:set name "jionjion" ex 5 将jionjion存入,key为name,且过期时间为5秒
     *  [PX milliseconds]设置过期时间,单位毫秒
     *  例如:set name "jionjion" px 5000 将jionjion存入,key为name,且过期时间为5秒
     *  [NX] 当key不存在时,存入;否则报错.类似新增操作(set if not exists)
     *  例如:set name "jionjion" NX 当name不存在,新增
     *  [XX] 当key存在是,更新;否则不做处理.类似更新操作(set exists)
     *  例如:set name "jionjion" XX 当name存在,更新
     */
    @Test
    public void testSet(){
        // 简单地保存,Key存在则覆盖
        valueOperations.set("KeyA","Aa");
        // 保存并指定过期时间
        valueOperations.set("KeyB","Bb",10, TimeUnit.SECONDS);
        valueOperations.set("KeyC","Cc", Duration.ofMillis(1));
    }

    @Test
    public void testSetIfAbsent(){
        // 保存,当Key不存在时,进行赋值
        Boolean result1 = valueOperations.setIfAbsent("KeyA","Aa");
        assertEquals(false, result1);
        // 保存并指定过期的时间
        Boolean result2 = valueOperations.setIfAbsent("KeyB","Bb",10,TimeUnit.SECONDS);
        assertEquals(true, result2);
        Boolean result3 = valueOperations.setIfAbsent("KeyC","Cc",Duration.ofMillis(1));
        assertEquals(true, result3);
    }


    @Test
    public void testSetIfPresent(){
        // 保存,当Key存在时,进行覆盖更新
        Boolean result1 = valueOperations.setIfPresent("KeyA","Aa");
        assertEquals(true, result1);
        // 保存,当Key存在时,进行覆盖更新,并指定过期的时间
        Boolean result2 = valueOperations.setIfPresent("KeyB","Bb",10,TimeUnit.SECONDS);
        assertEquals(false, result2);
        Boolean result3 = valueOperations.setIfPresent("KeyC","Cc",Duration.ofMillis(1));
        assertEquals(false, result3);
    }

    /**
     *  语法 GET key
     *  通过键,获得以键值对形式存在的值.如果值不存在,则返回nil;另外,键key只能为字符串
     */
    @Test
    public void testGet(){
        String result = valueOperations.get("KeyA");
        assertNotNull(result);
        log.info(result);
    }

    /**
     *  语法 GETSET key value
     *  将key对应的值替换为value,并将替换之前的value返回.如果是新建,则返回nil
     */
    @Test
    public void testGetAndSet(){
        String reuslt = valueOperations.getAndSet("KeyA","Aa1");
        assertEquals("Aa", reuslt);
        log.info(reuslt);
    }

    /**
     *  语法 STRLEN key
     *  获得指定key对应的value的长度,仅限在value类型为字符类型时才有长度,否则返回0
     */
    @Test
    public void testSize(){
        Long result = valueOperations.size("KeyA");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 APPEND key value
     *  如果value存在,则将其拼在原有的值后面,不存在则存入新值.并返回追加后的长度.
     */
    @Test
    public void testAppend(){
        Integer result = valueOperations.append("KeyA","AA");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 SETRANGE key offset value
     *  其中,key为字符类型,offset为非负数,value为字符类型.其目的是将key所对应的值的offset位置以后的内容替换为value所对应的值,类似于截取替换.
     *  当key不存在时,将value写入.
     *  当offset大于字符串的最大长度时,填充\x00
     *  当offset过大时,会消耗过多内容,容易造成阻塞!
     */
    @Test
    public void testSetrange(){
        valueOperations.set("KeyA","Bb",0);
    }

    /**
     *  语法 GETRANGE key start end
     *  其中key为字符类型,start和end类型为数字,正数表示位置从左往右,负数表示从右往左,-1为最后一个字符的位置.start和end的过大或者过小只会返回到极限位置对应的字符串
     */
    @Test
    public void testGetrange(){
        String result = valueOperations.get("KeyA",0,1);
        assertNotNull(result);
        log.info(result);
    }

    /**
     *  语法 INCR key
     *  其中,key对应的值必须为数字类型,表示并将其自增+1.当key不存在时,默认返回1
     *  注意,key对应的值在redis中以数字字符串进行保存!
     */
    @Test
    public void testIncrement(){
        Long result = valueOperations.increment("KeyZero");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 INCRBY key increment
     *  将key对应的值进行自增
     *  其中,key对应的值必须为整数类型,increment为整数,表示将key对应的值其加上increment增量
     */
    @Test
    public void testIncremenByLong(){
        Long result = valueOperations.increment("KeyZero",10L);
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 INCRBYFLOAT key increment
     *  其中,key对应的值必须为数字,但不限制精度.increment为数字类型,不限精度.表示对key对应的相加increment增量.
     *  注意:存储时会自动将尾数抹零.
     */
    @Test
    public void testIncrementByDouble(){
        Double result = valueOperations.increment("KeyZero",100D);
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 DECR key
     *  将key对应的值进行自减.
     */
    @Test
    public void testDecrement(){
        Long result = valueOperations.decrement("KeyZero");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 DECRBY key decrement
     *  其中,key对应的值必须为整数,decrement必须也为整数.表示key对应的值减去decrement增量
     */
    @Test
    public void testDecrementByLong(){
        Long result = valueOperations.decrement("KeyZero",-10L);
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 MSET key value [key value ...]
     *  其中,key为字符类型,value不做限制,表示对一个或者多对键值对进行赋值,如果key重复,则更新对应的value
     */
    @Test
    public void testMultiSet(){
        Map<String,String> keys = new HashMap<>();
        keys.put("KeyA","Aa");
        keys.put("KeyB","Ba");
        keys.put("KeyC","Cc");
        valueOperations.multiSet(keys);
    }

    /**
     *  语法 MSETNX key value [key value ...]
     *  其中,key为字符类型,value不做限制.当key不存在时,执行赋值操作.其中,如果任意一个key存在,则整个赋值操作执行失败.事务回滚
     *  返回1,表示执行成功;返回0,表示执行失败.
     */
    @Test
    public void testMultiSetIfAbsent(){
        Map<String, String> keys = new HashMap<>();
        keys.put("KeyA","Aa");
        keys.put("KeyB","Ba");
        keys.put("KeyC","Cc");
        Boolean result = valueOperations.multiSetIfAbsent(keys);
        assertEquals(false, result);
    }

    /**
     *  语法 MGET key [key …]
     *  其中,key为字符类型.获得一个或者多个key对应的值.不存在返回nil
     */
    @Test
    public void testMultiGet(){
        Set<String> keys = new HashSet<>();
        keys.add("KeyA");
        keys.add("KeyB");
        keys.add("KeyC");
        List<String> results = valueOperations.multiGet(keys);
        assertNotNull(results);
        log.info(results.toString());
    }
}
```



##  `ListOperations` 接口

`ListOperations` 对 `List` 的操作

### 常用方法

| 声明方法                                                     | Redis操作                               | 说明                                                         |
| ------------------------------------------------------------ | --------------------------------------- | ------------------------------------------------------------ |
| `Long leftPush(K key, V value)`                              | `LPUSH key value [value ...]`           | 将一个或者多个value值顺次插入到列表key的头前.key必须为列表类型,否则报错;如果key列表不存在,则自动创建.key列表的顺序由插入时的顺序决定,并且允许重复插入. |
| `Long leftPushAll(K key, V... values);`                      | `LPUSH key value [value ...]`           | 将一个或者多个value顺次插入到列表key的头前.key必须为列表类型,否则报错;如果key列表不存在,则自动创建.key列表的顺序由插入时的顺序决定,并且允许重复插入. |
| `Long leftPushIfPresent(K key, V value)`                     | `LPUSHX key value [value ...]`          | 将一个或者多个value值顺次插入到列表key的头前.key必须存在且为列表类型,否则不做任何处理. |
| `Long rightPush(K key, V value)`                             | `RPUSH key value [value ...]`           | 将一个或者多个value值顺次追加到列表尾.key必须为列表类型,否则报错.如果key列表不存在,则自动创建. |
| `Long rightPushAll(K key, Collection<V> values)`             | `RPUSH key value [value ...]`           | 将一个或者多个value值顺次追加到列表尾.key必须为列表类型,否则报错.如果key列表不存在,则自动创建. |
| `Long rightPushIfPresent(K key, V value)`                    | `RPUSHX key value [value ...]`          | 将一个或者多个value值顺次追加到列表为,key必须存在且为列表类型,否则不做任何修改. |
| `V leftPop(K key)`                                           | `LPOP key`                              | 将列表头元素删除,并返回.key不存在或者列表为空时,返回nil.     |
| `V rightPop(K key)`                                          | `RPOP key`                              | 将列表尾元素删除,并返回.key不存在或者列表为空时,返回nil.     |
| `V rightPopAndLeftPush(K sourceKey, K destinationKey)`       | `RPOPLPUSH source destination`          | source和destination必须列表类型,命令将source列表的最后一个元素弹出,并返回给客户端.同时,将弹出的元素重新插入到destination列表的第一个位置. |
| `Long remove(K key, long count, Object value)`               | `LREM key count value`                  | key必须为列表,count必须为整数,其绝对值表示将要从key列表中移除的value的个数 |
| `Long size(K key)`                                           | `LLEN key`                              | 返回类表的长度,如果列表不存在,或者为空列表,则返回0           |
| `V index(K key, long index)`                                 | `LINDEX key index`                      | 返回列表key中,索引位置为index的元素.其中,key必须为列表类型,index为必须为整数,正整数表示索引从前往后,0为列表第一个元素;负整数表示索引从后往前,-1为列表最后一个元素.如果索引超过列表长度,返回nil. |
| `void set(K key, long index, V value)`                       | `LSET key index value`                  | 将列表key的索引位置为index的元素的值替换为value              |
| `List<V> range(K key, long start, long end)`                 | `LRANGE key start stop`                 | 返回列表key中,从start到stop区间内的元素,索引从0开始,表示第一个元素;-1表示最后一个元素.start或stop过小或者过大均自动截取列表的极值 |
| `void trim(K key, long start, long end)`                     | `LTRIM key start stop`                  | 对列表key进行修剪,只保留start和stop的区间内的元素.start和stop超过列表长度,返回错误. |
| `V leftPop(K key, long timeout, TimeUnit unit)`              | `BLPOP key [key ...] timeout`           | 阻塞式弹出列表key的列首第一个元素,如过列表为空,等待timeout秒,如果timeout设置为0,表示一直等待到列表具有元素时才进行弹出. |
| `V rightPop(K key, long timeout, TimeUnit unit)`             | `BRPOP key [key ...] timeout`           | 阻塞式弹出列表key的列尾最后一个元素,如过列表为空,等待timeout秒,如果timeout设置为0,表示一直等待到列表具有元素时才进行弹出. |
| `V rightPopAndLeftPush(K sourceKey, K destinationKey, long timeout, TimeUnit unit)` | `BRPOPLPUSH source destination timeout` | 将source列表的列尾元素弹出,并插入destination列表的列首,如果source列表为空,则等待timeout秒,timeout为0,则持续等待. |



### 方法调用示例

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.Before;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.ListOperations;
import org.springframework.data.redis.core.RedisTemplate;
import top.jionjion.DataRedisApplicationTest;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import java.util.concurrent.TimeUnit;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 *  测试 List
 *      ListOperations 对List的操作
 */
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class ListOperationsTest {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    private ListOperations<String,String> listOperations;

    @Before
    public void initOperations(){
        if (Objects.isNull(listOperations)){
            listOperations = redisTemplate.opsForList();
        }
        // 初始化数据,启用事物,并提交
        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.multi();
        redisTemplate.delete("ListA");
        listOperations.rightPush("ListA","A");
        listOperations.rightPush ("ListA","B");
        listOperations.rightPush ("ListA","C");
        redisTemplate.delete("ListB");
        listOperations.rightPush ("ListB","D");
        listOperations.rightPush ("ListB","E");
        listOperations.rightPush ("ListB","F");
        redisTemplate.exec();


//        listOperations.leftPush();
//        listOperations.leftPushAll();
//        listOperations.leftPushIfPresent()
//        listOperations.rightPush();
//        listOperations.rightPushAll();
//        listOperations.rightPushIfPresent();
//        listOperations.leftPop();
//        listOperations.rightPop();
//        listOperations.rightPopAndLeftPush();
//        listOperations.remove();
//        listOperations.size();
//        listOperations.index();
//        listOperations.set();
//        listOperations.range();
//        listOperations.trim();

    }

    /**
     *  语法 LPUSH key value [value ...]
     *  将一个或者多个value值顺次插入到列表key的头前.key必须为列表类型,否则报错;如果key列表不存在,则自动创建.key列表的顺序由插入时的顺序决定,并且允许重复插入.
     */
    @Test
    public void testLeftPush(){
        Long result = listOperations.leftPush("ListA","D");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 LPUSH key value [value ...]
     *  将一个或者多个value顺次插入到列表key的头前.key必须为列表类型,否则报错;如果key列表不存在,则自动创建.key列表的顺序由插入时的顺序决定,并且允许重复插入.
     */
    @Test
    public void testLeftPushAll(){
        // 添加多个
        Long result1 = listOperations.leftPushAll("ListA","A","B","C");
        assertNotNull(result1);
        log.info(result1.toString());

        // 通过List添加
        List<String> values = new ArrayList<>();
        values.add("A");
        values.add("B");
        values.add("C");
        Long result2 = listOperations.leftPushAll("ListA",values);
        assertNotNull(result2);
        log.info(result2.toString());
    }

    /**
     *  语法 LPUSHX key value [value ...]
     *  将一个或者多个value值顺次插入到列表key的头前.key必须存在且为列表类型,否则不做任何处理.
     */
    @Test
    public void testLeftPushIfPresent(){
        // ListA 必须存在
        Long result = listOperations.leftPushIfPresent("ListA","A");
        assertNotNull(result);
        log.info(result.toString());
    }


    /**
     *  语法 RPUSH key value [value ...]
     *  将一个或者多个value值顺次追加到列表尾.key必须为列表类型,否则报错.如果key列表不存在,则自动创建.
     */
    @Test
    public void testRightPush(){
        Long result = listOperations.rightPush("ListA","A");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 RPUSH key value [value ...]
     *  将一个或者多个value值顺次追加到列表尾.key必须为列表类型,否则报错.如果key列表不存在,则自动创建.
     */
    @Test
    public void testRightPushAll(){
        // 添加多个
        Long result1 = listOperations.rightPushAll("ListA","A","B","C");
        assertNotNull(result1);
        log.info(result1.toString());

        // 通过List添加
        List<String> values = new ArrayList<>();
        values.add("A");
        values.add("B");
        values.add("C");
        Long result2 = listOperations.rightPushAll("ListA",values);
        assertNotNull(result2);
        log.info(result2.toString());
    }

    /**
     *  语法 RPUSHX key value [value ...]
     *  将一个或者多个value值顺次追加到列表为,key必须存在且为列表类型,否则不做任何修改.
     */
    @Test
    public void testRightPushIfPresent(){
        Long result = listOperations.rightPushIfPresent("ListA","A");
        assertNotNull(result);
        log.info(result.toString());
    }


    /**
     *  语法 LPOP key
     *  将列表头元素删除,并返回.key不存在或者列表为空时,返回nil.
     */
    @Test
    public void testLeftPop(){
        String result = listOperations.leftPop("ListA");
        assertNotNull(result);
        log.info(result);
    }

    /**
     *  语法 RPOP key
     *  将列表尾元素删除,并返回.key不存在或者列表为空时,返回nil.
     */
    @Test
    public void testRightPop(){
        String result = listOperations.rightPop("ListA");
        assertNotNull(result);
        log.info(result);
    }

    /**
     *  语法 RPOPLPUSH source destination
     *  source和destination必须列表类型,命令将source列表的最后一个元素弹出,并返回给客户端.同时,将弹出的元素重新插入到destination列表的第一个位置.
     *  source列表如果不存在,则不进行动作.
     *  destination列表如果不存在,则创建.
     *  source和destination可以为同一个列表,此时命令等同于rotation旋转操作.
     *  整体操作具有原子性,列表修改同时成功或者同时失败.
     */
    @Test
    public void testRightPopAndLeftPush(){
        String result = listOperations.rightPopAndLeftPush("ListA","ListB");
        assertNotNull(result);
        log.info(result);
    }

    /**
     *  语法 LREM key count value
     *  key必须为列表,count必须为整数,其绝对值表示将要从key列表中移除的value的个数,当count为正整数时,表示从左往右移除指定count个与value相同的元素;count为负数时,移除顺序相反;count为0,表示移除全部
     */
    @Test
    public void testRemove(){
        Long result = listOperations.remove("ListA",1L,"A");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 LLEN key
     *  返回类表的长度,如果列表不存在,或者为空列表,则返回0
     */
    @Test
    public void testSize(){
        Long result = listOperations.size("ListA");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 LINDEX key index
     *  返回列表key中,索引位置为index的元素.其中,key必须为列表类型,index为必须为整数,正整数表示索引从前往后,0为列表第一个元素;负整数表示索引从后往前,-1为列表最后一个元素.如果索引超过列表长度,返回nil.
     */
    @Test
    public void testIndex(){
        String result1 = listOperations.index("ListA",0);
        assertNotNull(result1);
        log.info(result1);

        String result2 = listOperations.index("ListB",-1);
        assertNotNull(result2);
        log.info(result2);
    }

    /**
     *  语法 LSET key index value
     *  将列表key的索引位置为index的元素的值替换为value.
     *  越界抛出异常
     */
    @Test
    public void testSet(){
         listOperations.set("ListA",2,"D");
    }

    /**
     *  语法 LRANGE key start stop
     *  返回列表key中,从start到stop区间内的元素,索引从0开始,表示第一个元素;-1表示最后一个元素.start或stop过小或者过大均自动截取列表的极值
     */
    @Test
    public void testRange(){
        List<String> result = listOperations.range("ListA",0,-1);
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 LTRIM key start stop
     *  对列表key进行修剪,只保留start和stop的区间内的元素.start和stop超过列表长度,返回错误.
     */
    @Test
    public void testTrim(){
        listOperations.trim("ListA",0,1);
    }

    /**
     *  语法 BLPOP key [key ...] timeout
     *  阻塞式弹出列表key的列首第一个元素,如过列表为空,等待timeout秒,如果timeout设置为0,表示一直等待到列表具有元素时才进行弹出.
     *  当有多个key时,会根据key的先后顺序,检测每一个列表,直到有一个能弹出列首元素为止.
     *  返回,被弹出的列表名称和弹出的元素
     */
    @Test
    public void testLeftPopB(){
        String result = listOperations.leftPop("ListA",10, TimeUnit.SECONDS);
        assertNotNull(result);
        log.info(result);
    }

    /**
     *  语法 BRPOP key [key ...] timeout
     *  阻塞式弹出列表key的列尾最后一个元素,如过列表为空,等待timeout秒,如果timeout设置为0,表示一直等待到列表具有元素时才进行弹出.
     *  当有多个key时,会根据key的先后顺序,检测每一个列表,直到有一个能弹出列尾元素为止.
     *  返回,被弹出的列表名称和弹出的元素
     */
    @Test
    public void testRightPopB(){
        String result = listOperations.rightPop("ListA",10, TimeUnit.SECONDS);
        assertNotNull(result);
        log.info(result);
    }

    /**
     *  语法 BRPOPLPUSH source destination timeout
     *  将source列表的列尾元素弹出,并插入destination列表的列首,如果source列表为空,则等待timeout秒,timeout为0,则持续等待.
     *  source和destination可以为同一个列表.
     */
    @Test
    public void testRightPopAndLeftPushB(){
        String result = listOperations.rightPopAndLeftPush("ListA","ListA",10,TimeUnit.SECONDS);
        assertNotNull(result);
        log.info(result);
    }
}
```



## `SetOperations` 接口

`SetOperations` 对 `Set` 的操作

### 常用方法

| 声明方法                                                | Redis操作                               | 说明                                                         |
| ------------------------------------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| `Long add(K key, V... values)`                          | `SADD key member [member ...]`          | 添加                                                         |
| `Boolean isMember(K key, Object o)`                     | `SISMEMBER key member`                  | 判断集合是否存在某元素                                       |
| `V pop(K key)`                                          | `SPOP key [count]`                      | 从集合key中随机移除count个成员,并返回.当集合不存在时,返回nil. |
| `V randomMember(K key)`                                 | `SRANDMEMBER key [count]`               | 从集合key中随机获得指定个数的成员.若集合为空,则返回nil       |
| `Set<V> distinctRandomMembers(K key, long count)`       | `SRANDMEMBER key [count]`               | 从集合key中随机获得数量值的成员,如果一个值多次出现则只出现一次,若集合为空,则返回nil |
| `Long remove(K key, Object... values)`                  | `SREM key member [member …]`            | 移除集合key中的一个或者多个成员,不存在的member则忽略.        |
| `Boolean move(K key, V value, K destKey)`               | `SMOVE source destination member`       | 将member成员从source集合删除,并添加到destination集合中.两个集合必须存在 |
| `Long size(K key)`                                      | `SCARD key`                             | 获得集合key中的成员数量.如果集合不存在或者为空,返回0         |
| `Set<V> members(K key)`                                 | `SMEMBERS key`                          | 获得集合中的所有成员,key或者为空,返回错误                    |
| `Set<V> intersect(K key, K otherKey)`                   | `SINTERSTORE destination key [key ...]` | 获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖. |
| `Long intersectAndStore(K key, K otherKey, K destKey)`  | `SINTERSTORE destination key [key ...]` | 获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖. |
| `Set<V> union(K key, K otherKey`                        | `SUNION destination key [key ...]`      | 获得多个key集合共同的并集,并返回集合对象                     |
| `Long unionAndStore(K key, K otherKey, K destKey)`      | `SINTERSTORE destination key [key ...]` | 获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖. |
| `Set<V> difference(K key, K otherKey)`                  | `SDIFF key [key ...]`                   | 获得key集合之间的差集,根据key的顺序依次相减,并返回差集成员.  |
| `Long differenceAndStore(K key, K otherKey, K destKey)` | `SDIFFSTORE destination key [key ...]`  | 获得多个key集合的差集,根据key的顺序相减,并赋值给destination.如果集合destination存在,则覆盖. |

### 方法调用示例

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.Before;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.SetOperations;
import top.jionjion.DataRedisApplicationTest;

import java.util.List;
import java.util.Objects;
import java.util.Set;

import static org.junit.Assert.*;

/**
 * @author Jion
 * 测试Set类
 *  SetOperations 对 Set 的操作
 */
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class SetOperationsTest {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    private SetOperations<String, String> setOperations;

    /**
     * 初始化数据
     */
    @Before
    public void initData() {
        if (Objects.isNull(setOperations)) {
            setOperations = redisTemplate.opsForSet();
        }
        // 初始化数据,启用事物,并提交
        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.multi();
        redisTemplate.delete("SetA");
        redisTemplate.delete("SetB");
        redisTemplate.delete("SetC");
        setOperations.add("SetA","A","B","C");
        setOperations.add("SetB","C","D","E");
        redisTemplate.exec();
    }

    /**
     * 测试SetOperations是否存在
     */
    @Test
    public void testAutowired() {
        assertNotNull(setOperations);
    }


    /**
     * 添加
     * SADD key member [member ...]
     */
    @Test
    public void testAdd() {
        Long result1 = setOperations.add("SetA", "D");
        log.info(String.valueOf(result1));
        assertNotNull(result1);
        Long result2 = setOperations.add("SetA", "E", "F");
        log.info(String.valueOf(result2));
        assertNotNull(result2);
    }

    /**
     * SISMEMBER key member
     * 判断集合是否存在某元素
     */
    @Test
    public void testIsMember() {
        Boolean result = setOperations.isMember("SetA", "C");
        log.info(String.valueOf(result));
        assertEquals(true, result);
    }

    /**
     * SPOP key [count]
     * 从集合key中随机移除count个成员,并返回.当集合不存在时,返回nil.
     */
    @Test
    public void testPop() {
        String result1 = setOperations.pop("SetA");
        log.info(result1);
        assertNotNull(result1);

        List<String> result2 = setOperations.pop("SetA", 2);
        log.info(String.valueOf(result2));
        if (result2 != null) {
            assertNotEquals(0, result2.size());
        }
    }

    /**
     *  SRANDMEMBER key [count]
     *  从集合key中随机获得指定个数的成员.若集合为空,则返回nil
     */
    @Test
    public void testRandomMember() {
        String result1 = setOperations.randomMember("SetA");
        log.info(result1);
        assertNotNull(result1);
        // List 可重复
        List<String> result2 = setOperations.randomMembers("SetA",2);
        if (result2 != null) {
            assertNotEquals(0,result2.size());
        }
        log.info(String.valueOf(result2));
    }

    /**
     *  SRANDMEMBER key [count]
     *  从集合key中随机获得数量值的成员,如果一个值多次出现则只出现一次,若集合为空,则返回nil
     */
    @Test
    public void testDistinctRandomMembers() {
        // Set,不重复
        Set<String> result = setOperations.distinctRandomMembers("SetA",2);
        assertNotEquals(0, result != null ? result.size() : 0);
        log.info(String.valueOf(result));
    }


    /**
     *  SREM key member [member …]
     *  移除集合key中的一个或者多个成员,不存在的member则忽略.
     */
    @Test
    public void testRemove() {
        Long result = setOperations.remove("SetA","A","B");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  SMOVE source destination member
     *  将member成员从source集合删除,并添加到destination集合中.两个集合必须存在.
     */
    @Test
    public void testMove() {
        Boolean result = setOperations.move("SetA","A","SetB");
        log.info(String.valueOf(result));
        assertEquals(true, result);
    }

    /**
     *  SCARD key
     *  获得集合key中的成员数量.如果集合不存在或者为空,返回0
     */
    @Test
    public void testSize() {
        Long result = setOperations.size("SetA");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  SMEMBERS key
     *  获得集合中的所有成员,key或者为空,返回错误
     */
    @Test
    public void testMembers() {
         Set<String> result = setOperations.members("SetA");
         log.info(String.valueOf(result));
         assertNotNull(result);
    }

    @Test
    public void testScan() {
        fail("未测试...");
    }

    /**
     *  SINTERSTORE destination key [key ...]
     *  获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖.
     */
    @Test
    public void testIntersect() {
        Set<String> result = setOperations.intersect("SetA","SetB");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  SINTERSTORE destination key [key ...]
     *  获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖.
     */
    @Test
    public void testIntersectAndStore() {
       Long result = setOperations.intersectAndStore("SetA","SetB","SetC");
       log.info(String.valueOf(result));
       assertNotNull(result);
       assertNotNull(setOperations.size("SetC"));
    }

    /**
     *  SUNION destination key [key ...]
     *  获得多个key集合共同的并集,并返回集合对象
     */
    @Test
    public void testUnion() {
        Set<String> result = setOperations.union("SetA","SetB");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  SINTERSTORE destination key [key ...]
     *  获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖.
     */
    @Test
    public void testUnionAndStore() {
        Long result = setOperations.unionAndStore("SetA","SetB","SetC");
        log.info(String.valueOf(result));
        assertNotNull(result);
        assertNotNull(setOperations.size("SetC"));
    }

    /**
     *  SDIFF key [key ...]
     *  获得key集合之间的差集,根据key的顺序依次相减,并返回差集成员.
     */
    @Test
    public void testDifference() {
        Set<String> result = setOperations.difference("SetA","SetB");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  SDIFFSTORE destination key [key ...]
     *  获得多个key集合的差集,根据key的顺序相减,并赋值给destination.如果集合destination存在,则覆盖.
     */
    @Test
    public void testDifferenceAndStore() {
        Long result = setOperations.differenceAndStore("SetA","SetB","SetC");
        log.info(String.valueOf(result));
        assertNotNull(result);
        assertNotNull(setOperations.size("SetC"));
    }
}
```



## `ZSetOperations` 接口

### 常用方法

| 声明方法                                                     | Redis操作                                                    | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Boolean add(K key, V value, double score)`                  | `ZADD  [NX/XX] [CH] [INCR] score member [score member ...]`  | 将成员member和指定排序顺序score放入有序集合key中             |
| `Double score(K key, Object o)`                              | `ZSCORE key member`                                          | 返回成员member在有序集合key中的顺序.如果key不是有序集合,或者key不存在,返回nil |
| `Double incrementScore(K key, V value, double delta) Double incrementScore(K key, V value, double delta)` | `ZINCRBY key increment member`                               | 为有序集合key中的成员member的order顺序加上增量increment.其中,key必须为有序集合,increment增量可正可负,精度不限. |
| `Long zCard(K key)`                                          | `ZCARD key`                                                  | 返回有序集合key的长度                                        |
| `Long count(K key, double min, double max)`                  | `ZCOUNT key min max`                                         | 获得有序集合key中,排序值在min和max闭区间内的成员数量.min和max必须为数字,大小不限制. |
| `Set<V> range(K key, long start, long end)`                  | `ZRANGE key start stop [WITHSCORES]`                         | 获得有序集合key的指定索引区间内的成员.索引从0开始,-1表示最后一个.当start和stop超出索引边界后,只回返回对应边界的极值所在的值.可选参数withscores表示返回对应成员的排序大小. 返回值,按照排序值,从小到大排列.相同排序值成员根据字典顺序排序. |
| `Set<TypedTuple<V>> rangeWithScores(K key, long start, long end)` | `ZRANGE key start stop [WITHSCORES]`                         | 获得有序集合key的指定索引区间内的成员.索引从0开始,-1表示最后一个.当start和stop超出索引边界后,只回返回对应边界的极值所在的值.可选参数withscores表示返回对应成员的排序大小. 返回值,按照排序值,从小到大排列.相同排序值成员根据字典顺序排序. |
| `Set<V> reverseRange(K key, long start, long end)`           | `ZREVRANGE key start stop [WITHSCORES]`                      | 获得有序集合key的指定索引区间内的成员.索引从0开始,-1表示最后一个. 当start和stop超出索引边界后,只回返回对应边界的极值所在的值.可选参数withscores表示返回对应成员的排序大小.  返回值,按照排序值,从大到小排列.相同排序值成员根据字典顺序排序. |
| `Set<TypedTuple<V>> reverseRangeWithScores(K key, long start, long end)` | `ZREVRANGE key start stop [WITHSCORES]`                      | 获得有序集合key的指定索引区间内的成员.索引从0开始,-1表示最后一个. 当start和stop超出索引边界后,只回返回对应边界的极值所在的值.可选参数withscores表示返回对应成员的排序大小.  返回值,按照排序值,从大到小排列.相同排序值成员根据字典顺序排序. |
| `Set<V> rangeByScore(K key, double min, double max)`         | `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` | 获得有序集合key中,所有的成员的排序值在min和max闭区间内的成员. 可以通过(,)限制区间为开区间. 返回结果根据排序值从小到大排列,相同的成员根据字典顺序排列. |
| `Set<TypedTuple<V>> rangeByScoreWithScores(K key, double min, double max)` | `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` | 获得有序集合key中,所有的成员的排序值在min和max闭区间内的成员. 可以通过(,)限制区间为开区间. 返回结果根据排序值从小到大排列,相同的成员根据字典顺序排列. |
| `Set<V> reverseRangeByScore(K key, double min, double max)`  | `ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]` | 获得有序集合key中,所有的成员的排序值在min和max闭区间内的成员. 可以通过(,)限制区间为开区间. 返回结果根据排序值从大到大小排列,相同的成员根据字典顺序排列. |
| `Set<TypedTuple<V>> reverseRangeByScoreWithScores(K key, double min, double max)` | `ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]` | 获得有序集合key中,所有的成员的排序值在min和max闭区间内的成员. 可以通过(,)限制区间为开区间. 返回结果根据排序值从大到大小排列,相同的成员根据字典顺序排列. |
| `Long rank(K key, Object o)`                                 | `ZRANK key member`                                           | 获得有序集合key的成员member的排名位置.排序规则为排序值从小到大排列.排名从0开始.如果成员member不存在,则返回nil |
| `Long reverseRank(K key, Object o)`                          | `ZREVRANK key member`                                        | 获得有序集合key的成员member的排名位置.排序规则为排序值从大到小排列.排名从0开始.如果成员member不存在,则返回nil |
| `Long remove(K key, Object... values)`                       | `ZREM key member [member …]`                                 | 移除有序集合key中一个或者多个成员member,若该成员不存在,则忽略.  返回被移除的成员个数 |
| `Long removeRange(K key, long start, long end)`              | ZREMRANGEBYRANK key start stop                               | 移除有序集合key中指定排名rank在start和stop闭区间内的所有成员.start从0开始,表示第一个元素;-1表示最后一个元素. 返回被移除成员的数量 |
| `Long removeRangeByScore(K key, double min, double max)`     | `ZREMRANGEBYSCORE key min max`                               | 移除有序集合key中排序值score在min和max闭区间内的所有成员     |
| `Set<V> rangeByLex(K key, Range range)`                      | `ZRANGEBYLEX key min max [LIMIT offset count]`               | 当有序集合key具有相同的排序值score时,根据字母的先后顺序进行排列,命令检索min字母到max字母之间的成员.  必须使用区间限定符,(和)表示开区间;[和]表示闭区间;+和-表示极大值和极小值 |
| `Long unionAndStore(K key, K otherKey, K destKey)`           | `ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM/MIN/MAX]` | 集合并集并保存 ,返回新集合的长度                             |
| `Long intersectAndStore(K key, K otherKey, K destKey)`       | `ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM/MIN/MAX]` | 集合交集并保存 返回新集合的长度                              |



### 方法调用示例

```java
package top.jionjion.cache;

import lombok.extern.slf4j.Slf4j;
import org.junit.Before;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.SetOperations;
import top.jionjion.DataRedisApplicationTest;

import java.util.List;
import java.util.Objects;
import java.util.Set;

import static org.junit.Assert.*;

/**
 * @author Jion
 * 测试Set类
 *  SetOperations 对 Set 的操作
 */
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class SetOperationsTest {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    private SetOperations<String, String> setOperations;

    /**
     * 初始化数据
     */
    @Before
    public void initData() {
        if (Objects.isNull(setOperations)) {
            setOperations = redisTemplate.opsForSet();
        }
        // 初始化数据,启用事物,并提交
        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.multi();
        redisTemplate.delete("SetA");
        redisTemplate.delete("SetB");
        redisTemplate.delete("SetC");
        setOperations.add("SetA","A","B","C");
        setOperations.add("SetB","C","D","E");
        redisTemplate.exec();
    }

    /**
     * 测试SetOperations是否存在
     */
    @Test
    public void testAutowired() {
        assertNotNull(setOperations);
    }


    /**
     * 语法 SADD key member [member ...]
     * 添加
     */
    @Test
    public void testAdd() {
        Long result1 = setOperations.add("SetA", "D");
        log.info(String.valueOf(result1));
        assertNotNull(result1);
        Long result2 = setOperations.add("SetA", "E", "F");
        log.info(String.valueOf(result2));
        assertNotNull(result2);
    }

    /**
     * 语法 SISMEMBER key member
     * 判断集合是否存在某元素
     */
    @Test
    public void testIsMember() {
        Boolean result = setOperations.isMember("SetA", "C");
        log.info(String.valueOf(result));
        assertEquals(true, result);
    }

    /**
     * 语法 SPOP key [count]
     * 从集合key中随机移除count个成员,并返回.当集合不存在时,返回nil.
     */
    @Test
    public void testPop() {
        String result1 = setOperations.pop("SetA");
        log.info(result1);
        assertNotNull(result1);

        List<String> result2 = setOperations.pop("SetA", 2);
        log.info(String.valueOf(result2));
        if (result2 != null) {
            assertNotEquals(0, result2.size());
        }
    }

    /**
     *  语法 SRANDMEMBER key [count]
     *  从集合key中随机获得指定个数的成员.若集合为空,则返回nil
     */
    @Test
    public void testRandomMember() {
        String result1 = setOperations.randomMember("SetA");
        log.info(result1);
        assertNotNull(result1);
        // List 可重复
        List<String> result2 = setOperations.randomMembers("SetA",2);
        if (result2 != null) {
            assertNotEquals(0,result2.size());
        }
        log.info(String.valueOf(result2));
    }

    /**
     *  语法 SRANDMEMBER key [count]
     *  从集合key中随机获得数量值的成员,如果一个值多次出现则只出现一次,若集合为空,则返回nil
     */
    @Test
    public void testDistinctRandomMembers() {
        // Set,不重复
        Set<String> result = setOperations.distinctRandomMembers("SetA",2);
        assertNotEquals(0, result != null ? result.size() : 0);
        log.info(String.valueOf(result));
    }


    /**
     *  语法 SREM key member [member …]
     *  移除集合key中的一个或者多个成员,不存在的member则忽略.
     */
    @Test
    public void testRemove() {
        Long result = setOperations.remove("SetA","A","B");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  语法 SMOVE source destination member
     *  将member成员从source集合删除,并添加到destination集合中.两个集合必须存在.
     */
    @Test
    public void testMove() {
        Boolean result = setOperations.move("SetA","A","SetB");
        log.info(String.valueOf(result));
        assertEquals(true, result);
    }

    /**
     *  语法 SCARD key
     *  获得集合key中的成员数量.如果集合不存在或者为空,返回0
     */
    @Test
    public void testSize() {
        Long result = setOperations.size("SetA");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  语法 SMEMBERS key
     *  获得集合中的所有成员,key或者为空,返回错误
     */
    @Test
    public void testMembers() {
         Set<String> result = setOperations.members("SetA");
         log.info(String.valueOf(result));
         assertNotNull(result);
    }

    @Test
    public void testScan() {
        fail("未测试...");
    }

    /**
     *  语法 SINTERSTORE destination key [key ...]
     *  获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖.
     */
    @Test
    public void testIntersect() {
        Set<String> result = setOperations.intersect("SetA","SetB");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  语法 SINTERSTORE destination key [key ...]
     *  获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖.
     */
    @Test
    public void testIntersectAndStore() {
       Long result = setOperations.intersectAndStore("SetA","SetB","SetC");
       log.info(String.valueOf(result));
       assertNotNull(result);
       assertNotNull(setOperations.size("SetC"));
    }

    /**
     *  语法 SUNION destination key [key ...]
     *  获得多个key集合共同的并集,并返回集合对象
     */
    @Test
    public void testUnion() {
        Set<String> result = setOperations.union("SetA","SetB");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  语法 SINTERSTORE destination key [key ...]
     *  获得多个key集合之间的交集,并将结果赋值给集合destination.如果集合destination存在,则覆盖.
     */
    @Test
    public void testUnionAndStore() {
        Long result = setOperations.unionAndStore("SetA","SetB","SetC");
        log.info(String.valueOf(result));
        assertNotNull(result);
        assertNotNull(setOperations.size("SetC"));
    }

    /**
     *  语法 SDIFF key [key ...]
     *  获得key集合之间的差集,根据key的顺序依次相减,并返回差集成员.
     */
    @Test
    public void testDifference() {
        Set<String> result = setOperations.difference("SetA","SetB");
        log.info(String.valueOf(result));
        assertNotNull(result);
    }

    /**
     *  语法 SDIFFSTORE destination key [key ...]
     *  获得多个key集合的差集,根据key的顺序相减,并赋值给destination.如果集合destination存在,则覆盖.
     */
    @Test
    public void testDifferenceAndStore() {
        Long result = setOperations.differenceAndStore("SetA","SetB","SetC");
        log.info(String.valueOf(result));
        assertNotNull(result);
        assertNotNull(setOperations.size("SetC"));
    }
}
```



## `HashOperations` 接口

`HashOperations` 对 `Hash` 的操作

### 常用方法

| 声明方法                                                | Redis操作                                 | 说明                                                         |
| ------------------------------------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| `void put(H key, HK hashKey, HV value)`                 | `HSET hash field value`                   | 其中,hash为字符类型,代表哈希表的名字,field为域,同为字符类型,value为域中存放的值,类型不限 .当不存这个hash或者field时,则自动创建;field重复时,则更新value. |
| `void putAll(H key, Map<? extends HK, ? extends HV> m)` | `HMSET key field value [field value ...]` | 为key哈希表中,创建多个field域并赋值.如果field域已经存在,则覆盖值;如果field域不存在,则创建并赋值 |
| `HV get(H key, Object hashKey)`                         | `HGET hash field`                         | 其中,hash为字符类型,为已存在的哈希表,field字符类型,哈希表对应的域.命令为获得指定哈希表中的字段的值. |
| `Boolean hasKey(H key, Object hashKey)`                 | `HEXISTS hash field`                      | 检查hash哈希表中是否含有field域.其中,hash字符类型,哈希表的名字.field字符类型. |
| `Long delete(H key, Object... hashKeys)`                | `HDEL key field [field ...]`              | 删除key对应哈希表的一个或者多个field域.key字符类型,field字符类型  如果field域不存在,不影响 |
| `Long size(H key)`                                      | `HLEN key`                                | 获得当前哈希表中的域值数量                                   |
| `Long lengthOfValue(H key, HK hashKey)`                 | `HSTRLEN key field`                       | 获得key哈希表中field域的字符串的值的长度                     |
| `Long increment(H key, HK hashKey, long delta)`         | `HINCRBY key field increment`             | 为key哈希表中的field域的值加上增量increment,其中field对应的值必须为整数,不限正负,若不存在则创建,increment也必须为整数,正数为加上增量,负数为减去增量 |
| `Double increment(H key, HK hashKey, double delta)`     | `HINCRBYFLOAT key field increment`        | 为key哈希表中的field域加上增量increment,其中field对应的值必须为数字,不限制正负数.increment也必须为数字.程序会为运算后的结果自动抹去尾零,并以数字字符串的形式存入. |
| `Boolean putIfAbsent(H key, HK hashKey, HV value)`      | `HMSET key field value [field value ...]` | 为key哈希表中,创建多个field域并赋值.如果field域已经存在,则覆盖值;如果field域不存在,则创建并赋值 |
| `List<HV> multiGet(H key, Collection<HK> hashKeys)`     | `HMGET key field [field ...]`             | 获得key哈希表中的多个field的值.如果field值不存在,则返回nil.  |
| `Set<HK> keys(H key)`                                   | `HKEYS key`                               | 获得key哈希表中所有的field的信息.key不存在时,返回空集合.     |
| `List<HV> values(H key)`                                | `HVALS key`                               | 获得key哈希表中的所有域的值.key不存在时,返回空集合.          |
| `Map<HK, HV> entries(H key)`                            | `HVALS key`                               | 获得key哈希表中的所有域的值.key不存在时,返回空集合.          |

###  方法调用示例

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.Before;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.HashOperations;
import org.springframework.data.redis.core.RedisTemplate;
import top.jionjion.DataRedisApplicationTest;

import java.util.*;

import static org.junit.Assert.*;

/**
 * @author Jion
 *  测试 Hash
 *      HashOperations 对Hash的操作
 */
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class HashOperationsTest{

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    private HashOperations<String,String,String> hashOperations;

    @Before
    public void initOperations(){
        if (Objects.isNull(hashOperations)){
            hashOperations = redisTemplate.opsForHash();
        }
        // 初始化数据,启用事物,并提交
        redisTemplate.setEnableTransactionSupport(true);
        redisTemplate.multi();
        redisTemplate.delete("HashA");
        hashOperations.put("HashA","zero","0");
        hashOperations.put("HashA","a","Aa");
        hashOperations.put("HashA","b","Bb");
        hashOperations.put("HashA","c","Cc");
        redisTemplate.exec();

//        BoundHashOperations 类方法与其一致.
//        hashOperations.put();
//        hashOperations.putAll();
//        hashOperations.putIfAbsent();
//        hashOperations.get();
//        hashOperations.hasKey();
//        hashOperations.delete();
//        hashOperations.size();
//        hashOperations.lengthOfValue();
//        hashOperations.increment();
//        hashOperations.increment();
//        hashOperations.multiGet();
//        hashOperations.keys();
//        hashOperations.values();
//        hashOperations.entries();
//        hashOperations.scan();
    }

    /**
     * 语法 HSET hash field value
     * 其中,hash为字符类型,代表哈希表的名字,field为域,同为字符类型,value为域中存放的值,类型不限
     * 当不存这个hash或者field时,则自动创建;field重复时,则更新value.
     */
    @Test
    public void testPut(){
        hashOperations.put("HashA","d","Dd");
    }

    /**
     *  语法 HMSET key field value [field value ...]
     *  为key哈希表中,创建多个field域并赋值.如果field域已经存在,则覆盖值;如果field域不存在,则创建并赋值
     */
    @Test
    public void testPutAll(){
        Map<String,String> map = new HashMap<>();
        map.put("d","Dd");
        map.put("e","Ee");
        hashOperations.putAll("HashA",map);
    }

    /**
     *  语法 HGET hash field
     *  其中,hash为字符类型,为已存在的哈希表,field字符类型,哈希表对应的域.命令为获得指定哈希表中的字段的值.
     */
    @Test
    public void testGet(){
        String result = hashOperations.get("HashA","a");
        assertNotNull(result);
        log.info(result);
    }

    /**
     *  语法 HEXISTS hash field
     *  检查hash哈希表中是否含有field域.其中,hash字符类型,哈希表的名字.field字符类型.
     */
    @Test
    public void testHasKey(){
        Boolean result = hashOperations.hasKey("HashA","a");
        assertTrue(result);
        log.info(result.toString());
    }

    /**
     *  语法 HDEL key field [field ...]
     *  删除key对应哈希表的一个或者多个field域.key字符类型,field字符类型
     *  如果field域不存在,不影响
     */
    @Test
    public void testDelete(){
        Long result = hashOperations.delete("HashA","a","b");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 HLEN key
     *  获得当前哈希表中的域值数量.
     */
    @Test
    public void testSize(){
        Long result = hashOperations.size("HashA");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 HSTRLEN key field
     *  获得key哈希表中field域的字符串的值的长度.
     */
    @Test
    public void testLengthOfValue(){
        Long result = hashOperations.lengthOfValue("HashA","a");
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 HINCRBY key field increment
     *  为key哈希表中的field域的值加上增量increment,其中field对应的值必须为整数,不限正负,若不存在则创建,increment也必须为整数,正数为加上增量,负数为减去增量
     */
    @Test
    public void testIncrementLong(){
        Long result = hashOperations.increment("HashA","zero",100L);
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 HINCRBYFLOAT key field increment
     *  为key哈希表中的field域加上增量increment,其中field对应的值必须为数字,不限制正负数.increment也必须为数字.程序会为运算后的结果自动抹去尾零,并以数字字符串的形式存入.
     */
    @Test
    public void testIncrementDouble(){
        Double result = hashOperations.increment("HashA","zero",3.1415926D);
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 HMSET key field value [field value ...]
     *  为key哈希表中,创建多个field域并赋值.如果field域已经存在,则覆盖值;如果field域不存在,则创建并赋值
     */
    @Test
    public void testPutIfAbsent(){
        Boolean result = hashOperations.putIfAbsent("HashA","d","Dd");
        log.info(result.toString());
        assertTrue(result);
    }

    /**
     *  语法 HMGET key field [field ...]
     *  获得key哈希表中的多个field的值.如果field值不存在,则返回nil.
     */
    @Test
    public void testMultiGet(){
        Set<String> hashKeys = new HashSet<>();
        hashKeys.add("a");
        hashKeys.add("b");
        List<String> result = hashOperations.multiGet("HashA",hashKeys);
        assertNotNull(result);
        log.info(result.toString());
    }

    /**
     *  语法 HKEYS key
     *  获得key哈希表中所有的field的信息.key不存在时,返回空集合.
     */
    @Test
    public void testKeys(){
        Set<String> hashKeys = hashOperations.keys("HashA");
        assertNotNull(hashKeys);
        log.info(hashKeys.toString());
    }

    /**
     *  语法 HVALS key
     *  获得key哈希表中的所有域的值.key不存在时,返回空集合.
     */
    @Test
    public void testValues(){
        List<String> hashValues = hashOperations.values("HashA");
        assertNotNull(hashValues);
        log.info(hashValues.toString());
    }

    /**
     *  语法 HVALS key
     *  获得key哈希表中的所有域的值.key不存在时,返回空集合.
     */
    @Test
    public void testEntries(){
        Map<String, String> entries = hashOperations.entries("HashA");
        assertNotNull(entries);
        log.info(entries.toString());
    }

    @Test
    public void testScan(){
        fail("暂未处理...");
    }
}
```

# 相关配置

## 序列化配置

支持序列化汉字, `Key` 多结构

```java
@Configuration
public class RedisCacheConfig {

    /**
     * 配置 RedisTemplate
     *
     * @param factory 配置工厂
     * @return RedisTemplate
     */
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate redisTemplate = new RedisTemplate();
        RedisSerializer stringSerializer = new StringRedisSerializer();
        redisTemplate.setConnectionFactory(factory);
        redisTemplate.setKeySerializer(stringSerializer);
        redisTemplate.setValueSerializer(stringSerializer);
        redisTemplate.setHashKeySerializer(stringSerializer);
        redisTemplate.setHashValueSerializer(stringSerializer);
        return redisTemplate;
    }
}
```

