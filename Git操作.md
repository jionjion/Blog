---
title: Git操作
abbrlink: c4c88997
date: 2019-06-12 20:52:15
categories:
  - Git
tags: Git
---



>  使用Git命令进行各种操作

<!--more-->



# Git 使用指南

当前使用 `git` 账户

## 账户

### 删除用户密码

删除 `git` 用户的密码,这样就被禁止登录了

`passwd -d git`



### 禁用账户登录Shell

修改 `/etc/passwd`  文件, 将其中的关于 `git` 用户的命令进行修改

`git:x:1001:1000::/home/git:/usr/bin/git-shell`



## 仓库

### 配置公钥私钥

公钥:进行分发,放置在服务器上
私钥:妥善保管,保存在本地



### 初始化Git仓库

1.新建文件夹,并进入
`mkdir SpringCloud`
2.初始化仓库, `--bare` 创建裸仓库
`git init --bare SpringCloud.git`
3.克隆已经建好的仓库
`git clone git@111.11.1.11:/home/git/git/SpringCloud.git/`





## 提交

### 添加文件

添加全部文件
`git add .` 
添加指定文件
`git add application.properties`



### 提交本地

提交到本地仓库,并说明提交内容 `-m` 表随附消息,必须填写
`git commit -m '提交内容'`



### 回退版本

将当前已经提交的代码回退一个版本.  `reset` 重置  `--soft`  软回滚 `HEAD~1` 回退一个版本
`git reset --soft HEAD~1` 



## 分支

### 查看分支

查看本地所有分支 `-a` 查询本地及远程所有分支信息
`git branch -a`



### 远程分支

添加远程地址,当未通过git拉取初始化时,通过关联后上传.
`git remote add origin 你的远程库地址`



### 拉取分支

远端仓库必须不为空,拉取后合并到本地主分支
`git pull --rebase origin master`


### 推送分支

推送到远程主分支
`git push -u origin master`
推送新分支到远程
`git push --set-upstream origin 新分支名称`


### 检出分支

从本地当前分支检出新分支,并切换到新分支
`git checkout -b 新分支的名字`

检出一个新分支,并不切换
`git branch 新分支的名字`



### 合并分支

合并一个其他分支到当前分支,图谱显示两个分支为并行开发
`git merge 其他分支`

合并一个其他分支到当前分支,并将其他分支的历史记录作为当前分支的历史变更记录 
`git rebase 其他分支`



### 移动分支

移动分支,会将当前 `HEAD` 指针指向的相对于当前最近分支的提交节点前的某个提交纪录中.
相对当前 `master` 分支,向前移动一步
`git checkout master^`

相对当前 `master` 分支,向前移动 `6` 步
`git checkout  master~6`



### 强制移动分支

会将任意分支的指向其他分支,无论之前是否有过相关提交记录

 使 `master` 分支指向 `HEAD` 的第三级父提交
`git branch -f master HEAD~3`

使分支 `master` 指向 `a4b99c`  提交
`git branch -f master a4b99c`



### 删除分支

删除本地分支 `-d` 删除分支
`git branch -d 分支名`

强制删除本地分支 `-D`
`git branch -D 分支名`



### 撤销本地分支

使用 `reset` 删除当前分支 `commit` 提交纪录, 但是本地变更还在且未加入暂存区
使当前 `HEAD` 指针所在分支版本,回滚上一个版本.
`git reset HEAD~1`



### 撤销远端分支

使用 `revert`创建一个新的 `commit` 提交纪录, 本次提交对上个版本做出的所有修改做出回滚操作,该删除的删除,该恢复的恢复.多用于远端分支
使当前 `HEAD` 指针所在分支版本新建一个提交纪录,用以冲销旧的修改
`git revert HEAD`



### 删除已跟踪的版本

使用 `rm` 命令, 将已经纳入到版本管理的文件进行剔除. 多用于剔除编译后的无用文件夹.
删除版本管理中的文件夹
` git rm -rf 文件夹名`

删除版本管理中的文件
`git rm -f 文件名`



### 合并指定提交

将某些提交修改拷贝到当前分支中.
比如:热修复了某些bug, 需要本次提交修复的内容同款提交
获得某次 `a8c26e` 的提交修改记录,并在当前分支作同样操作
`git cherry-pick a8c26e`

当然,可以指定多个提交记录,依次进行合并
`git cherry-pick a8c26e c15a9e`



### 修改提交顺序

交互式命令 `rebase` ,将当前指针指向的版本和之前的 `4` 个版本进行提交先后的排序修改,或者回滚提交记录
`git rebase -i HEAD~4`



## 操作记录