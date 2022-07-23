---
title: Oracle基础-约束篇
abbrlink: 283f4b9a
date: 2017-09-22 22:53:01
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

> Oracle 表结构约束的使用

<!--more-->



## 内容简介

对Oracle数据库中约束进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。

| 约束 | 说明 |
| ---- | --- |
| `not null`    | 不能为空            |
| `unique`      | 唯一                |
| `check`       | 检查约束            |
| `primary`     | 主键约束            |
| `foreign key` | 外键约束            |
| `扩展约束`    | 如,JSON格式检查约束 |

## 查看约束

``` sql
-- 查看指定表的约束
select * from user_constraints 
where table_name='EMP';                                                     -- 表名字为系统创建,必须大写.
```


## 非空约束

``` sql

-- 创建表时,设置非空约束
create table emp(
empno number(2)  not null,                                                  -- not null 不为空
ename varchar2(10)
);
 
-- 修改非空约束
alter table emp
modify empno number(2) null;                                                -- null表示可以为空

-- 创建表时,设置字段级主键约束
create table emp(
empno number(2) primary key,                                                -- 在创建表时指定主键字段 (唯一,且非空),主键名交由系统创建
ename varchar2(10)
);

-- 创建表时,设置表级主键约束
create table emp(                                                           -- 创建一个新表
empno number(2),
ename varchar2(10),                                                         -- 注意字段声明后逗号结束,再声明段级的约束
constraint pk_empno                                                         -- pk表示主键
primary key(empno)
);
```
## 主键约束

``` sql
-- 创建表时,设置表级联合主键
createtable emp(
empno number(2),
ename varchar2(10),
constraint  pk_empno_ename
primary key (empno,ename)                                                   -- 联合主键,字段内的值组合不能有重复
);


-- 增加主键约束
alter table emp
add constraint pk_empno                                                     -- 只能增加创建之前没有创建的,修改只能删除后重新定义
primary key (empno);
 
-- 更改主键的名字
alter table emp                                                             -- 有问题!!!
rename constraint emp to pk_empno;
 
-- 禁用主键
alter table emp
disable constraint emp;
 
-- 查看主键状态
select status from user_constraints where table_name='EMP';                  -- 引号内的,默认是系统生成的文字,大写. 
 
-- 启用主键
alter table emp
enable constraints pk_empno;
 
-- 删除主键
-- 1)指定名字删除
alter table emp
drop constraint pk_empno;
 
-- 2)直接删除主键
alter table emp
drop primary key;                                                            -- 直接删除主键

```

## 外键约束

``` sql
-- 外键约束 (需要主表的主键,键的数据类型应该一致
-- 1)列级设置
create table emp (
empno number(2),
ename varchar2(10),
deptno number(2) references dept (deptno)                                    -- 如果主表主键为联合主键,则都需要写出   
);
 
-- 2)表级设置
create table emp(
empno number(2),
ename varchar2(10),
deptno number(2),                                                            -- 注意创建完后,添加逗号分隔.
constraint fk_deptno                                                         -- 外键约束
foreign key (deptno)                                                         -- 从表
references dept(deptno)                                                      -- 主表
on delete cascade                                                            -- 选项,及联删除,主表删除时,从表该数据也删除
);
 
-- 增加外键约束
alter table emp
add constraint pk_deptno                                                     -- 外键约束的名称不能同名
foreign key (deptno)
references dept(deptno);
 
-- 禁用外键约束
alter table emp
disable constraint pk_deptno;

-- 启用外检
alter table sc3
enable constraint pk_deptno;
 
-- 查看外键约束的状态
select constraint_name,constraint_type ,status 
from user_constraints 
where table_name='EMP';

-- 删除外键约束
alter table emp
drop constraint pk_deptno;
```
## 唯一约束
空并不受唯一约束限制
``` sql
-- 唯一约束,确保表中数据的唯一.                             	
-- 区别于主键约束:1.可以为空,可以有多个,可以为多个字段联合生成唯一约束
-- 1)列级设置
create table emp(
empno number(2) unique,
ename varchar2(10) unique);

-- 表级设置
create table emp(
empno number(2),
ename varchar2(10),
constraint uk_empno_ename                                                    -- 指定唯一约束的名字
unique (empno,ename));                                                       -- 当多条字段为唯一约束时,只用在其后用逗号分隔即可
 
-- 在创建表后修唯一约束
alter table emp                                              
add constraint uk_empno                                                      -- 添加约束的名字
unique(empno);                                                               -- 指定唯一约束的字段
 
-- 禁用唯一约束
alter table emp
disable constraint uk_empno;
 
-- 查询约束状态
select constraint_name,constraint_type,status 
from user_constraints 
where table_name = 'EMP';
 
-- 启用唯一约束
alter table emp
enable constraint uk_empno;
 
-- 删除唯一约束
alter table emp
drop constraint uk_empno;

```

## 检查约束

``` sql
-- 检查约束 ,可以存在多个,对多个字段进行检查
-- 1)列级设置
create table emp(
empno number(2),
ename varchar2(10),
sal number(3) check(sal between 100 and 3000 )                               -- 注意,between中,小数在前,大数在后
);
      
-- 2)表级设置
create table emp(
empno number(2),
ename varchar2(10),
sal number(3),
constraint ck_sal
check (sal between 100 and 3000 )                                           -- 通过在表的结束后追加设置,完成检查约束
);

-- 修改增加检查约束 
alter table emp
add constraint ck_sal
check (sal between 100 and 3000 );

-- 禁用检查约束
alter table emp
disable constraint ck_sal;
 
-- 查看检查约束的状态
select constraint_name,constraint_type,status 
from user_constraints  
where table_name = 'EMP';
 
-- 启用检查约束
alter table emp
enable constraint ck_sal;                                                   -- 如果记录中有不满足约束的条件,其不可以再修改回去检查约束
 
-- 删除约束
alter table emp
drop constraint ck_sal;
```

## 位图索引
使用压缩格式存放数据,便于检索具有大量重复数据的列

``` sql
create bitmap index emp_deptno_idx on emp(deptno);
```