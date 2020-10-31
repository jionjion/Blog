---
title: 阿里云-CentOS操作
abbrlink: 8490a1b3
date: 2018-06-20 12:32:42
categories:
  - 服务器
  - CentOS
tags: CentOS
---

## 更换YUM源
yum的文件以.repo结尾,可以存在多个,便于对未被标准源收录的下载地址进行扩充.
#### 备份

将 `/etc/yum.repos.d/`文件夹下的CentOS-Base.repo剪贴备份
``` shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.back
```
### 更换下载文件
下载CentOS7对应的文件,到指定目录下
``` shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
其他版本的只需要修改后边的`-7`为指定版本号即可

### 刷新缓存

``` shell
yum makecache
```

## 安装JAVA8的JRE环境
下载Sum平台的JDK.这里使用rpm包进行安装
推荐网站`https://pkgs.org`,这里有Linux各版本需要的npm包.

### 创建下载
复制下载地址,使用`wget`命令,下载到服务器中
``` shell
wget http://mirror.yandex.ru/fedora/russianfedora/russianfedora/nonfree/el/updates/7/x86_64//java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64.rpm
```
### 执行安装

npm包便于安装,下载后直接安装即可,不需要另外另外配置环境变量

``` shell
rpm -ivh java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64.rpm
```


### 异常排查
如抛出依赖异常,则根据提示安装需要的依赖即可
大概也就十来个,不多,半个小时搞定,需要的rpm包去上面的网站上查找

``` shell
warning: java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 1453936d: NOKEY
error: Failed dependencies:
	java-1.8.0-oracle-headless = 1.8.0.172-3.el7.R is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libawt.so()(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libjava.so()(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libjava.so(SUNWprivate_1.1)(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libjvm.so()(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libjvm.so(SUNWprivate_1.1)(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libX11.so.6()(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libXext.so.6()(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libXi.so.6()(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libXrender.so.1()(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
	libXtst.so.6()(64bit) is needed by java-1.8.0-oracle-1.8.0.172-3.el7.R.x86_64
```

## 安装JAVA8的JDK环境
去官网下载对应版本的JDK开发包程序,下载到桌面本地
`http://www.oracle.com/technetwork/java/javase/archive-139210.htm`


## 上传到服务器
将下载后的rpm包上传

## 安装
`rpm -ivh jdk-8u144-linux-x64.rpm`


## 安装Tomcat引擎
### 下载
下载对应的rpm包

``` shell
wget https://harbottle.gitlab.io/harbottle-main/7/x86_64/00761142-tomcat8/tomcat8-8.5.31-1.el7.harbottle.x86_64.rpm

wget 
```

### 安装

``` shell
rpm -ivh tomcat8-8.5.31-1.el7.harbottle.x86_64.rpm
```

### 安装目录

| 目录 | 位置 |
| ---- | ---- |
| 系统配置位置| `/etc/tomcat8/server.xml` |
| 系统配置位置| `/etc/tomcat8/` |
| 服务位置 | `/usr/lib/systemd/system/tomcat8.service`|
| jar包位置 | `/usr/libexec/tomcat8/`|
| 本地jar包位置| `/usr/share/tomcat8/` |
| 启动脚本位置 | `/usr/libexec/tomcat8/startup.sh` |
| 停止脚本位置 | `/usr/libexec/tomcat8/shutdown.sh`|
| WEB存放目录    |  `/var/lib/tomcat8/webapps/`  , `/var/lib/tomcat8/webapps/`  |
| 日志位置  |   `/var/log/tomcat8/`  , `/var/log/tomcat8/` |
| 缓存位置 | `/var/cache/tomcat8/work/` |


### 命令
`service tomcat8 start`和 `service tomcat8 stop` 进行启动/停止
`service tomcat8 status`查看服务状态

### 安装管理界面开启管理员
默认服务器在启动后,访问8080端口会进如到Tomcat内置的管理界面.
如果没有,去官网下一个完整版,拷贝项目到`webapp`目录下运行即可
开启管理员角色,便于项目控制
修改`/etc/tomcat8/tomcat-users.xml`在`<tomcat-users>`标签中加入
``` xml
<role rolename="manager-gui"/>
<user username="root" password="root" roles="tomcat,role1,manager-gui,manager-script,manager-jmx,manager-status"/>
```
重启后管理员角色用户生效


## 安装mySQL
这里使用yum方式,安装mySQL数据库.

### 官网下载yum源
通过访问官网,获得想要现在的版本源
`https://dev.mysql.com/downloads/repo/yum/`

### 安装

``` shell
yum install -y mysql-server mysql mysql-devel
```

### 目录
配置文件位置.
`/etc/my.cnf` 配置文件

### 创建root用户
这里安装的为8.0版本的MySQL,在安装时系统会自动创建root用户和初始用户密码.
默认放在`/var/log/mysqld.log`下,会有这么一句
`A temporary password is generated for root@localhost: XXXXXX`,
登陆后只能使用`alter user 'root'@'localhost' identified by '新密码'`修改密码

但是,我忘了修改后的密码 = =
修改`/etc/my.cnf`下的配置文件.追加一行
`skip-grant-tables` 表示不启用权限控制
然后登陆`mysql -u root -p`免密登陆root用户
刷新权限`flush privileges;`
使用`alter user'root'@'localhost' identified with mysql_native_password by '新密码';
`修改为新密码,默认密码必须有大小写特殊字符构成,否则会提示你密码过于简单,不能修改
退出,注释掉修改后的文件

### 开启远程连接
默认数据库创建时只能本地连接,如果需要开启远程连接,需要修改权限表
修改`mysql`数据库下的user表,将`host`字段改为`%`即为不限制连接主机IP

``` sql
use mysql
update user set host='%' where user='root';
flush privileges;
```


### 命令
`service mysqld start` 启动
`service mysqld stop` 停止
`service mysqld status` 查看状态

### 查看安装目录
yum安装实际为通过rpm方式进行的软件,因此,实际以rpm命令查询软件的安装位置

`rpm -qa | grep mysql` 查询mysql相关的rpm包
`rpm -ql mysql-community-server-8.0.11-1.el7.x86_64` 查询这个rpm包的具体安装位置


## 安装Python3运行环境
### 下载源码包,并解压
去官网下载对应的源码包,放到服务器本地.
解压后出现Python-3.6.5文件夹.
``` shell
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz

tar -xvf Python-3.6.5.tgz 
```
### 创建安装目录并编译
因为下载的为源码包,因此需要经过编译才可以安装,在编译之前,需要创建一个文件夹,作为将来编译后的安装目录
`mkdir /usr/local/python3`
随后,在解压后的源码目录中执行编译命令,生成编码文件
`./configure --prefix=/usr/local/python3`
分别执行`make`和`make install`进行安装
安装后执行文件为`/usr/bin/python`,

### 创建软连接

使用命令` ls /usr/bin/ | grep python`,可以查看命令目录下,可以执行的python打开方式.

``` shell
[root@iZ2ze8d8xqvlhuia12n5kbZ bin]# ls /usr/bin/ | grep python
python
python2
python2.7
python3
```


因为系统自带有python2.7的关系,需要修改原有连接并创建一个新的连接,指向`python`关键字
软链接类型Windows中的快捷方式,可以快速指向一个文件或者程序
备份原有连接,使其失效
`mv /usr/bin/python /usr/bin/python_bak`
创建新链接,指向`python`关键字
`ln -s /usr/local/python3/bin/python3 /usr/bin/python`

### 修改PIP的安装目录
`pip -V`查看当前pip服务的版本和python目录,如果与当前系统版本不一致,需要修改
原有pip的备份
`mv /usr/bin/pip /usr/bin/pip_back`
创建新的pip链接
`ln -s /usr/local/python3/bin/pip3 /usr/bin/pip`
新增环境变量

``` shell
PATH=$PATH:$HOME/bin:

PATH=$PATH:$HOME/bin:/usr/local/python3/bin
```

升级,测试当前pip是否可用
`pip install --upgrade pip`


### 测试
输入 `python -V`查看python安装版本
输入`pip -V`查看pip安装版本


## 安装Redis数据库
### 下载
去官网 `https://redis.io/` 下载最新版的redis服务,官网有安装方式教程.
需要注意的是,这里下载编译后直接生成运行文件,并不是rpm安装包.

### 编译安装
下载对应的版本
``` shell
$ wget http://download.redis.io/releases/redis-4.0.10.tar.gz
$ tar xzf redis-4.0.10.tar.gz
$ cd redis-4.0.10
$ make
```

### 创建连接便于启动
将解压目录和安装目录整体拷贝到`usr/local`下,作为用户安装程序
顺便修改目录名
``` shell
$ cp -r redis-4.0.10  /usr/local/
$ mv redis-4.0.10 redis
```
这样可以通过 `/usr/local/redis/src/redis-server` 启动服务
通过 `/usr/local/redis/src/redis-cli`进入本地客户端交互界面
通过 `/usr/local/redis/src/redis-cli shutdown`本地客户端停止服务

### 开启远程连接权限,容灾
还不会 = =!

### 创建自定义服务脚本
还不会 = =!

## 安装Node.js环境
### 下载安装包
`wget https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.xz`

### 解压安装
`xz -d node-v10.9.0-linux-x64.tar.xz`
`tar xvf node-v10.9.0-linux-x64.tar`

### 归档解压文件
`mv ./node /usr/local/node`

### 将文件路径写入环境变量
`export PATH=$PATH:/usr/local/node/bin`

刷新环境变量
`source /etc/profile`

### 修改NPM命令为CNPM
这里,将通过别名,创建`cnpm`命令,实际为`npm`命令指向淘宝镜像仓库
```
alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"
```

## 安装Python环境
不知道什么时候安好的,就不记录了



## 安装Docker服务
检查内核版本,必须在3.10以上
`uname -r`
如果不是,通过yum升级
`yum update`
安装Docker软件
`yum install docker`
启动服务
`systemctl start docker`
开机启动
`systemctl enable docker`
停止
`systemctl stop docker`



## 用户管理

### 新用户

创建新用户,并分配密码

```bash
# 创建组
groupadd docker

# 创建用户 docker ,并分配组
useradd docker -g docker

# 修改 docker 用户密码
passwd docker

# 允许管理员命名 sudo

```



### 允许使用 `sudo`

编辑权限文件, 通过命令 `sudo visudo` 进入编辑. 在其中,加入需要启用的用户

```
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
docker  ALL=(ALL)       ALL
```





### 用户添加公钥私钥访问

思路: 在远程生成公钥和私钥,公钥作为认证 `authorized_keys` 文件, 私钥下载到本地电脑,作为连接条件

```bash
# 生成公钥和私钥文件 在用户目录 .ssh
ssh-keygen -t rsa

# 公钥重命名为
mv id_rsa.pub authorized_keys

# 私钥重命名
mv id_rsa id_rsa_docker

# .ssh权限设置为700，公钥文件authorized_keys 644
chmod 700 ../.ssh/
chmod 644 authorized_keys

# 下载私钥,到本地,作为连接认证
```



### 登录访问策略

修改配置文件 `/etc/ssh/sshd_config` , 修改后重启服务 `service sshd restart`

```bash
# 禁用root远程登录
PermitRootLogin no
# 禁用普通用户通过密码登录
PasswordAuthentication no
# 启用秘钥认证登录
PubkeyAuthentication yes
```



### Windows秘钥权限设置

参考 [博客](https://blog.csdn.net/joshua2011/article/details/90208741)