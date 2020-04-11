---
title: Docker基础-Redis容器化
categories:
  - 服务器
  - Docker
tags:
  - Linux
  - Docker
  - Redis
abbrlink: c1accc53
date: 2020-02-25 10:36:29
---



# 简介

通过 `Docker` 容器化 `Redis` 数据库

[容器官网](https://hub.docker.com/_/redis/)



## 镜像安装

执行命令,下载最新的容器版本

`docker pull redis`



## 容器操作

### 启动容器

启动容器,并为 `--name`  容器命名 `local-redis` , `-d` 表示后台运行. `redis` 为刚下载的镜像

`docker run --name local-redis -d redis`

容器默认在 `6379` 端口开放

容器数据券在 `/data` 目录下



### 进入控制台

执行 `exec` 命令,  `-it` 交互界面 启动容器为 `local-redis` 的 `redis-cli` 程序,进入 `redis` 操作界面

`docker exec -it local-redis redis-cli`



### 远程连接

修改配置文件,开启远程连接,并通过命令访问

` redis-cli -h 127.0.0.1 -p 6379`



### 维护容器

开启

` docker start local-redis`

关闭

` docker stop local-redis`

重启

`docker stop local-redis`

###创建用后即销毁的容器

启动容器, 在交互界面结束后,销毁容器

`docker run --name my-redis --rm redis:latest`

### 数据持久化

将镜像内 `/data` 文件夹对外挂载

### 生成环境

`docker run --name my-redis -d -p6379:6379 redis:latest`



### 设置配置信息

```bash
FROM redis
COPY redis.conf /usr/local/etc/redis/redis.conf
CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
```





