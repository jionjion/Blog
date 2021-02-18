---
title: Ubuntu桌面系统
abbrlink: 2a435475
date: 2019-03-25 12:34:28
categories:
  - 服务器
  - Ubuntu
tags: [Ubuntu]
---

# 环境
系统 `Ubuntu 20.04`
内存 `16GB`
处理 `*8CPU 2.4GHz`

# 预装软件

## 安装软件

安装vim编辑器
`sudo apt-get install vim`

安装rpm包管理器
`sudo apt-get install rpm`

安装alien管理器
`sudo apt-get install alien`



## 切换 `apt-get` 的源
备份源信息配置文件
`cd /etc/apt`
备份
`sudo cp sources.list sources.list.back`
编辑
`sudo gedit sources.list`
添加一下内容
阿里云的地址

```
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
```
刷新一下
`sudo apt-get update`
升级软件
`sudo apt-get upgrade`
如果出现部分索引文件下载失败,注释对应的下载地址即可



## 安装搜狗输入法

访问搜狗输入法
[官网!](https://pinyin.sogou.com/linux/)
安装按照要求下载后，使用默认软件下载打开就好




## 安装网易云音乐
访问云村[官网](!https://music.163.com/)
按照要求下载.

### 异常,点击无效果处理
首先修改启动脚本
`sudo gedit /etc/sudoers`
在最后面加一行,用户名为当前登录用户名
`用户名 ALL = NOPASSWD: /usr/bin/netease-cloud-music`
随后修改快捷方式
`sudo gedit /usr/share/applications/netease-cloud-music.desktop`
修改`Exec=netease-cloud-music %U`
为 `Exec=sudo netease-cloud-music %U`

可参考 [CSDN](https://blog.csdn.net/TJU_Tahara/article/details/83866289)



## 安装对比软件

访问网站 [下载网站!](https://www.scootersoftware.com/download.php)
下载对应版本的rpm安装包,用默认安装软件进行安装就好

## 安装 `TeamViewer`
访问网站 [下载!](https://www.teamviewer.com/en/download/linux/)



## 安装 `VNC` 远程连接

访问网站 [下载!](https://www.realvnc.com/en/connect/download/vnc/linux/)



## 安装 `JDK8`

访问网站 [下载!](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
到下载目录下,下载文件为`jdk-8u201-linux-x64.tar.gz`
创建JVM文件目录
`sudo mkdir -p /usr/lib/jvm`
解压到对应目录
`sudo tar -zxvf jdk-8u201-linux-x64.tar.gz -C /usr/lib/jvm/`
设置系统环境变量
编辑文件`sudo vim /etc/profile`
```
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_201
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH
```
为当前用户,刷新环境变量
`source /etc/profile`
查看是否安装成功
`java -version`和`javac -version`查看版本



## 安装 `Node` 环境
访问网站 [官网!](https://nodejs.org/zh-cn/)
下载文件`node-v10.15.3-linux-x64.tar.xz`
创建Node软件目录
`sudo mkdir -p /usr/local/node`
解压文件到对应的目录
`sudo tar -xf node-v10.15.3-linux-x64.tar.xz -C /usr/local/node/`
创建软连接到用户`bin`目录下,作为系统命令`node`和`npm`命令
```
sudo ln -s /usr/local/node/node-v10.15.3-linux-x64/bin/node /usr/local/bin/node
sudo ln -s /usr/local/node/node-v10.15.3-linux-x64/bin/npm /usr/local/bin/npm
```
查看安装情况
```
node -v
npm -v
```
安装淘宝cnpm
通过别名形式,安装.并把别名写入文件
```
alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"

# Or alias it in .bashrc or .zshrc
$ echo '\n#alias for cnpm\nalias cnpm="npm --registry=https://registry.npm.taobao.org \
  --cache=$HOME/.npm/.cache/cnpm \
  --disturl=https://npm.taobao.org/dist \
  --userconfig=$HOME/.cnpmrc"' >> ~/.zshrc && source ~/.zshrc
```

## 安装 `Pycharm`
下载
解压缩
在`bin`目录下执行安装命令
接受协议,下一步下一步.



## 安装 `idea`

下载
解压缩
在`bin`目录下执行安装命令




## 安装 `Eclipse`
访问官网[下载](!https://www.eclipse.org/)
安装



## 设置共享
安装文件
`sudo apt-get install open-vm-dkms`
重新安装`vmware-tools`



## 下载Git

命令
`apt-get install git`



## 安装 `H.264` 视频解码

命令
`apt install gstreamer1.0-libav`



## 安装 `WineHQ`

访问`https://wiki.winehq.org/Ubuntu_zhcn`
按照上面安装即可



## 安装 `aptitude`

用于解决依赖冲突,替换`apt-get`命令



## 安装 `gnome-tweak-tool`

用于更好的优化设置，如屏幕缩放



## 安装 `maven`

访问官网获得下载版本[官网](!http://maven.apache.org/)
下载解压缩到合适的目录
如`/usr/local/maven/apache-maven-3.6.0`
配置环境变量
修改文件`/etc/profile`，添加

```shell
export MAVEN_HOME=/usr/local/maven3
export PATH=${PATH}:${MAVEN_HOME}/bin
```
运行`source /etc/profile`刷新环境变量

### 更换`.m2`文件夹下载的仓库地址

修改`/conf`目录下的`settings.xml`文件，新增`<localRepository>`节点，节点内指向本地地址
`<localRepository>本地仓库地址.....</localRepository>`

### 更换maven源
修改`/conf`目录下的`settings.xml`文件，`<mirrors>`节点下新增
```xml
<mirror>  
    <id>nexus-aliyun</id>  
    <mirrorOf>central</mirrorOf>    
    <name>Nexus aliyun</name>  
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
</mirror>  
```

### 命令行使用
例如 Junit
```xml
<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

下载到本地.m2文件夹下，使用默认的下载仓库
`mvn dependency:get -DgroupId=junit -DartifactId=junit -Dversion=4.12`

部署到远程（把本地库指定jar上传到中央库）
`mvn deploy:deploy-file -DgroupId=com.zhongan -DartifactId=zaOpenapiSdk -Dversion=0.0.1 -Dpackaging=jar -Dfile=C:/Users/Administrator/Desktop/tmp/zaOpenapiSdk-0.0.1.jar -DrepositoryId=central -Durl=http://bb.com/artifactory/libs-release-local`

安装到本地（把指定的jar安装到本地库）
`mvn install:install-file -Dfile=C:/Users/aaa/Desktop/zaOpenapiSdk-0.0.1.jar -DgroupId=com.zhongan -DartifactId=zaOpenapiSdk -Dversion=0.0.1 -Dpackaging=jar`

拷贝到本地指定路径中（从本地库或者中央库拷贝）
`mvn org.apache.maven.plugins:maven-dependency-plugin:2.10:copy -Dartifact=com.zhongan:zaOpenapiSdk:0.0.1:jar -DoutputDirectory=C:/Users/aaa/Desktop/tmp/`

从指定远程仓库下载jar到指定路径
`mvn org.apache.maven.plugins:maven-dependency-plugin:2.10:copy -DartifactId=device-service -Dversion=0.0.74.5 -Dpackaging=jar -Dfile=C:/Users/Administrator/Desktop/tmp/zaOpenapiSdk-0.0.1.jar -DrepositoryId=central -Durl=http://bb.com/artifactory/libs-release-local`



## 安装 `anaconda`

`anaconda`提供对于python类库的支持更新。
下载地址[官网](!https://www.anaconda.com/)
选择合适的版本，这里下载的为`Anaconda3-2019.03-Linux-x86_64.sh`
将下载后的`.sh`文件赋操作权限，并执行`chmod +x Anaconda3-2019.03-Linux-x86_64.sh`
执行安装.`sudo ./Anaconda3-2019.03-Linux-x86_64.sh`
按照提示进行安装。安装时会提示阅读协议，输入`yes`同意协议；随后让你选择安装路径，默认即可，也可以指定。我这里指向`/usr/local/anaconda/anaconda3`。指向文件夹地址必须不存在，交由安装软件创建。如果目录已经存在，则需要在启动`.sh`文件时增加`-u`参数.
然后等待安装。
配置软连接，指向安装目录下的`/bin/conda`,便于命令行运行。软连接存在`/usr/bin/conda`
`sudo ln /usr/local/anaconda/anaconda3/bin/conda /usr/bin/conda`

### 修改源
参考[清华大学开源镜像](!https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)



## 安装 `gparted`

软件用于磁盘管理



## 安装 `shutter`
用于屏幕截图，及截图完成后编辑
`sudo apt-get install shutter`
如无法编辑，参考[博客](!https://www.cnblogs.com/jaxu/archive/2018/08/30/9561992.html)



#  用户与权限

## 创建 `Root` 用户

`sudo passwd root`
输入当前用户的密码后,指定root用户的密码.
随后就可以登录root用户了.



## 用户管理

查看当前系统有那些用户
`cat /etc/passwd`



## 用户组管理

查看当前系统有那些组
`cat /etc/group`



示例:

```bash
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
```

其格式为
`group_name:passwd:GID:user_list`

1. 第一字段：用户组名称；
2. 第二字段：用户组密码；
3. 第三字段：`GID`
4. 第四字段：用户列表，每个用户之间用,号分割；本字段可以为空；如果字段为空表示用户组为GID的用户名；



## 用户同时归属于多个组

1. 查看当前用户登录用户归属哪些组
   `groups`
2. 将当前用户 `${USER}` 添加 `-a` 到 `docker` 组
   `sudo gpasswd -a ${USER} docker`
3. 刷新 `docker` 组成员
   `newgrp docker`