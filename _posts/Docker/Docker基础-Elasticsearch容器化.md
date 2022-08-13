---
title: Docker基础
categories:
  - 服务器
  - Docker
tags:
  - Linux
  - Docker
abbrlink: 1d9fbb6a
date: 2022-06-21 20:21:37
---

> 介绍 `Docker` 容器的基本操作

<!--more-->



## 版本

`8.X` 以上版本商业试用,需要购买版权.因此这里使用 `elasticsearch:7.17.4` 版本



## 快速安装

使用 Docker 拉取镜像, 这里是最新版本的镜像

`docker pull docker.elastic.co/elasticsearch/elasticsearch:8.2.3`

执行安装, 此时会显示下密码..

`docker run --name my-elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.2.3`

显示密码如下, 这里密码为 `8pLC5K=YL8gKbLe0v1yr` , 注册令牌只有30分钟有效期为`eyJ2ZXIiOiI4LjIuMyIsImFkciI6WyIxNzIuMTcuMC4zOjkyMDAiXSwiZmdyIjoiNzMxZWNlM2FhZTgwZDg3YjI1Y2U4YjUxYjQzYzllOTEwYzIyOGFkZjQ1MzViMTQxNjUxYWU5NmI3YTUwZTg2MCIsImtleSI6IjBKcUNpNEVCRk9iZHhpcDh5WXBoOlAzcHZGaFMtUy1tRGdGT0phdDVkeVEifQ==`

```bash
------------------------------------------------------------------------------------------------------------------------
-> Elasticsearch security features have been automatically configured!
-> Authentication is enabled and cluster connections are encrypted.

->  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  _NpRry7RcNGBGF1hvR9P

->  HTTP CA certificate SHA-256 fingerprint:
  067e5a32fa71dbaf8215a4adfcfac7f21197fc878ffebbb805a02d1466b1f3da

->  Configure Kibana to use this cluster:
* Run Kibana and click the configuration link in the terminal when Kibana starts.
* Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjIuMyIsImFkciI6WyIxNzIuMTcuMC4zOjkyMDAiXSwiZmdyIjoiMDY3ZTVhMzJmYTcxZGJhZjgyMTVhNGFkZmNmYWM3ZjIxMTk3ZmM4NzhmZmViYmI4MDVhMDJkMTQ2NmIxZjNkYSIsImtleSI6IkhraXFpNEVCOTE2NFh0RDhwUDBNOmRGN1FIazVIU1kyTzJtRzFNSjZYVkEifQ==

-> Configure other nodes to join this cluster:
* Copy the following enrollment token and start new Elasticsearch nodes with `bin/elasticsearch --enrollment-token <token>` (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjIuMyIsImFkciI6WyIxNzIuMTcuMC4zOjkyMDAiXSwiZmdyIjoiMDY3ZTVhMzJmYTcxZGJhZjgyMTVhNGFkZmNmYWM3ZjIxMTk3ZmM4NzhmZmViYmI4MDVhMDJkMTQ2NmIxZjNkYSIsImtleSI6IklFaXFpNEVCOTE2NFh0RDhwUDBNOkxpQjFaN1NaUTRhMnlKeHVEY19xb1EifQ==

  If you're running in Docker, copy the enrollment token and run:
  `docker run -e "ENROLLMENT_TOKEN=<token>" docker.elastic.co/elasticsearch/elasticsearch:8.2.3`
------------------------------------------------------------------------------------------------------------------------
```



### 异常处置 - 内存不足

如果抛出虚拟机内存不足... 由于本机用的是 WSL 子系统, 因此需要设置内存

`max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

设置内存子系统虚拟机内存.. 进入`wsl -d 发行版虚拟机` , 进入对应的虚拟机

```bash
wsl -d docker-desktop
echo 262144 >> /proc/sys/vm/max_map_count
```



### 减少内存

修改 `elasticsearch` 中的虚拟机设置 `/usr/share/elasticsearch/config/jvm.options` 

```bash
-Xms108m
-Xmx108m
```







## 图形工具

安装 `Kibana` 进行图形化管理

`docker pull docker.elastic.co/kibana/kibana:8.2.3`

启动工具

`docker run --name my-kibana -p 5601:5601 docker.elastic.co/kibana/kibana:8.2.3`

访问端口

`http://127.0.0.1:5601/?code=971242`

第一次进入需要配置下令牌, 令牌在上面安装的时候出现过,有效期30分钟...用户名 `elastic`  密码在上面安装时出现...

