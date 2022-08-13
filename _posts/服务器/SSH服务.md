---
title: SSH服务
categories:
  - SSH
tags:
  - SSH
abbrlink: 5d70bba
date: 2021-11-16 21:09:38
---

> Redis 操作记录

<!--more-->



# SSH服务

## CentOS系统升级

### 编译安装OpenSSL

```bash
# 查看版本 OpenSSL 1.1.1g FIPS  21 Apr 2020
openssl version

# 备份.将原文件备份为 xxxxx.bak
mv /usr/bin/openssl{,.bak}
mv /usr/include/openssl{,.bak}

# 下载新版本,到当前文件
wget https://www.openssl.org/source/openssl-1.1.1l.tar.gz

# 解压
tar xf openssl-1.1.1l.tar.gz

# 进入到解压目录编译安装OpenSSL
cd openssl-1.1.1l
./config shared && make && make install

# 查看编译结果. 默认安装到 /usr/local/ 文件夹下.
ll /usr/local/bin/openssl
ll -d /usr/local/include/openssl/

# 软连接, 将本地软件创建软连接到命令文件
ln -s /usr/local/bin/openssl /usr/bin/openssl
ln -s /usr/local/include/openssl/ /usr/include/openssl

# 选刷新文件库,将 /usr/local/lib64 目录作为系统编译目录
echo "/usr/local/lib64" >> /etc/ld.so.conf.d/usr-local.conf
/sbin/ldconfig

# 查看升级版本
openssl version
```



### 编译安装OpenSSH

```bash
# 备份
mv /etc/ssh{,.bak}

# 下载安装
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.8p1.tar.gz

# 解压
tar xf openssh-8.8p1.tar.gz

# 创建安装目录, 预将其编译安装到当前文件
mkdir /usr/local/openssh

# 编译
cd openssh-8.8p1
./configure --prefix=/usr/local/openssh --sysconfdir=/etc/ssh --with-openssl-includes=/usr/local/include --with-ssl-dir=/usr/local/lib64 --with-zlib --with-md5-passwords --with-pam && make && make install

# 备份二进制文件
mv /usr/sbin/sshd{,.bak}
mv /usr/bin/ssh{,.bak}
mv /usr/bin/ssh-keygen{,.bak}
 
# 创建二进制文件软连接
ln -s /usr/local/openssh/bin/ssh /usr/bin/ssh
ln -s /usr/local/openssh/sbin/sshd /usr/sbin/sshd
ln -s /usr/local/openssh/bin/ssh-keygen /usr/bin/ssh-keygen

# 查看版本
ssh -V

# 拷贝服务
cp openssh-8.8p1/contrib/redhat/sshd.init /etc/init.d/sshd
cp openssh-8.8p1/contrib/redhat/sshd.pam /etc/pam.d/sshd.pam

# 服务脚本添加执行权限
chmod +x /etc/init.d/sshd

# 重启服务
service sshd restart
```



## 阿里云升级

### 编译安装OpenSSH

```bash
# 安装关联软件包和编译工具包
yum update openssl openssh -y
yum install vim gcc gcc-c++ glibc make autoconf openssl-devel pcre-devel pam-devel zlib-devel rsync -y
yum install pam* zlib* -y

# 源码文件放置位置 /opt 目录下.
mkdir -p /opt/openssh || cd /opt/openssh

# 下载安装
wget https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.8p1.tar.gz

# 解压
tar -zxvf openssh-8.8p1.tar.gz

# 备份
mv /etc/ssh{,.bak}

# 编译, 安装到 /usr/local/openssh 文件夹, 指定 /etc/ssh 文件夹为默认配置文件夹
/opt/openssh/openssh-8.8p1
./configure --prefix=/usr/local/openssh --sysconfdir=/etc/ssh --with-md5-passwords --with-zlib --with-pam
make

# 先卸载openssh.不一定存在
rpm -e --nodeps `rpm -qa | grep openssh`

# 执行安装
make install

# 拷贝启动脚本,递归进行文件拷贝并打印.  rsync 命令,进行增量拷贝.
rsync -av ./contrib/redhat/sshd.init /etc/init.d/sshd
rsync -av ./contrib/redhat/sshd.pam /etc/pam.d/sshd.pam
chmod +x /etc/init.d/sshd

## 添加开机启动
chkconfig sshd on

# 拷贝用户命令,其实建立软连接更好..
rsync -avb /usr/local/openssh/bin/* /usr/bin/
rsync -avb /usr/local/openssh/sbin/* /usr/sbin/

# 通过服务脚本.重启服务
/etc/init.d/sshd restart

```



## 配置

### 服务配置

默认服务配置文件在 `/etc/ssh/sshd_config` 中.

```bash
# 支持协议版本
Protocol 2,1

# 登录时日志记录
SyslogFacility AUTHPRIV

# 允许Root登录
PermitRootLogin no

# 使用秘钥对登录
PubkeyAuthentication yes

# 使用用户名密码登录
PasswordAuthentication yes

# 设置超时时间
ClientAliveInterval 600
# 设置同时连接数
ClientAliveCountMax 2
```

