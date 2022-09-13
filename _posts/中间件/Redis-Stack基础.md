---
title: Redis-Stack基础
categories:
  - Redis
tags:
  - Redis
abbrlink: 6c9fbf13
date: 2022-08-19 09:05:13
---

> Redis-Stack 的技术基础, 提供了对 Redis 的更多操作

<!-- more -->

# Redis Stack

## 快速安装

### 版本选择

- `redis/redis-stack` 包含Redis Stack服务器和RedisInsight。这个容器最适合本地开发，因为可以使用嵌入的RedisInsight来可视化数据。
- `redis/redis-stack-server` 仅提供Redis Stack服务器。此容器最适合生产部署。

### 安装命令

安装测试环境服务器

```bash
docker run -d --name my-redis-stack -p 6379:6379 -p 8001:8001 -v /V/data/redis-stack/data/:/data -v /V/data/redis-stack/data/redis-stack.conf:/redis-stack.conf redis/redis-stack:latest
```

- `6379 ` 端口为 Redis 服务对外端口
- `8001` 端口为 RedisInsight 工具, 可以通过将浏览器指向 [localhost:8001](http://localhost:8001/)
- `/V/data/redis-stack/data/redis-stack.conf` 为 Windows 电脑V盘下的文件夹, 其中有 redis-stack 的配置文件



安装生成环境服务器

```bash
docker run -d --name my-redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```



### 环境变量

要传入任意配置更改，您可以设置以下任何环境变量：

- `REDIS_ARGS`: Redis 的额外参数
- `REDISEARCH_ARGS`: RediSearch 的参数
- `REDISJSON_ARGS`: RedisJSON 的参数
- `REDISGRAPH_ARGS`: RedisGraph 的参数
- `REDISTIMESERIES_ARGS`: RedisTimeSeries 的参数
- `REDISBLOOM_ARGS`: RedisBloom 的参数

例如，以下是如何使用`REDIS_ARGS`环境变量将`requirepass`指令传递给 Redis：

```bash
docker run -e REDIS_ARGS="--requirepass redis-stack" redis/redis-stack:latest
```

以下是为 RedisTimeSeries 设置保留策略的方法：

```bash
docker run -e REDISTIMESERIES_ARGS="RETENTION_POLICY=20" redis/redis-stack:latest
```



## 模块大纲

| 模块            | 说明                                                 |
| --------------- | ---------------------------------------------------- |
| RediSearch      | 全文检索                                             |
| RedisJSON       | JSON 结构支持                                        |
| RedisGraph      | 一个图数据库，使用稀疏邻接矩阵的基于cypphr的查询语言 |
| RedisTimeSeries | Redis的时间序列数据结构                              |
| RedisBloom      | 可伸缩的布隆过滤器                                   |



## 模块 - RedisJSON

实现对 `JSON` 的批量操作

### 命令列表

| 命令           | 语法                                              | 示例                                                         | 说明                                |
| -------------- | ------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------- |
| JSON.SET       | JSON.SET key json                                 | JSON.SET animal $ '"dog"'                                    | 赋值字符串对象                      |
| JSON.GET       | JSON.GET key $                                    | JSON.SET animal $                                            | 获取字符串对象                      |
| JSON.TYPE      | JSON.TYPE keey $                                  | JSON.TYPE animal $                                           | 获取对象类型, 返回 `string` 类型    |
| JSON.STRLEN    | JSON.STRLEN key $                                 | JSON.STRLEN animal $                                         | 获取字符串对象的长度                |
| JSON.STRAPPEND | JSON.STRAPPEND key $ value                        | JSON.STRAPPEND animal $ '" (Canis familiaris)"'              | 为字符串添加内容                    |
|                |                                                   |                                                              |                                     |
| JSON.SET       | JSON.SET key $ 0                                  | JSON.SET num $ 0                                             | 赋值数字                            |
| JSON.NUMINCRBY | JSON.NUMINCRBY key $ num                          | JSON.NUMINCRBY num $ 1                                       | 数字自增 +1                         |
| JSON.NUMMULTBY | JSON.NUMMULTBY key $ num                          | JSON.NUMMULTBY num $ 100                                     | 数字自乘 X100                       |
|                |                                                   |                                                              |                                     |
| JSON.SET       | JSON.SET key $ json                               | JSON.SET example $ '[true,{"answer":42},null]'               | 设置json对象                        |
| JSON.GET       | JSON.GET key $                                    | JSON.GET example $                                           | 获取json对象                        |
| JSON.GET       | JSON.GET key $ path                               | JSON.GET example $[1].answer                                 | 获取json对象中path路径下的值        |
| JSON.DEL       | JSON.DEL key $[index]                             | JSON.DEL example $[-1]                                       | 删除json对象中指定索引的属性        |
|                |                                                   |                                                              |                                     |
| JSON.SET       | JSON.SET array $ []                               | JSON.SET arr $ []                                            | 设置数组对象                        |
| JSON.GET       | JSON.GET array $                                  | JSON.GET arr $                                               | 获取数组对象                        |
| JSON.ARRAPPEND | JSON.ARRAPPEND array $ value                      | JSON.ARRAPPEND arr $ 0                                       | 为数组对象追加元素 `0`              |
| JSON.ARRINSERT | JSON.ARRINSERT array $ index value1 value2 value3 | JSON.ARRINSERT arr $ 0 -2 -1                                 | 在数组0位置后中添加元素 `-2` , `-1` |
| JSON.ARRTRIM   | JSON.ARRTRIM array $ start end                    | JSON.ARRTRIM arr $ 1 2                                       | 截取数组中, 索引在[1,2]的元素       |
| JSON.ARRPOP    | JSON.ARRPOP array $                               | JSON.ARRPOP arr $                                            | 弹出数组中的最后一个元素            |
|                |                                                   |                                                              |                                     |
| JSON.SET       | JSON.SET key  json                                | JSON.SET obj $ '{"name":"Leonard Cohen","lastSeen":1478476800,"loggedOut": true}' | 设置json对象                        |
| JSON.OBJLEN    | JSON.OBJLEN key $                                 | JSON.OBJLEN obj $                                            | json对象的长度                      |
| JSON.OBJKEYS   | JSON.OBJKEYS key $                                | JSON.OBJKEYS obj $                                           | json对象的属性名称集合              |





## 模块 - RedisSearch

### 索引 `HASH`

#### 创建索引

使用 `FT.CREATE 索引名 ON 存储类型 PREFIX 1 前缀 SCHEMA  属性 TEXT` 进行创建

```bash
127.0.0.1:6379> FT.CREATE myIdx ON HASH PREFIX 1 user: SCHEMA name TEXT WEIGHT 5.0 address TEXT phone TEXT
OK
```

创建索引 `myIdx` 在 `HASH` 哈希表上且前缀为 `user:` , 对哈希表中的 `name` 字段索引, 权重为 5,  `address` 和 `phone` 字段文本索引, 默认权重为 1



#### 添加数据

```bash
HSET user:1 name "Jion" address "上海" phone "15512344321"
HSET user:2 name "Arise" address "上海" phone "19556788765"
```



#### 使用索引

在创建过索引的哈希表上通过索引查询. 通过 `FT.SEARCH` 命令, 检索整个索引.. 

```bash
127.0.0.1:6379> FT.SEARCH myIdx "Jion" LIMIT 0 10
1) (integer) 1
2) "user:1"
3) 1) "name"
   2) "Jion"
   3) "address"
   4) "\xe4\xb8\x8a\xe6\xb5\xb7"
   5) "phone"
   6) "15512344321"

127.0.0.1:6379> FT.SEARCH myIdx "上海" LIMIT 0 10
1) (integer) 2
2) "user:1"
3) 1) "name"
   2) "Jion"
   3) "address"
   4) "\xe4\xb8\x8a\xe6\xb5\xb7"
   5) "phone"
   6) "15512344321"
4) "user:2"
5) 1) "name"
   2) "Arise"
   3) "address"
   4) "\xe4\xb8\x8a\xe6\xb5\xb7"
   5) "phone"
   6) "19556788765"
```

使用 `FT.SEARCH` 命令检索索引 `myIdx` , 命中 `Jion` 和 `上海` 关键字, 并限制查询分页... 返回具体的 `key`  和 内容



#### 删除索引

使用 `FT.DROPINDEX` 命令删除索引, 如果添加 `DD` 参数, 级联删除对应的源数据

```bash
127.0.0.1:6379> FT.DROPINDEX myIdx DD
OK
```

删除索引 `myIdx` 及对应的源数据



#### 自动索引

使用 `FT.SUGADD` 添加自动索引, 通过 `FT.SUGGET` 去检索..

```bash
127.0.0.1:6379> FT.SUGADD myAutocomplete "hello world" 100
(integer) 1
127.0.0.1:6379> FT.SUGGET myAutocomplete "he" 
1) "hello world"
```

设置自动索引 `myAutocomplete`  和通过 `myAutocomplete` 索引进行模糊匹配



### 索引 `JSON`

