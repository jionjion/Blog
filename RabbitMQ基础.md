---
title: RabbitMQ基础
date: 2020-11-07 13:26:44
categories:
  - RabbitMQ
tags: [RabbitMQ]
---

# 简介

使用 `AMQP` 协议的软件, 支持多种场景, 事务一致性, 持久化消息.

[官网](https://www.rabbitmq.com/)
[中文翻译](http://rabbitmq.mr-ping.com/)



## `AMQP` 协议模型

生产者和消费者之间通过 `RabbitMQ` 进行消息交换

生产者: 通过通道 `channel` 的形式进行转换, 将信息发送到交换或直接将消息发送到队列中.
每个生产者对应一个虚拟主机, 便于相互隔离.

虚拟主机: 与用户相关联, 将在具体使用时, 需要对应访问权限.

消费者: 从队列接收消息.

![image-20201107170450734](./RabbitMQ基础/AMQP.png)



## 消息策略

[参阅](https://www.rabbitmq.com/getstarted.html)

- 点对点..  生产者直接发送到队列
- 广播
- 发布-订阅
- 路由
- 动态路由
- `RPC`
- 发布确认



## `Windows` 安装

首先安装依赖 `Erlang` 语言, 然后在安装 `RebbitMQ` 服务.. 

[参考](https://www.cnblogs.com/vaiyanzi/p/9531607.html)



## 启用管理控制插件

启用网页端管理控制插件

`rabbitmq-plugins enable rabbitmq_management`

访问浏览器页面 http://localhost:15672 , 默认用户/密码为 `guest/guest`

查看当前所有插件

`rabbitmq-plugins list`

## 应用场景

- 异步处理, 将系统不重要的操作通过消息队列异步操作完成.
- 多系统间调用, 通过队列的方式进行多系统间调用. 避免因为系统间依赖导致问题.
- 流量削峰. 如果请求过多,首先将请求写入消息队列中,随后再由系统慢慢执行.

# 基础操作

默认配置文件路径 `c:/Users/[user]/AppData/Roaming/RabbitMQ/advanced.config`

## 服务操作

```bash
# 启动
rabbitmq-service start   
# 关闭
rabbitmq-service stop
# 禁用服务
rabbitmq-service disable 
# 启用服务
rabbitmq-service enable
```

## 控制操作

使用 `rabbitmqctl` 进行命令行控制操作.可以进行

- 节点
- 集群
- 副本集
- 用户
- 访问控制
- 虚拟主机
- 环境信息
- ....



## 插件命令

使用 `rabbitmq-plugins`  对插件进行管理

```bash
# 停用插件
rabbitmq-plugins disable [插件名]
# 启用插件
rabbitmq-plugins enable [插件名] 
# 查询插件
rabbitmq-plugins list [插件名]
# 设置插件
rabbitmq-plugins set [插件名]
```



## `Web` 管理页面

默认位置 `http://localhost:15672/` ,登录用户密码 `guest / guest`



### 创建虚拟主机

在 `Admin` 选项卡下, 创建虚拟主机. **主机命名以 `/` 开头**

![创建虚拟主机](./RabbitMQ基础/创建虚拟主机.png)



### 创建用户并分配虚拟主机

在 `Admin` 选项卡下, 创建用户. 建议用户名与虚拟主机一致, 同时设置密码和权限

![创建用户](./RabbitMQ基础/创建用户.png)

点击创建后的用户名, 进入详细页面.并分配主机

![分配主机](./RabbitMQ基础/分配主机.png)



# `Java` 调用

# 环境准备

`POM` 文件如下

```java

```



## 点对点模型

生产者直接将消息发送到消费者, 消费者持续等待并处理获取消息.



## 工作队列模型

生产者提供消息到队列, 多个消费者同时读取这个队列.共同处理消息

默认采用轮询的消息确认机制平均分配消息者.



## 广播队列模型

生产者提供消息到交换机.
交换机将消息发送到不同的临时队列.
消费者绑定临时队列, 监听消息

实现一条消息被多个消费者消费



## 路由模式 (订阅)

定制路由规则, 针对不同的消息分发到不同的交换机.

生产者提供消息到交换机,并指定路由关键字
交换机根据消息规则发送到不同的临时队列.
消费者绑定临时队列,指定路由关键字, 监听消息.

如果生产者的消息没有绑定路由关键字, 则不会被绑定关键字的消费者接收



## 动态路由模式 (发布 - 订阅)

支持通配符的方式绑定路由关键字.

建议路由命名以 `.` 作分割符号, 便于使用通配符, 匹配**单词** `*` , 多个**单词** `#`





## 远程调用模式 (`RPC`)

