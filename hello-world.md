---
title: Hello World
abbrlink: 4a17b156
date: 2017-01-01 00:00:00
categories:
  - Hexo
tags: Hexo
---

> 介绍 Hexo 作为博客的部署

<!--more-->



# Hexo静态博客

## 快速开始

### 创建一个新的页面

``` bash
$ hexo new "My New Post"
```

### 清除已有缓存
``` bash
$ hexo clean
```


### 开启服务器

``` bash
$ hexo server
```


### 生成静态页面

``` bash
$ hexo generate
```

### 发布页面到远端服务器

``` bash
$ hexo deploy
```



## 命令列表

| 命令       | 说明                                   | 示例                     | 示例说明                                          |
| ---------- | -------------------------------------- | ------------------------ | ------------------------------------------------- |
| `init`     | 在当前文件夹下初始化`Hexo`             |                          |                                                   |
| `config`   | 显示/设置当前配置文件信息              |                          |                                                   |
| `new`      | 创建一个新的博客文章                   | `hexo new '新的博客'`    |                                                   |
| `list`     | 显示当前站点下文件列表信息             |                          |                                                   |
| `generate` | 生成静态文件                           |                          |                                                   |
| `clean`    | 删除当前生成的静态文件及缓存.          |                          |                                                   |
| `server`   | 启动当前服务器,并随当前文档变动而修改. | `hexo server -p 4000 -o` | 开放4000端口运行站点服务,并用默认浏览器打开主页面 |
| `deploy`   | 发布静态站点信息到远端服务器           |                          |                                                   |
| `publish`  | 将博客从草稿箱移动到文章文件夹下       |                          |                                                   |
| `migrate`  | 将您的博客从其他网站迁移到Hexo平台     |                          |                                                   |
| `render`   | 使用特定渲染插件,渲染文章              |                          |                                                   |
| `version`  | 显示当前信息                           |                          |                                                   |
| `help`     | 显示帮助信息                           |                          |                                                   |



![游戏人生](./hello-world/no_game_no_life.png)

