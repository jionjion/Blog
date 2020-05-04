---
title: Docker基础
abbrlink: 1d9fbb6a
date: 2020-03-02 20:21:37
categories:
tags:

---





# 命令集

## 配置

**Docker 是一个服务端和客户端程序**

Docker客户端只需向Docker服务器或守护进程发出请求，服务器或守护进程将完成所有工作并返回结果。Docker提供了一个命令行工具 `docker` 以及一整套 `RESTfulAPI`。
你可以在同一台宿主机上运行Docker守护进程和客户端，也可以从本地的Docker客户端连接到运行在另一台宿主机上的远程Docker守护进程。

### CentOS

- 配置文件在 ` /etc/docker/` 下.
- 加载/重启 `sudo systemctl daemon-reload` / `sudo systemctl restart docker`
- 启动/停止 `sudo systemctl start docker` / `sudo systemctl stop docker`
- 开机自启 `sudo  systemctl enable docker`
- 环境变量 `DOCKER_HOST` ,  服务端默认连接 `IP`

### Client

- 查看Docker服务 `docker info -H 127.0.0.1:2375 `



## 容器

### `run` 

通过镜像启动一个容器,并返回容器ID

#### 交互式启动

`docker run -it ubuntu /bin/bash`
通过镜像 `ubuntu` 启动容器的 `bash` 命令, `-i` 开启容器标准输入, `-t` 告诉容器分配一个伪 `tty` 终端.

#### 命名容器

`docker run -it --name my-ubuntu ubuntu /bin/bash`
通过 `--name` 可以为容器分配一个名字

#### 运行守护容器

`docker run -d nginx`
使用 `-d` 将当前容器挂入后台运行, 返回生成的容器ID

#### 自动重启容器

`docker run --restart=always nginx`
当 `nginx` 容器以外关闭时,自动重启容器. 

`docker run --restart=on-failure:5 nginx`
当 `nginx` 容器因为程序意外关闭时,自动重启,最大尝试重启5次

#### 绑定端口

将宿主机的端口与容器的端口相绑定. 一个端口只能绑定一个容器

`docker run -p 80:80 nginx`
运行`nginx` 并绑定宿主机的 `80` 端口到容器的 `80` 端口

`docker run -P nginx`
运行`nginx` 并将容器内的端口随机映射到宿主机

#### 测试容器

`docker run -it --name 容器的名字 --rm -p 虚拟机端口:本地端口 <镜像ID>:<镜像标签> /bin/bash`
创建一次性的容器,程序结束容器销毁. `--rm` 在容器结束运行后, 删除容器

#### 容器资源限制

`docker run --memory 150M --memory-swap=200M <镜像ID> `
创建容器,限制内存大小. `--memory` 限制内存(单位 `G`,`M`, `KB`), `--memory-swap` 限制内存+交换分区大小

### `ps`

查看容器运行状态,默认显示正在运行的容器.可以看到:
容器的ID, 源镜像, 容器最后执行的命令, 创建时间, 退出状态, 容器绑定端口, 容器名字

#### 查看所有容器

`docker ps -a`
查看容器的所有运行状态

#### 查询最后一次运行的容器

`docker ps -l`
查询容器最后一次运行的容器,无论现在容器是否关闭

#### 查看最后的容器状态

`docker ps -n <数量>`
查看最后几个容器的运行状态,无论容器是否正在运行

### `start`

启动容器

#### 启动容器

`docker start <容器1ID> <容器2ID>`
指定多个容器ID,并启动

### `restart`

#### 重启容器

`docker restart <容器ID1> <容器ID2>`
重启多个容器

### `stop`

停止正在运行的容器

#### 停止

`docker stop <容器ID>`
停止一个容器

### `kill`

杀掉一个正在运行的容器, 非正常闭关

#### 杀掉

`docker kill <容器ID>`
立即结束一个容器, 信号为 `kill`

### `exec`

在容器内启动新的进程. 可以是后台进程或者交互式进程

####交互式进程

`docker exec -it <容器ID> /bin/bash`
通过交互式界面,进入容器内的 `bash` 窗口

#### 后台进程

`docker exec -d <容器ID> <命令>`
后台执行命进程

### `cp`

在宿主机和容器之间拷贝文件

#### 拷贝文件

`docker cp <容器ID>:<容器内文件路径>  <宿主机文件路径>`
将容器内的文件拷贝到宿主机

`docker cp <宿主机文件路径> <容器ID>:<容器内文件路径>`  
将宿主机的文件拷贝到容器中

### `pause`

暂停容器的进行, 服务重启, 容器关闭

#### 暂停

`docker pause <容器ID>`
暂停一个容器

### `unpause`

恢复暂停容器的进行, 服务重启, 容器关闭

#### 暂停

`uocker unpause <容器ID>`
恢复暂停一个容器

### `rename`

重命名一个容器

#### 重命名

`docker rename <容器ID> <新的容器名>`
将一个容器重新命名

### `rm`

删除容器

#### 删除容器

`docker rm <容器ID>`
删除指定的容器

### `attach`

将当前 `tty` 附着到容器中,获得容器的标准输入,标准输出,错误输出等...

#### 附着容器

`docker attach <容器ID>`

附着到指定容器中,获得容器的状态

### `log`

查看容器的日志

#### 查看容器的日志

`docker log -tf <容器ID>`
查看容器的运行日志,将当前日志信息实时打印出来. `-t` 显示日志时间, `-f` 实时打印日志输出

####查看容器部分日志

`docker log --lines 20 <容器ID>`
查看容器的日志的最后 `20` 行

### `top`

查看容器中,运行程序的进程信息.

#### 查看容器进程

`docker top <容器ID>`
查看容器的进程信息,可以看到容器的所以进程以及进程对应的 `PI` 和 `USER` 及 `TIME`

### `inspect`

查看容器的详细信息

#### 查看容器信息

`docker inspect <容器ID>`
查看容器的所有信息

#### 各种信息

使用 `-f` 指定解析规则

```bash
# 获得容器的IP信息
docker inspect -f "{{ .NetworkSettings.IPAddress }}" <容器ID>
```



### `stats`

查看容器的运行状态

#### 运行状态

`docker stats`
查看容器的运行状态,包括资源占用,端口映射,命名等

## 镜像

### `images`

查看当前所有的镜像信息

#### 查看镜像

`docker images`
查看当前所有镜像

### `commit`

提交当前容器,并将期间的操作写入镜像层, 构建新的镜像

#### 提交镜像

`docker commit <容器ID> <镜像仓库>/<镜像名称>:<版本标签>`
将当前容器,通过 `commit` 构建为一个镜像,并指定镜像的仓库和标签信息

#### 提交镜像署名用户信息

`docker commit -m "提交信息" --author="作者" <容器ID> <镜像仓库>/<镜像名称>:<版本标签>`
使用 `-m` 绑定提交信息, `--author` 署名作者

### `tag`

为当前镜像搭上标签

#### 添加标签

`docker tag <镜像ID> <仓库地址>/<镜像名称>:<版本标签>`
为镜像添加仓库信息与标签信息

### `builder` 

使用 `Dockerfile` 构建镜像

#### 构建本地镜像

`docker builder .`
在当前文件夹下, 通过 `Dockerfile` 构建镜像

### `history`

显示镜像的构建命令历史

#### 显示构建历史

`docker history <镜像ID>`
显示一个镜像的构建历史

### `rmi`

删除一个镜像

#### 删除镜像

`docker rmi <镜像ID>`
删除指定镜像

## 仓库

### `logout`

退出登录

####  登出阿里云

`docker logout <阿里云分配的仓库地址>`
登出阿里云

### `login`

登录远端仓库

#### 登录阿里云

`docker login --username=阿里云用户名 <阿里云分配的仓库地址>`
登录,输入用户名,随后输入密码

### `pull`

拉取远端镜像仓库的指定镜像

#### 拉取镜像

`docker pull ubuntu:16.04`
使用 `pull` 拉取镜像 `ubuntu` 并指定标签为 `16.04`

### `search` 

查询远端镜像, `NAME` 镜像的仓库和标签, `DESCRIPTION` 镜像描述信息, `STARS` 镜像的欢迎程度,  `OFFICIAL` 是否为官方镜像, `AUTOMATED` 是否由官方自动构建完成.

#### 查询远端镜像

`docker search ubuntu`
查询远端的 `ubuntu` 镜像

### `push`

将当前镜像推送到远端仓库

#### 推送镜像

`docker push <镜像仓库>/<镜像名称>:<镜像标签>`



### `version`

查看 Docker 的版本 `docker version`