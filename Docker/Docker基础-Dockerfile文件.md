---
title: Docker基础-Dockerfile文件
abbrlink: 381ea0b5
date: 2019-10-27 15:01:38
categories:
  - 服务器
  - Docker
tags: [Linux, Docker]
---

> 介绍 Dockerfile 的编写和存放一些示例.



## Dockerfile

Dockerfile是一个包含用于组合映像的命令的文本文档.可以使用在命令行中调用任何命令. Docker通过读取Dockerfile中的指令自动生成映像.

docker build命令用于从Dockerfile构建映像.可以在 `docker build` 命令中使用 `-f` 标志指向文件系统中任何位置的Dockerfile.

`docker build -f /path/to/a/Dockerfile`
 `#` 为 Dockerfile 中的注释.

### 脚本结构

- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行指令


### 操作指令

Docker以从上到下的顺序运行Dockerfile的指令.
为了指定基本映像,第一条指令必须是FROM.
可以在Docker文件中使用RUN,CMD,FROM,EXPOSE,ENV等指令.


#### FROM
指定基础镜像,必须为第一个命令

- 格式
　　`FROM <image>`
　　`FROM <image>:<tag>`
　　`FROM <image>@<digest>`
- 示例
　　`FROM mysql:5.6`
- 注
　　`tag` 或 `digest` 是可选的,如果不使用这两个值时,会使用 `latest` 版本的基础镜像

#### MAINTAINER
维护者信息,选填

- 格式
   `MAINTAINER <name>`
- 示例
    - `MAINTAINER Jion`
    - `MAINTAINER Jion@aliyun.com`
    - `MAINTAINER Jion<jion@aliyun.com>`
#### RUN
构建镜像时执行的命令, 在每一次执行时,会新生成一层新的镜像层,因此建议将多个命令串行执行
`RUN` 用于在镜像容器中执行命令,其有以下两种命令执行方式

`RUN` 指令创建的中间镜像会被缓存,并会在下次构建中使用.如果不想使用这些缓存镜像,可以在构建时指定 `--no-cache` 参数,如 `docker build --no-cache`

**shell执行**

- 格式
    `RUN <command>`

- 示例
  
- `RUN mkdir -p /dirA/dirB`
      执行递归创建文件夹命令
  
    - `RUN pip install -i https://mirrors.aliyun.com/pypi/simple/ scrapy scrapyd \
       && pip install -i https://mirrors.aliyun.com/pypi/simple/ scrapyweb`
    
      一次执行多个命令,减少构建

**exec执行**

- 格式
    `RUN ["executable", "param1", "param2"]`
- 示例
    - `RUN apk update`
      更新 `apk` 包
    - `RUN ["/etc/execfile.sh", "arg1", "arg1"]`
  执行命令文件


#### ADD
将本地文件添加到容器中, `tar` 类型文件会自动解压(网络压缩资源不会被解压),可以访问网络资源,类似 `wget`
支持路径匹配表达式

- 格式
    `ADD <src>... <dest>`
    `ADD ["<src>",... "<dest>"]` 用于支持包含空格的路径
- 示例
    - `ADD hom* /mydir/` 
      添加所有以"hom"开头的文件
    - `ADD hom?.txt /mydir/`
      ? 替代一个单字符,例如"home.txt"
    - `ADD test relativeDir/` 
      添加 "test" 到 `WORKDIR`/relativeDir/, `WORKDIR` 由命令指定.默认家目录
    - `ADD test /absoluteDir/` 
      添加 "test" 到 /absoluteDir/

#### COPY
功能类似ADD,但是是不会自动解压文件,也不能访问网络资源
将将宿主机器上的的文件复制到镜像内,如果目的位置不存在,Docker会自动创建.

#### CMD
构建容器后调用,也就是在容器启动时才进行调用.常作为默认命令
CMD不同于RUN, CMD用于指定在容器启动时所要执行的命令, 而RUN用于指定镜像构建时所要执行的命令.

- 格式
    `CMD ["executable","param1","param2"]` (执行可执行文件,优先)
    `CMD ["param1","param2"]` (设置了ENTRYPOINT,则直接调用ENTRYPOINT添加参数)
    `CMD command param1 param2` (执行shell内部命令)
- 示例
    - `CMD echo "This is a test." | wc -`
      执行带管道符命令
    - ``CMD ["/usr/bin/wc","--help"]`
      执行命令

#### ENTRYPOINT
配置容器,使其可执行化.配合CMD可省去"application",只使用参数.

ENTRYPOINT与CMD非常类似,不同的是通过docker run执行的命令不会覆盖ENTRYPOINT,而docker run命令中指定的任何参数,都会被当做参数再次传递给ENTRYPOINT.

Dockerfile中只允许有一个ENTRYPOINT命令,多指定时会覆盖前面的设置,而只执行最后的ENTRYPOINT指令.

- 格式
    `ENTRYPOINT ["executable", "param1", "param2"]` (可执行文件, 优先)
    `ENTRYPOINT command param1 param2` (shell内部命令)
- 示例
    ```bash
	FROM ubuntu
    // 默认命令
    ENTRYPOINT ["top", "-b"]
	CMD ["-c"]
	```

#### LABEL
用于为镜像添加元数据

使用LABEL指定元数据时,一条LABEL指定可以指定一或多条元数据,指定多条元数据时不同元数据之间通过空格分隔.推荐将所有的元数据通过一条LABEL指令指定,以免生成过多的中间镜像层.

- 格式
    `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
- 示例
　`LABEL version="1.0" description="这是一个Web服务器" by="JionJion"`

#### ENV
设置环境变量

- 格式
    `ENV <key> <value>`   
	`<key>` 之后的所有内容均会被视为其 `<value>` 的组成部分,因此,一次只能设置一个变量
    
	`ENV <key>=<value> ...`
	可以设置多个变量,每个变量为一个 `<key>=<value>` 的键值对,如果 `<key>` 中包含空格,可以使用 `\` 来进行转义,也可以通过""来进行标示；另外,反斜线也可以用于续行
	
- 示例
    `ENV myName Jion Jion`
    `ENV myDog Rex The Dog`
    `ENV myCat=fluffy`

#### EXPOSE
指定于外界交互的端口

`EXPOSE` 并不会让容器的端口访问到主机.要使其可访问,需要在 `docker run` 运行容器时通过 `-p` 来发布这些端口,或通过 `-P` 参数来发布 `EXPOSE` 导出的所有端口

- 格式
    `EXPOSE <port> [<port>...]`
- 示例
    - `EXPOSE 80 443`
      暴露多个端口
    - `EXPOSE 8080`
      暴露一个端口
    - `EXPOSE 11211/tcp 11211/udp`
      暴露多个端口及其支持协议类型

#### VOLUME

用于指定持久化目录

- 格式
    `VOLUME ["/path/to/dir"]`
- 示例
    `VOLUME ["/data"]`
    `VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"`
- 注
　　一个卷可以存在于一个或多个容器的指定目录,该目录可以绕过联合文件系统,并具有以下功能
1 卷可以容器间共享和重用
2 容器并不一定要和其它容器共享卷
3 修改卷后会立即生效
4 对卷的修改不会对镜像产生影响
5 卷会一直存在,直到没有任何容器在使用它

#### WORKDIR
工作目录,指定当前操作的位置

通过 `WORKDIR` 设置工作目录后,`Dockerfile` 中其后的命令 `RUN`、`CMD` 、`ENTRYPOINT` 、`ADD` 、`COPY` 等命令都会在该目录下执行.在使用 `docker run` 运行容器时,可以通过 `-w` 参数覆盖构建时所设置的工作目录.

- 格式
    `WORKDIR /path/to/workdir`
- 示例
    `WORKDIR /a  (这时工作目录为/a)`
    `WORKDIR b  (这时工作目录为/a/b)`
    `WORKDIR c  (这时工作目录为/a/b/c)`

#### USER
指定运行容器时的用户名或 `UID`,后续的 `RUN` 也会使用指定用户.使用 `USER` 指定用户时,可以使用用户名、 `UID` 或 `GID`,或是两者的组合.当服务不需要管理员权限时,可以通过该命令指定运行用户.并且可以在之前创建所需要的用户

使用 `USER` 指定用户后,`Dockerfile` 中其后的命令 `RUN`、`CMD`、`ENTRYPOINT` 都将使用该用户.镜像构建完成后,通过 `docker run` 运行容器时,可以通过 `-u` 参数来覆盖所指定的用户.

- 格式
　　`USER user`
`USER user:group`
`USER uid`
`USER uid:gid`
`USER user:gid`
`USER uid:group`
 - 示例
　　`USER www`

#### ARG
用于指定传递给构建运行时的变量

- 格式
    `ARG <name>[=<default value>]`
- 示例
    `ARG site`
    `ARG build_user=www`
#### ONBUILD
用于设置镜像触发器

当所构建的镜像被用做其它镜像的基础镜像,该镜像中的触发器仅会被最近一上层镜像影响.

- 格式
　　`ONBUILD [INSTRUCTION]`
- 示例
　　`ONBUILD ADD . /app/src`
　　`ONBUILD RUN /usr/local/bin/python-build --dir /app/src`



 ## 示例文件

