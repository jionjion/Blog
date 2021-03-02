---
title: Docker基础-Python容器化
abbrlink: f89fd7d2
date: 2020-01-20 16:46:44
categories:
  - 服务器
  - Docker
tags: [Linux, Docker, Python]
---

> 介绍 `Python` 的容器化部署

<!--more-->



## 官网

参考网站 [DockerHub](https://hub.docker.com/_/python)

## 命令

下载镜像命令

`docker pull python`



运行镜像中的Python环境

` docker exec -it amazing_elgamal python`

进入镜像

`docker exec -it amazing_elgamal /bin/bash`



## 镜像构建

参考脚本

```dockerfile
# 使用Python3构建镜像
FROM python:3

# 确定工作空间
WORKDIR /usr/src/app

# 拷贝本地文件
COPY requirements.txt ./

# 安装一些常用软件
RUN pip install --no-cache-dir -r requirements.txt

# 拷贝其他
COPY . .

# 运行预设的脚本
CMD [ "python", "./your-daemon-or-script.py" ]
```





## 容器运行Python脚本



`docker run -it --rm --name my-running-script -v "$PWD":/usr/src/myapp -w /usr/src/myapp python:3 python your-daemon-or-script.py`





## Scrapy容器化

### 脚本

创建目录, `scrapyweb` ,并在该文件夹下创建容器脚本和初始化执行脚本

`dockerfile` 脚本

```bash
# 使用Python3构建镜像
FROM python:3

# 版本信息
LABEL version="1.0" 

# 个人信息
MAINTAINER Jion<jion@aliyun.com>

# 确定工作空间,初始位置
WORKDIR /usr/src/app

# 拷贝脚本.写全路径.不然会拷贝到工作空间下
COPY init.sh /root/init.sh

# 安装软件并运行
RUN pip install --upgrade pip \
 && pip install scrapy \ 
 && pip install scrapyd \
 && pip install scrapydweb \
 && chmod +x /root/init.sh

# 开放端口
EXPOSE 5000

# 运行预设的脚本

# 默认命令,执行脚本
CMD ["/root/init.sh"]
```

`init.sh` 脚本

```bash
#!/bin/bash
echo 'start init.bash -----'

# 后台启动 scrapyd
scrapyd &

# 启动成功后,执行
if [ $? -eq 0 ]
then
# 后台启动 scrapyweb
scrapydweb &
fi

#保留一个终端，防止容器自动退出
/bin/bash

echo 'end init.bash '
```



### 步骤

在当前 `scrapyweb` 目录下,执行构建命令

构建镜像

` docker build`

运行镜像,并指定端口

`docker run -p 5000:5000 0a03bfeff2f0`

