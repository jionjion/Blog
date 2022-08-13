---
title: Oracle基础-表分区篇
typora-root-url: ../
categories:
  - Oracle
  - SQL
tags:
  - Oracle
  - SQL
abbrlink: 88771a9d
date: 2022-07-24 10:41:21
---

> Oracle 表分区介绍

<!-- more -->



# 表分区

* 允许将表分为若干个分区
* 用户可以执行查询,只访问表中的特定分区
* 将不同的分区储存在不同的磁盘,以提高访问性能和安全性
* 可以独立的备份和恢复每个分区



# 分区指南

## 查看分区

```sql
-- 查看分区
select * from user_tab_partitions ;
```



## 范围分区

根据表的某一个列或者一组列的范围进行分区

```sql
-- 创建分区
create table emp(eno number , ename varchar2(200))
partition by range(eno)     -- 根据eno的列的值进行分区
(
    partition eno1 values less than(100), -- eno小于100时,进入第一分区
    partition eno2 values less than(200),
    partition eno3 values less than(300)  -- 大于300的,不能被录入
);
-- 查询指定分区
insert into emp values (200,'Jion');
select * from emp;
select * from emp partition(eno1);
-- 增加分区
alter table emp add partition eno4 values less than(maxvalue);  -- 增加分区,大于300的存入这个分区
-- 合并分区
alter table emp merge partitions eno3 , eno4 into partition eno4;
-- 拆分分区
alter table emp split partition eno4 at(3000) into (partition eno3 , partition eno4); -- 在3000点处,拆分分区
-- 截断分区
alter table emp truncate partition eno4;
-- 删除分区
alter table emp drop partition eno4;
```



## 散列分区

根据指定字段的散列来确定应放在哪一个分区中,将数据均匀地分到不同的分区之中

```sql
-- 创建分区
create table emp(eno number ,ename varchar2(200))
partition by hash(eno)
(
    partition eno1 , partition eno2   -- 根据散列放到其中一个
)
--查询指定分区
insert into emp values (200,'Jion');
select * from emp;
select * from emp partition(eno1);
```



## 列表分区

允许用户将不相关的数据组织在一起

```sql
-- 创建分区
create table emp(eno number , ename varchar2(200) , esex varchar2(10))
partition by list(esex)
(
    partition boy values('boy','man','he'),          -- 定义分区以外的元素不会被插入
    partition girl values('girl','woman','she'),
    partition other values(' ',null,'it')         
)
-- 查询指定分区
insert into emp values (200,'Jion','boy');      
select * from emp partition(boy);
```



## 复合分区

范围分区和散列分区;范围分区和列表分区相结合,被称为复合分区

```sql
  create table emp(eno number , ename varchar2(200) , esex varchar2(10))
  partition by range(eno) -- 首先根据eno范围分区
  subpartition by hash(esex)  -- 随后根据esex散列分区
  subpartitions 4   -- 哈希分区分4个
  (
      partition eno1 values less than(100),    -- 分区的范围情况
      partition eno2 values less than(200),    -- 每个范围分区内,有4个哈希子分区
      partition eno3 values less than(maxvalue)
  )
```



## 引用分区

由于外键引用父表的分区的方法,它依赖已有的父表子表的关系,子表通过外键关联到父表,进而继承了父表的分区方式而不用自己创建,子表还继承了父表的维护操作

```sql
-- 创建主表
create table student(sno number primary key , sname varchar2(20) , grade varchar2(10))
partition by range(sno)
(
    partition sno1 values less than(100),
    partition sno2 values less than(200),
    partition sno3 values less than(maxvalue)
);
-- 创建子表
create table score(cno number primary key , sno number not null , couse_name varchar2(20) , couse_value number,
                   constraint fk_score foreign key(sno) references student(sno))  -- 外键约束
partition by reference(fk_score); -- 引用外键,创建引用分区
```



## 间隔分区

完全自动地根据间隔值创建范围分区,它是范围分区的扩展

```sql
-- 创建表
create table emp(eno number , ename varchar2(20) , hiredate date)
partition by range(hiredate)  -- 范围分区
interval (numtoyminterval(1,'YEAR'))  -- 间隔每一年,新创建一个分区
(
    partition p_2018 values less than(to_date('2018-01-01','yyyy-MM-dd')) -- 原始分区
);
```



## 基于虚拟列

```sql
-- 创建表
create table emp(eno number , ename varchar2(20) , sal number , comm number,
                 tol as (sal + comm) virtual -- 虚拟列
                ) 
partition by range(tol)
(
    partition t_2000 values less than(2000),
    partition t_4000 values less than(4000),
    partition t_max values less than(maxvalue)
);

insert into emp(eno,ename,sal,comm) values(1,'Jion',100,50);
select * from emp;
```



## 系统分区

不指定分区依据,交由数据库自主决定分区.

```sql
-- 创建分区表
create table emp(eno number , ename varchar2(20))
partition by system
(
    partition e1 , partition e2 , partition e3 -- 系统分区,分3区
)
```

