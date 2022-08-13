---
title: Hexo-说明
typora-root-url: ../
abbrlink: 589f47b6
date: 2021-11-30 08:39:56
tags:
---

> 介绍如何使用 `Hexo` 管理文章

<!-- more -->



## 基础命令
| 命令       | 说明                             |
| ---------- | -------------------------------- |
| `init`     | 初始化文章仓库                   |
| `config`   | 获取/设置配置项目                |
| `new`      | 新建一个文章                     |
| `publish`  | 从草稿箱发布到正式文章           |
| `generate` | 生成静态页面资源                 |
| `clean`    | 删除所有的已经渲染的静态页面资源 |
| `list`     | 查看当前文章列表                 |
| `server`   | 启动服务                         |
| `deploy`   | 部署网站                         |
| `version`  | 查看当前版本                     |
| `help`     | 帮助                             |


## 新建

### 新建文章

创建一个新的文章
`Hexo new "文章标题"`



## 启动

启动服务, 默认访问 ` http://localhost:4000`
`hexo server`

启动服务, 并指定端口. 自动打开浏览器
`hexo server --open --port 5000`



## 部署

部署到远端
`hexo deploy`



### 图片储存

图片储存

通过在文章 `mate` 部分中添加 `typora-root-url: ../` 将当前图片指向上级文件.并在插入图片时,复制图片到静态资源文件夹中..



![图片](./images/2021-11/u=1257797105,1266894652&fm=26&fmt=auto.webp)



