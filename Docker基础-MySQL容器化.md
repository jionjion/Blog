---
title: Docker基础-MySQL容器化
abbrlink: b4d7f78f
date: 2020-04-11 16:06:51
categories:
tags:
---

## MySQL容器化

参考 [官网](https://hub.docker.com/_/mysql)

### 启动

通过以下命令创建测试镜像,  `--name` 命名容器, `--rm` 在容器结束后销毁, `MYSQL_ROOT_PASSWORD` 指定数据库root用户密码为123456,  指定镜像版本 `mysql:latest`

`docker run --name test --rm -e MYSQL_ROOT_PASSWORD=123456 mysql:latest`

进入容器
` docker exec -it 容器ID /bin/bash`

进入数据库, 并随后输入密码
` docker exec -it 容器ID /usr/bin/mysql -uroot -p`

### 自定义配置

镜像中,数据库的配置文件在 `/etc/mysql/conf.d` 文件夹下,通过将该文件夹外部挂载,可以实现自定义配置
数据文件存在 `/var/lib/mysql` 文件夹下.

```bash
# 创建本地文件,分别存配置文件和数据库
mkdir -p /data/mysql/conf.d  
mkdir -p /data/mysql/data

# 测试,启动镜像,并挂载指定文件夹
docker run --name test --rm -p3306:3306 -v /data/mysql/data:/var/lib/mysql -v /data/mysql/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 mysql:latest
# 正式启动
docker run --name my-mysql -d -p3306:3306 -v /data/mysql/data:/var/lib/mysql -v /data/mysql/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 mysql:latest

```



### 环境变量

`MYSQL_ROOT_PASSWORD`
必填, 管理员账户的密码

`MYSQL_DATABASE`
可选, 在容器启动后创建该数据库

`MYSQL_USER`, `MYSQL_PASSWORD`
可选,创建与root相同权限的户,便于管理数据库

`MYSQL_ALLOW_EMPTY_PASSWORD`
可选,当为 `yes` 时允许任何人不使用密码登入数据库

`MYSQL_RANDOM_ROOT_PASSWORD`
可选,在容器启动后,为root用户随机生成密码

`MYSQL_ONETIME_PASSWORD`
可选,在容器启动后,直接过期root用户的密码,以便下载登录时重置



### 其他设置

#### 备份与还原

```bash
# 备份
docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql
# 还原
docker exec -i some-mysql sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < /some/path/on/your/host/all-databases.sql
```

#### Navicat for MySQL连接

当版本过低时,默认连接会失效.在数据库命令行执行以下内容

```bash
# 修改密码
ALTER USER '用户'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
# 允许远程连接,切勿在生产环境使用
ALTER USER '用户'@'%' IDENTIFIED WITH mysql_native_password BY '新密码';
# 刷新权限
FLUSH PRIVILEGES;
```

#### 创建数据库及用户并远程连接

```sql
# 创建数据库
CREATE DATABASE 数据库名;

# 创建用户,并允许远程连接
CREATE USER '用户名'@'%' IDENTIFIED BY '密码';

# 切换到对应数据库下
USE 数据库名
# 授权数据库给某个用户
GRANT ALL PRIVILEGES ON 数据库.* TO '用户名'@'%';

# 刷新权限
FLUSH PRIVILEGES;
```

