---
title: Docker基础-Nginx容器化
abbrlink: a2c33bc3
date: 2020-04-07 20:20:42
categories:
tags:
---

## Nginx容器化

参考  [官网](https://hub.docker.com/_/nginx)



### 镜像启动

启用测试镜像

`docker run --name test --rm -p 80:80 nginx`

### 托管静态目录

`docker run --name test --rm -v /data/hexo/public:/data/hexo/public:ro -p 80:80 nginx`



## DockerFile 编写

构建命令, 创建镜像 `jiojion/nginx'

`docker build -t  jionjion/nginx:1.0 .`

其中,` Dockerfile` 如下

```bash
# DockerFile ,创建本地镜像
# 来源,官方镜像
FROM nginx

# 维护者信息
MAINTAINER Jion<jion@aliyun.com>

# 指定容器内运行用户
USER root

# 拷贝配置
# 1.Nginx默认配置文件到容器
COPY ./resource/nginx.conf       /etc/nginx/nginx.conf
# 2.各个子域名配置信息
COPY ./resource/conf.d/          /etc/nginx/conf.d/
# 3.各个子域名证书
COPY ./resource/cert/            /etc/nginx/cert/

# 暴露端口
EXPOSE 80
EXPOSE 443
```



构建测试容器

`docker run --name test --rm -v /data/hexo/public:/data/hexo/public:ro -p 80:80 jionjion/nginx:1.0`

支持443端口的测试容器

`docker run --name test --rm -v /data/hexo/public:/data/hexo/public:ro -p 80:80 -p 443:443 jionjion/nginx:1.0`

正式启动

`docker run --name my-nginx -d -v /data/hexo/public:/data/hexo/public:ro -p 80:80 -p 443:443 jionjion/nginx:1.0`