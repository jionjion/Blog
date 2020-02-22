---
title: Spring全家桶-Dokcer镜像打包
abbrlink: 9642c4f
date: 2019-10-27 14:59:48
categories:
  - Java
  - Spring
tags: [Java, Spring, Docker]
---



##简介

介绍SpringBoot与Docker整合.通过maven插件,完成自动打包,生成镜像.



### 简单的单应用打包

#### 修改Pom文件

添加插件`dockerfile-maven-plugin`

其中,指定镜像仓库`repository`为 项目前缀/版本号

属性`buildArgs` 列表中,声明一个属性`JAR_FILE`,在DockerFile中使用,作为变量传入.其值为当前项目打包成Jar文件后的文件位置

```xml
<build>
    <plugins>
        <!-- 插件...镜像打包时使用 -->
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.9</version>
            <configuration>
                <!-- 镜像:仓库/版本 -->
                <repository>${docker.image.prefix}/${project.artifactId}</repository>
                <!-- 属性参数 -->
                <buildArgs>
                    <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### 编写Dockerfile

在项目根目录下,创建文件`Dockerfile`.内容入下

其中局部变量`JAR_FILE`不赋值,由插件执行时通过其参数列表传入

```bash
# 来源基础镜像
FROM openjdk:8u232

# 镜像制作人
MAINTAINER JionJion <jionjion@aliyun.com>

# 设置局部变量,容器工作目录.仅在当前文件中有效.    ENV:环境变量范围内设置变量
ARG WORKDIR=/app

# 设置局部变量,拷贝的Jar文件位置
ARG JAR_FILE

# 挂载目录
VOLUME ${WORKDIR}

# 工作目录,当前命令执行时所在的容器目录,及日后进入容器时的初始位置
WORKDIR ${WORKDIR}

# 获得宿主机在桥段内的IP地址,并命名为  dockerhost
#RUN /sbin/ip route|awk '/default/ { print  $3,"\tdockerhost" }' >> /etc/hosts

# 将Jar包添加到服务器中,并重命名.  默认存放位置为初始工作目录
# ADD springboot-docker-1.0.jar service-web-console.jar

# 拷贝,变量在pom.xml文件中通过插件设置
COPY ${JAR_FILE} service-web-console.jar

# 暴露容器的端口,并未实际打开.镜像生成容器时指定
EXPOSE 80

# 容器运行后,第一条命令. 随机数生成.. 仅能配置一条
# ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar"]
ENTRYPOINT ["java","-jar","service-web-console.jar"]
```

#### 打包命令

在当前目录下,或者通过插件执行.

直接打包

`mvn  dockerfile:build`

清除当前输出目录,跳过测试用例,打包

`mvn clean package dockerfile:build -Dmaven.test.skip=true`

#### 查看容器中的IP

默认通过桥接模式连接容器,其宿主机IP为所在网段的路由IP.

通过命令查看,默认容宿主机的IP为 `10.0.75.1`

`/sbin/ip route|awk '/default/ { print  $3,"\tdockerhost" }'`

