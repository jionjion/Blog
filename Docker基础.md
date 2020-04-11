---
title: Docker基础
abbrlink: 1d9fbb6a
date: 2020-03-02 20:21:37
categories:
tags:
---



# 常用操作

## 镜像创建

通过 `Dockerfile` 创建镜像;在当前文件夹下,执行, 会将 `Dockerfile` 中定义好的步骤进行搭建镜像

`docker builder` 



存在多个 `tag` 下删除指定版本的镜像

` docker rmi 仓库名:版本`



## 创建容器

创建一次性的容器,程序结束容器销毁

`docker run -it --name 容器的名字 --rm -p 虚拟机端口:本地端口 镜像ID:[版本] /bin/bash`



创建容器,限制内存大小. `--memory` 限制内存(单位 `G`,`M`, `KB`), `--memory-swap` 限制内存+交换分区大小

`docker run --memory 150M --memory-swap=200M 镜像ID `



正式环境运行示例

` docker run -d  -p 8761:8761 --name eureka-server-0.0.1 --memory 170M --memory-swap=200M  b632ece7e92a`



## 推送阿里云仓库

登录,输入用户名,随后输入密码

`docker login --username=阿里云用户名 阿里云分配的仓库地址`

为将要推送的镜像搭上仓库和标签

`docker tag 镜像ID 阿里云分配的仓库地址:版本标签`

推送仓库和对应版本

`docker push 阿里云分配的仓库地址:版本标签`



## 查看当前容器占用资源

1.查询当前容器的实例ID
`docker ps -a`
2.根据容器实例ID查询进程ID
` ps -ef | grep 容器实例ID`
3.查看资源占用
`top -p 进程ID`