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



## 物化视图

  包含一个查询结果的数据库对象,它是远程数据的本地副本,或者用来生成基于数据表求和的汇总表.
  物化视图储存基于远程表的数据,又称为快照
  物化视图可以查询表.视图,和其他物化视图
  物化视图中的数据是真是存在的,但是为只读的

### 技术点

#### 数据复制技术
物化视图 DG 消息队列

#### 查询重写

`enable query rewrite` / `disable query rewrite`
当数据库查询时,自动判断是否通过查询物化视图来获得结果

#### 物化视图日志

根据不同物化视图的刷新速度,创建为 `rowid` 或者 `primary key` 类型的

#### 刷新模式

`on demand` / `on commit`
用户在需要时刷新 和 用户执行DML语句时刷新
`dbms_mview.refresh` 进行手动刷新
默认为 `on demand force`

#### 刷新方式

`fast` / `complete` / `force` / `never`
快速刷新,只刷新修改的
立即刷新,完全刷新
强制刷新,自动判断,并执行以上两者之一
从不刷新

### 物化视图方式

#### 基于主键的物化视图

```sql
-- 授权
grant create materialized view to zhangqian;
grant select on zhangqian.MLOG$_EMP to zhangqian;

-- 0.当远程主表中必须含有主键
-- 1.远端,创建主表,并创建基于主表的视图日志
create table emp(empno number(4) primary key, ename varchar2(10), sal number(7,2));  -- 创建主表,带主键
create materialized view log on emp;     	-- 创建物化视图的日志

-- 查看当前用户表, MLOG$_EMP和RUPD$_EMP分别为物化视图日志表
select * from user_tables;
-- 支持可更新的表
select * from RUPD$_EMP;
-- 物化视图的真正日志
select * from MLOG$_EMP;

-- 2.本地,创建一个过程,实现刷新本地物化视图动作;创建一个job,进行调用过程的操作;运行这个job,则本地数据会与远端数据自动同步刷新一次
create materialized view emp_mtlv        	-- 创建物化视图
refresh fast                             	-- 刷新方式[可省]
start with sysdate                       	-- 刷新开始时间[可省]
next sysdate + 1/1440                    	-- 下次刷新时间,每分钟刷新[可省]
with primary key                         	-- 物化视图基表含有主键[可省]
as
select * from zhangqian.emp           	 	-- 物化视图的基本表
```

#### 使用定时任务刷新

```sql
-- 使用定时任务刷新
create or replace procedure refresh_emp_mtlv is
begin
  -- 手动刷新物化视图数据
  dbms_mview.refresh('EMP_MTLV'); 
end ; 

-- 定时任务调用
declare job_id number;
begin
  -- 提交定时任务,  返回jobId,定时执行的过程,执行开始时间,执行结束时间
  dbms_job.submit(job_id, 'refresh_emp_mtlv', sysdate , sysdate+1/1440);
  -- 定时任务执行
  dbms_job.run(job_id);
end ;
```

#### 基于rowid的物化视图

```sql
-- 0.远端主表没有主键
-- 1.远端,创建表,无需创建表日志
create table emp(empno number(4), ename varchar2(10), sal number(7,2));  -- 创建主表,带主键

-- 2.本地,创建一个过程,定时刷新
create materialized view emp_mtlv        	-- 创建物化视图
refresh with rowid                       	-- 必填,刷新方式 rowid
start with sysdate                       	-- 刷新开始时间[可省]
next sysdate + 1/1440                    	-- 下次刷新时间,每分钟刷新[可省]
as
select * from zhangqian.emp
```

#### 相关查询

```sql
drop table emp;  								-- 删除表时,自动删除日志表
drop materialized view emp_mtlv;  				-- 删除物化视图

select * from emp;								-- 查询基表
select * from emp_mtlv;							-- 查询物化视图
```
