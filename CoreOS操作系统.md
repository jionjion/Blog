---
title: CoreOS操作系统
abbrlink: a3fb60ed
date: 2018-04-25 23:15:53
categories:
  - 服务器
  - CentOS
tags: [Linux, CentOS]
---

# 简介


## 思想
系统文件系统只读,软件通过安装在不同容器中进行执行,推荐一个容器安装一个运行程序.

# Docker容器
Docker 客户端
Docker 守护进程
Docker 镜像仓库

## 更换为阿里云镜像
阿里容器镜像地址:https://dev.aliyun.com/search.html

1. 申请开发者账号,申请完成后会有专属加速器地址.
2. 修改Docker配置文件,记得备份哦.另外,Docker某些文件系统运行时为只读状态,是不能修改的,因此拷贝`/usr/lib/systemd/system/flannel-docker-opts.service`到`/etc/systemd/system`下进行修改
3. 修改内容,追加`--registry-mirror=https://ybxgrzcu.mirror.aliyuncs.com`在`ExecStart`属性中.
4. 执行,`systemctl daemon-reload`命令,重新加载容器配置文件
5. 重启容器.`systemctl restart docker`

``` 
[Unit]
Description=flannel docker export service - Network fabric for containers (System Application Container)
Documentation=https://github.com/coreos/flannel
PartOf=flanneld.service
Requires=flanneld.service
After=flanneld.service
Before=docker.service

[Service]
Type=oneshot

Environment="FLANNEL_IMAGE_TAG=v0.8.0"
Environment="RKT_RUN_ARGS=--uuid-file-save=/var/lib/coreos/flannel-wrapper2.uuid"
Environment="FLANNEL_IMAGE_ARGS=--exec=/opt/bin/mk-docker-opts.sh"

ExecStartPre=-/usr/bin/rkt rm --uuid-file=/var/lib/coreos/flannel-wrapper2.uuid
ExecStart=/usr/lib/coreos/flannel-wrapper -d /run/flannel/flannel_docker_opts.env -i --registry-mirror=https://ybxgrzcu.mirror.aliyuncs.com \
ExecStop=-/usr/bin/rkt stop --uuid-file=/var/lib/coreos/flannel-wrapper2.uuid

[Install]
WantedBy=multi-user.target
~
"flannel-docker-opts.service" 21L, 807C
```


4. 重新启动容器

## 命令行

### `help`命令
查看所有帮助

#### **语法**

``` shell
docker help (选项)
```

#### **示例**
**查看所有帮助**
``` shell
docker help
```

### `search`命令
搜寻官网提供的镜像

#### **语法**
``` shell
docker search (选项) [镜像名]
```

#### **选项**
`--automated=false` 仅显示自动创建的镜像
`--no-trunc=false` 显示信息不截断
`-s  0`,`--stars=0` 仅显示指定星数以上的镜像


#### **示例**
**搜寻MySQL镜像**
``` shell
docker search mysql
```
### `pull`命令
下载镜像

#### **语法**

``` shell
docker pull (选项) [镜像名]
```

#### **示例**
**安装最新版的乌班图系统**
``` shell
docker pull ubuntu:latest
```
### `images`命令
查看已经下载好的镜像	

#### **语法**

``` shell
docker images (选项)
```

### `tag`命令
为镜像打上标签

#### **语法**

``` shell
docker tag [镜像名] [标签名]
```

#### **为Ubuntu系统打上Studying标签**

``` shell
docker tag ubuntu studying
```


### `inspect`命令
查看某个运行时镜像

#### **语法**

``` shell
docker inspect (选项) [镜像ID]
```

#### **选项**
`-f`  查看指定选项,使用`{{参数}}`指定要获取的信息


#### **示例**
**查看某镜像的状态**
``` shell
docker inspect ubuntu
```

**查看镜像的操作系统**

``` shell
docker inspect -f {{".Os"}}  ubuntu
```


### `rmi`命令
通过镜像标签删除镜像:
当标签存在多个时,被删除的只是对应标签
当标签只有一个时,镜像将被彻底删除
通过镜像ID删除镜像:
删除镜像文件和对应的所有的标签

注意:当容器正在运行时,不能删除!
#### **语法**

``` shell
docker rmi ubuntu
```

#### **选项**
`-f` 强制删除


### `commit`命令
创建本地镜像文件

#### **语法**
``` shell
docker commit (选项) [镜像名[:标签]]
```

#### **选项**
`-a`,`--author=" "`  提交作者的信息
`-m`,`--message=" "` 提交信息
`-p`,`-- pause=true` 提交时暂停容器运行


### `import`命令
从本地模板中导入镜像
#### **语法**

``` shell
docker import 本地镜像
```


### `save`命令
将当前镜像保存为本地文件

#### **语法**

``` shell
docker save (选项) [镜像的名字]
```

#### **选项**
`-o` 导出文件的名字

#### **示例**
**导出ubuntu镜像到本地**

``` shell
docker save -o ubuntu.tar ubuntu
```


### `load`命令
加载本地文件到镜像库

#### **语法**

``` shell
docker load (选项) [文件的名字]
```

#### **示例**
**导入本地ubuntu镜像**

``` shell
docker load --input ubuntu.tar
```


### `push`命令
上传命令到远端仓库
#### **语法**

``` shell
docker push 镜像名[:标签]
```


### `create`命令
根据本地镜像创建一个新的容器

#### **语法**

``` shell
docker create (选项) [镜像的名字]
```
#### **选项**
`-i` 自动安装
`-t` 暂停

### `ps`命令
查看容器运行情况

#### **语法**

``` shell
docker ps (选项)
```

#### **选项**

`-a` 查看所有容器运行情况	
`-q` 查看终止状态的容器

#### **示例**
**查看所有容器运行情况**
``` shell
docker ps -a
```
**查看所有终止状态的容器**

``` shell
docker ps -a -q
```



### `run`命令
基于镜像进行容器启动
执行某镜像,不存在则官网拉取镜像
等价于首先`docker create`和`docker start`先后执行.

#### **语法**

``` shell
docker run (选项) [镜像] [镜像命令]
```

#### **选项**
`-i`,`-- interactive`	 		交互执行容器
`-t`,`--TTY`		分配伪终端,并绑定标准输出
`-- name`					指定容器名字
`-d`,`-- detach`	以守护进程执行容器	
`-p 主机端口:容器端口`  分配端口映射
`-- publish`	绑定容器端口和主机端口
`-v`,`-- volume`	指定挂载的数据卷位置,在容器内创建数据卷,可多次创建	
`-- volumes-from`	从其他容器中,挂载数据卷.	
`-- link`	连接容器名和容器别名	便于查看环境变量和主机hosts文件映射
`-- rm`	容器运行结束后自动删除,常用作测试使用

#### **示例**
**交互模式,分配TTY端口 运行在本地80端口进行执行**
``` shell
docker run --interactive --tty --publish 80 ubuntu:latest /bin/bash 
```

**Tomcat的8080端口映射为主机80端口,对外提供80端口访问**

``` shell
docker run --interactive --tty --publish 80 --volume /ubuntu_data ubuntu:latest
```



### `exit`命令
在容器内部执行,用于退出容器.
`Ctrl + D`强制退出容器

### `pause`命令
暂停某个运行时容器

#### **语法**

``` shell
docker pause (选项) [容器名字]
```
### `uppause`命令
恢复某个运行时容器

#### **语法**

``` shell
docker unpause (选项) [容器名字]
```

### `stop`命令
停止某个运行时容器

#### **语法**

``` shell
docker stop (选项) [容器名字]
```

### `kill`命令
强行关闭某个运行时容器

#### **语法**

``` shell
docker kill (选项) [容器名字]
```

### `start`命令
启动某个已停止的容器

#### **语法**

``` shell
docker start (选项) [容器名字]
```


### `restart`命令
重启某个容器

#### **语法**

``` shell
docker restart (选项) [容器名字]
```


### `port`命令
查看容器的端口映射关系

#### **语法**

``` shell
docker port (选项)
```



### `exec`命令
进入某个守护进程运行的容器

#### **语法**

``` shell
docker exec (选项)


#### **选项**
`-i`,`-- interactive`	 		交互执行容器
`-t`,`--TTY`		分配伪终端,并绑定标准输出
`-d`,`-- detach`	以守护进程执行容器	
```

#### **示例**
**以交互式命令进入到乌班图系统中**
``` shell
docker exec --tty --interactive suspicious_franklin /bin/bash
```



### `rm`命令
彻底删除某个容器

#### **语法**

``` shell
docker rm (选项) [容器名字]
```

#### **选项**
`-- volums` 删除时级联删除数据卷
`-f` 强制删除




### `export`命令
导出某个容器到文件系统中
无论容器运行状态,均可到导出

#### **语法**

``` shell
docker export (选项) [镜像的名字]
```



### `login`命令
登录自己的远程镜像仓库

### `push`命令
上传镜像

### `build`命令
使用DockerFile脚本创建本地镜像服务


### `logs`命令
显示某个容器当前运行日志



## 容器监控

## 数据卷
对文件系统进行不同容器间的共享


## 数据仓库


# 端口映射



# Systemd服务管理
**Unit** Systemd管理的最小单元,每一个服务为一个unit单元,并使用一个文件进行定义,配置服务运行时的环境,服务启动和结束时执行的实际操作.
**Target** Systemd中用于指定服务组启动组的方式,相当于SysV-init中的''运行级别''.


| 命令       | 选项            | 参数 | 说明             | 实例 |
| ---------- | --------------- | ---- | ---------------- | ---- |
| systemctl  | start           |      | 启动服务         |      |
|            | restart         |      | 重启服务         |      |
|            | stop            |      | 结束服务         |      |
|            | kill            |      | 杀死服务         |      |
|            | list-units      |      | 查看所有服务     |      |
|            | list-unit-files |      | 产看所有服务文件 |      |
|            | daemon-reload   |      | 重新加载服务文件 |      |
| journalctl | dmesg           |      | 查看系统启动日志 |      |
|            | unit            |      | 查看指定服务日志 |      |
|            | follow          |      | 实时输出日志     |      |
|            |                 |      |                  |      |





# 常见软件安装


## Vim编辑器安装
[官网安装教程][1]
[GitHub地址][2]

1. 从github上下载编辑器
	- `wget https://github.com/vim/vim/archive/master.zip`
2. 解压,移动到源代码目录
	- `unzip master.zip`解压到当前文件夹
	- 进入`/vim/src`目录下
3. 编译安装
	- 
4. 

## Yum服务及国内镜像配置
Yum是rpm系统的管理工具,可以很方便的实现软件的安装,更新,卸载.一般镜像源中不包括Yum服务,我们需要单独下载,便于日后软件安装.
这里使用CentOS系统,通过修改为阿里云镜像.

1. 备份`mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup`
2. 下载新的CentOS-Base.repo 到`/etc/yum.repos.d/`
	- 如果是 CentOS7系统 `wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo`
	- 如果是 CentOS6系统 `wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo`
3. 使用`yum makecache`重载缓存

## MySQL数据库安装

[官方镜像文件][3]

1. 创建文件夹,用来保存数据卷
	- `~/mysql/data`,`~/mysql/logs`,`~/mysql/conf`.三个文件夹,分别保存数据,日志和配置文件
2. 拉取官方镜像
	- `docker pull mysql`
3. 运行镜像
	- `docker run -p 3306:3306 --name MySQL -v $PWD/logs:/logs -v $PWD/data:/mysql_data -e MYSQL_ROOT_PASSWORD=123456 -d mysql`
		- `-p 3306:3306`：将容器的3306端口映射到主机的3306端口
		- `-v $PWD/conf/my.cnf:/etc/mysql/my.cnf`：将主机当前目录下的conf/my.cnf挂载到容器的/etc/mysql/my.cnf
		- `-v $PWD/logs:/logs`：将主机当前目录下的logs目录挂载到容器的/logs
		- `-v $PWD/data:/mysql_data`：将主机当前目录下的data目录挂载到容器的/mysql_data
		- `-e MYSQL_ROOT_PASSWORD=123456`：初始化root用户的密码
4. 服务器本地进入数据库
	- `docker run -it --rm mysql mysql 127.0.0.1 -uroot -p` 
5. 进入容器
	- `docker run -it  mysql /bin/bash`
5. 开启远程连接




## Tomcat服务安装
[官方镜像文件][4]


1. 创建本地文件,作为容器安装目录.
	- `~/tomcat/webapps`
2. 拉取官方镜像
	- `docker pull tomcat`
3. 运行容器
	- `docker run -dit -p 8080:8080 --name tomcat8 tomcat`
可以看到对应输出,设置有对应环境变量信息,访问对应的IP地址和端口号,可以进行相应的访问.
4. 服务器开启后可能会有半分钟左右的延时.

``` 
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /docker-java-home/jre
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
```

### 开启用户管理
1. `docker cp 5d3dff713ddc://usr/local/tomcat/conf/tomcat-users.xml ~/
`拷贝配置文件到本地,因为容器中可能没有相关的编辑工具,苦啊~
2. 在`tomcat-users.xml`中添加如下信息.
创建用户admin,密码123456
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users>
<user username="admin" password="123456" roles="manager-gui"/>
</tomcat-users>
```
3. 将源文件替换`docker cp ~/tomcat-users.xml  5d3dff713ddc://usr/local/tomcat/conf/tomcat-users.xml`
4. 修改`/usr/local/tomcat/webapps/manager/META-INF/context.xml`文件,开启远程连接管理.
``` xml
<Context antiResourceLocking="false" privileged="true" >
<!--
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```


4. 重启服务.

### 修改容器字符集
1. 拷贝文件本地修改`\conf\server.xml`.
2. 修改内容

``` xml
<Connector executor="tomcatThreadPool"
        port="8080" protocol="HTTP/1.1"
        connectionTimeout="20000"
        redirectPort="8443"
        URIEncoding="UTF-8" />
```


### 版本说明
不同版本的配置文件在`/usr/local/tomcat/conf/`文件下,通过修改文件指向环境变量,可以生成对应的版本.

**Tomcat7和Tomcat8环境变量**
``` 
CATALINA_BASE:   /usr/local/tomcat
CATALINA_HOME:   /usr/local/tomcat
CATALINA_TMPDIR: /usr/local/tomcat/temp
JRE_HOME:        /usr
CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
```

**Tomcat6环境变量**
``` 
CATALINA_BASE:   /usr/local/tomcat
CATALINA_HOME:   /usr/local/tomcat
CATALINA_TMPDIR: /usr/local/tomcat/temp
JRE_HOME:        /usr
CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar
```



## Redis数据库安装
使用Redis作为缓存数据库,应对高并发请求.

1. `docker pull redis`下载镜像
2. ` docker run --name redis -d redis` 从镜像启动运行容器
	- 容器命名为redis
	- `-d`后台运行

## FTP服务安装


[1]: https://vim.sourceforge.io/download.php#unix
[2]: https://github.com/vim/vim
[3]: https://hub.docker.com/_/mysql/
[4]: https://store.docker.com/images/tomcat