---
title: Oralce基础-视图篇
abbrlink: f1df3097
date: 2017-09-23 09:26:51
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

> Oracle 视图的使用

<!--more-->



## 内容简介

对Oracle数据库中视图的使用进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。

## 视图

* 提供了另一种级别的表安全性,隐藏敏感字典
* 简化SQL查询
* 隔离基表的结构改变
* 通过重命名列,从另一个角度提供数据

## 关于视图与基表

- 键保留表:  	主表,其主键作为视图唯一区分,可以更新
- 非键保留表: 	不可以更新, `instead of`  触发器可以完成任何情况下的视图更新

更新视图后表也会更新的条件:

 - select语句在选择列表中没有聚合函数，也不包含top,group by,union(除非视图是分区视图)或distinct子句。聚合函数可以用在from子句的子查询中，只要不修改函数返回的值。  
 - select语句的选择列表中没有派生列。派生列是由任何非简单列表达式(使用函数、加法或减法运算符等)所构成的结果集列。  select语句中的from子句至少引用一个表。
 - select语句不能只包含非表格格式的表达式(即不是从表派生出的表达式)。

视图能够被更改的条件:

- 只能修改一个底层的基表
  * 如果为违反了基表的约束条件,不能更新
  * 如果视图含有连接操作符,distinct关键字,集合操作符,聚合函数,group by子句,则无法更新
  * 如果视图包含伪列或者表达式,则无法更新视图



## 视图的创建

### 语法
force:创建视图的表不存在也会创建视图, 当表创建后, 视图可以使用
noforce:(默认)创建视图的表必须存在,否则不会创建视图
or replace:存在则创建,否则则更新;

``` sql
create [force | noforce | or replace] view 视图名称[(别名1,别名2,...)] as
查询语句;
```

### 示例
``` sql
-- 创建视图的权限
grant create view to zhangqian;
 
-- 创建
create or replace view v_emp as
select empno,ename,job,sal from emp
order by empno;

-- 强制创建一个视图
create force view emp_lv as select * from emp;
 
-- 查看
select * from emp_view;

-- 查询当前用户视图
select * from user_views;
 
-- 修改,对于修改视图,原表也会改变
update emp_view set sal=3100
where empno=1001;

```
### 创建只读视图
`with check option`子句,保证视图的创建条件不被修改
` with read only`子句,使视图只读

``` sql
-- with check option子句,保证视图的创建条件不被修改
create [force | noforce | or replace] view 视图名称[(别名1,别名2,...)] as
子查询
[with check option [constraint 约束名称]]
-- with read only子句,保证视图不能被更新
```

### 示例

```sql
-- 创建一个视图,不能执行删除等,使结果集减少的操作
create view emp_lv as select * from emp
with check option;

-- 创建一个只读视图
create view emp_lv as select * from emp
with read only;
```



## 其他查询

查看所有视图

```sql
select * from user_views;
```
