---
title: Oracle基础-同义词篇
typora-root-url: ../
date: 2022-07-24 11:41:18
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

## 同义词

现有对象的别名,可以简化SQL对象,隐藏对象的名称和所有者,提供对象的公共访问类型

- 私有同义词  只能在其模式内访问,且不能与当前模式的对象同名
- 共有同义词  可以为所有用户访问



## 语句

```sql
-- 授权同义词创建
grant create synonym to zhangqian;
grant create public synonym to zhangqian;

-- 建私有同义词
create synonym se for scott.emp;

-- 创建共有同义词
create public synonym se for scott.emp;

-- 创建或删除
create or replace synonym se for scott.emp;  

-- 查询当前用户的所有同义词对象
select * from user_synonyms;

-- 查询当前用户所能访问的所有同义词对象
select * from all_synonyms;  

-- 使用同义词
select * from se;

-- 删除同义词
drop synonym se;

-- 删除共有同义词
drop public synonym se;
  
-- 查看当前用户的全部对象
select * from all_objects;
```

