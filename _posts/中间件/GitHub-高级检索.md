---
title: GitHub-高级检索
abbrlink: 647339ea
date: 2020-05-27 13:01:33
categories:
  - Git
tags: Git
---

>  GitHub 划水指南

<!--more-->



# 简介

## 浏览

### 热门栏目

-  [Trending](https://github.com/trending) 中可以获得本天/周/月内最受欢迎的项目
-  [Topics](https://github.com/topics) 为当前正在进行的讨论

## 检索

### 关键字检索

通过 `awesome` 关键字检索,可以检索出相关工具仓库. 
如 `awesome 识图工具`

### 检索条件

| 格式                    | 示例                   | 说明                                              |
| :---------------------- | ---------------------- | ------------------------------------------------- |
| language:<语言>         | language:Java          | 开发语言为 `Java` 的仓库                          |
| <关键字> in:name        | spring  in:name        | 仓库名中,包含 `spring` 关键字的仓库               |
| <关键字> in:description | spring in:description  | 仓库介绍中,包含 `spring` 关键字的仓库             |
| <关键字> in:readme      | spring in:readme       | Readme文档中,包含 `spring` 关键字的仓库           |
| <关键字> in:name,readme | spring  in:name,readme | 在仓库名或Readme文档中,包含 `spring` 关键字的仓库 |
| stars:>=<收藏数>        | stars:>=500            | 搜索收藏星星大于500的仓库                         |
| forks:>=<拷贝数>        | forks:>=500            | 收藏数大于500的仓库                               |
| followers:>=<关注数>    | followers:>=500        | 关注数大于500的仓库                               |



