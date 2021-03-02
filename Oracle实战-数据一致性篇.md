---
title: Oracle实战-数据一致性篇
abbrlink: 36f6c580
date: 2017-09-23 15:28:18
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---
> Oracle 数据一致性的使用

<!--more-->



## 内容简介

根据项目经验，对Oracle数据库使用中常涉及到的一些业务规则进行抽象整理，涉及到表结构同步、数据录入等内容。

## 行级数据
### 传入数据，存在则更新，不存在则插入

``` sql
begin
     update                                                                                    -- 更新语句
         emp e                                                                                 -- 准备更新的表
     set
         e.empno     = '809987',                                                               -- 更新的字段
         e.ename  = 'jion',
         e.job = 'salesman'
     where
         e.empno     = '809987'                                                                -- 用来判断该记录是否是新记录的字段
         and
         e.ename  = 'jion'       
         and
         e.job = 'manager';
     if sql%notfound then                                                                      -- 如果sql没有执行
         insert                                                                                -- 执行插入语句
         into
             emp (  empno , ename ,job , mgr ,hiredate , sal ,comm , deptno  )                 -- 插入的值
         values ( '809987',  'jion', 'manager', 7782,  to_date('1980-8-18','yyyy-mm-dd'), 1000, 200, 1 );
     end if;
 end;
```


## 表级数据
### 根据条件，同步两张表，存在刷新，不存在则新增

``` sql
merge into  copy_emp c
using (select * from emp where rownum <= 2) e                                                   -- 可以在where中设置条件
on (c.empno = e.empno) 
when matched then                                                                               -- 这里可以选择执行update语句或者delete
    update   set      
    -- s.id = c.id                                                                              -- 这里不需要将连接键进行更新
    c.ename = e.ename                                                                           -- 如果两个表的ename字段相同,则认为两条记录相同
when not matched then 
    insert (empno,ename,job) values (e.empno,e.ename,e.job);                                    -- 执行操作


```
