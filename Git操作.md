---
title: Git操作
abbrlink: c4c88997
date: 2019-06-12 20:52:15
categories:
  - Git
tags: Git
---

# 介绍
使用Git命令进行各种操作
当前Git用户非root用户


### 配置公钥私钥
公钥:进行分发,放置在服务器上
私钥:妥善保管,保存在本地


### 初始化Git仓库

1.新建文件夹,并进入
`mkdir SpringCloud`
2.初始化仓库, `--bare` 创建裸仓库
`git init --bare SpringCloud.git`
3.克隆已经建好的仓库
`git clone git@168.55.1.73:/home/git/git/SpringCloud.git/`

### 添加文件
添加全部文件
`git add .` 
添加指定文件
`git add application.properties`


### 提交本地
提交到本地仓库,并说明提交内容 `-m` 表随附消息,必须填写
`git commit -m '提交内容'`


### 关联远程分支
添加远程地址,当未通过git拉取初始化时,通过关联后上传.
`git remote add origin 你的远程库地址`

### 获取远程分支,并与本地分支合并
远端仓库必须不为空,拉取后合并到本地主分支
`git pull --rebase origin master`

### 提交远程分支
推送到远程主分支
`git push -u origin master`
