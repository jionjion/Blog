---
title: CentOS桌面系统
abbrlink: 42ead4a8
date: 2018-10-20 20:20:15
categories:
  - 服务器
  - CentOS
tags: [CentOS, Linux]
---
-------------------------------

# 共享文件夹方式
## 安装相关依赖

## 安装Tools工具包

## 创建共享文件夹

创建目录文件，并修改权限

## 将远程目录挂在文件夹下
`vmhgfs-fuse .host:/Hexo /mnt/hgfs/Hexo/`


# 安装Hexo服务
在安装完node环境之后
`cnpm install hexo-cli -g`

后续可以直接使用`hexo`命令完成为文档操作

# 安装WPS
安装依赖
`yum install libpng12`
`yum install libGLU`
去官网,http://community.wps.cn/download/
找到rpm安装包,美滋滋下载下,解压后安装即可
`rpm -ivh wps-office-10.1.0.6634-1.x86_64.rpm `
如果少了什么依赖,按照要求补就好
推荐网站 https://pkgs.org/
自求多福,救不了,告辞,溜


