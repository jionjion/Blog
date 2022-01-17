---
title: Oracle基础-数据库连接篇
abbrlink: f9a7bbe5
date: 2020-07-26 18:56:35
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

> Oracle 使用 DBLink

<!--more-->



# 简介

通过 `DBLike` 连接其他数据库,以便进行同步操作.

## 语法

创建 `DBLink` 

```sql
create database link 连接名称
  connect to 数据库用户名 identified by "数据库密码" using
  '(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=数据库服务器IP)(PORT=数据库服务器端口))(LOAD_BALANCE=yes)(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=数据库服务名)))';
```

## 示例

```sql
-- 查询当前用户的所有 dblink
select * from all_db_links;

-- 删除
drop database link dblink_name;

-- 创建
create database link dblink_name
  connect to username identified by "password"  using
  '(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=xx.xx.xxx.xxx)(PORT=1521))(LOAD_BALANCE=yes)(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=XXXX)))';
```

