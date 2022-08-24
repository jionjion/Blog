---
title: Redis-Stack基础
date: 2022-08-19 09:05:13
categories:
  - Redis
tags: [Redis]
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

实现对JSON的批量操作

