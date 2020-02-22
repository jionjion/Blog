---
title: Oracle基础-数据表篇
abbrlink: 3c29c5af
date: 2017-09-22 17:09:21
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

----------

## 内容简介

对Oracle数据库中数据定义和数据操作进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。

## 数据表
### 创建
创建一般的数据表
``` sql
--创建表
create table emp(            -- 创建的表名
empno    number(8),          -- 8位的整数
ename    varchar2(20),       -- 初始长度20位的可变字符串
job      varchar2(9),
mgr      number(8),
hiredate date, 				 -- 日期格式
sal      number(7,2),        -- 总共7位数,小数点后保留2位
comm     number(7,2),
deptno   number(4)
);
```
创建具有自增序列的表
``` sql
create table emp(            -- 创建的表名
empno    number(8) by default as identity(increment by 1 
                                            start with 1
                                            maxvalue 1000
                                            minvalue 0 
                                            cycle
                                            cache 20),  -- 自增序列
ename    varchar2(20),       -- 初始长度20位的可变字符串
job      varchar2(9),
mgr      number(8),
hiredate date, 				 -- 日期格式
sal      number(7,2),        -- 总共7位数,小数点后保留2位
comm     number(7,2),
deptno   number(4)
);
```

### 拷贝表

``` sql
-- 复制表,包括数据,需要要更高的管理权限,或者授权查看其他用户的表
create table emp 
as select * from scott.emp;                -- 将scoot用户下的emp表全部拷贝过来

-- 赋值表,指定数据
create table emp                 			--实现对指定字段的赋值,where 控制条件,union控制联合
as
select * from scott.emp where empno = 7521 union
select * from scott.emp where empno > 7800

-- 复制表,部分赋值
create table emp(empno,ename,job,sal,deptno)                     
as select empno,ename,job,sal,deptno from scott.emp; 	-- 将scoot用户下的emp表指定字段拷贝过来

-- 复制表结构,不包括数据
create table emp 
as select * from scott.emp                 
where 1=2;							-- 限制复制的记录数量,为0,则表示不复制数据
```

### 修改表字段属性
注意,如果为新增加的字段设置了默认值,则新增字段会在创建时自动写入默认值.对于大数据量表修改同一字段可以采用先删除该字段,再为添加具有默认值的该字段.完成批量更新,记得随后要将默认值去掉.
``` sql
-- 修改字段的默认值
alter table emp                                                             -- 要修改的表名
modify comm default '0';                                                    -- 默认字段的默认值
      
-- 追加表字段
alter table emp                                                             
add address varchar2(20);                                                   -- 增加字段address,初始长度为20的可变字符型
 
-- 修改表字段的名称
alter table emp
rename column empno to eno;                                                 -- 仅限于修改名称
 
-- 修改表字段的属性
alter table emp
modify empno number(10);                                                    -- 仅限于修改属性
 
-- 删除字段
 alter table emp drop column deptno;
 
-- 删除多个字段
 alter table emp drop (deptno,address);
 
-- 字段,设置无用列 
alter table emp set unused(deptno);
alter table emp set unused column deptno;

-- 删除无用列
alter table emp drop unused column;
 
 -- 设置字段可见/不可见,在表创建时也可以指定
alter table emp modify(ename invisible)
alter table emp modify(ename visible)
 
-- 增加字段的描述
comment on column emp.empno is '员工表的ID';

-- 增加表的描述
comment on table emp is '这是员工表';
```

### 元数据表
对数据库表的表对象进行描述的对象被称为元数据,作为数据库基础对象存放在不同的表中,常见的有

``` sql
-- 注释表信息
select * from user_tab_comments;

-- 注释字段信息
select * from user_col_comments;```

### 拷贝记录

​``` sql
-- 插入表数据(表结构相同)
insert into table_name_new                                                  -- 全部匹配字段,要求字段类型和数量对应
select * from table_name_old;    

-- 复制表数据(表结构不同)
insert into table_name_new(column1,column2...)                              -- 部分匹配,要求字段类型和数量对应
select column1,column2... from table_name_old;								

-- 插入数据
-- 1)全部插入
insert into emp
values (10000,'Jion','ANALYST','7782',sysdate,1200,100,20);
-- 2)部分插入 (空缺的为null)
insert into emp(empno,ename,job)                                            -- 为插入的部分表示为空
values (10000,'Jion','ANALYST');
-- 3)批量插入
insert into emp(empno,ename,job)                                            --插入查询出来的多条记录
select empno,ename,job from scott.emp where empno=7839 union
select empno,ename,job from scott.emp where empno=7844 union
select empno,ename,job from scott.emp where empno=7902 ;
```

### 更新记录

``` sql
--修改数据
update emp set ename='Jion'                                                 -- 修改表中字段对应数据
where empno=7902;                                                           -- 限制条件,如果不加默认修改该字段的所有记录
```


### 删除记录

``` sql
--删除数据
delete from emp                                                             -- 删除表中数据
where empno=7902;                                                           -- 限制条件,如果不加默认删除全部数据
```


### 清空表

``` sql
-- 清空表,速度快,保留表结构
truncate table emp;

-- 强制删除,忽略键
drop table emp cascade constraint;
```

### 删除表

``` sql
-- 删除表
drop table emp;                                                             -- 速度慢,删除表结构及相关约束
```


## 嵌套表
### 单数据类型
#### 语法
只具有一列指定数据格式的表结构
``` sql
-- 创建嵌套表类型
create [or replace] type 类型名称 as | is table of 数据类型 [not null]
```

可以在创建表时指定嵌套表列及储存空间
``` sql
-- 创建表指定嵌套表储存空间名称
create table 表名(
  字段名    字段类型
  ...
  嵌套字段  嵌套表类型
) nested table 嵌套表字段  store as 储存空间名称
```
#### 创建嵌套表类型

``` sql
-- 创建嵌套表类型
create or replace type dept_depiction as table of varchar2(200) not null;
```

#### 创建嵌套表

``` sql
-- 创建数据表
create table dept
(
  deptno number(2),
  dname  varchar2(14),
  loc    varchar2(13),
  depiction dept_depiction
) nested table depiction store as dept_depiction_table;
```

#### 增删改查

``` sql
-- 添加数据
insert into dept(deptno, dname, loc, depiction)
values ('10' , 'ACCOUNTING','NEW YORK',dept_depiction('描述:这是会计部门...','描述:部门位于纽约市...'));

insert into dept(deptno, dname, loc, depiction)
values ('20' , 'RESEARCH','DALLAS',dept_depiction('描述:这是研发部门...','描述:部门位于德克萨斯州...'));

-- 更新数据
update table (select depiction from dept where deptno = '20') temp  -- 起别名
set value(temp) = '修改后的描述...'  -- 更新字段
where temp.column_value = '描述:这是研发部门...';  -- 条件

-- 删除数据
delete from table (select depiction from dept where deptno = '20') temp
where temp.column_value = '修改后的描述...';

-- 查看数据
select * from dept where deptno='20';

-- 查询嵌套表中数据
select * from table (
  select depiction from dept where deptno = '20'
);
```

### 复核数据类型

####  创建嵌套表类型

``` sql
-- 创建对象数据类型
create or replace type dept_others_obj as object
(
  depiction varchar2(200),
  create_user number,
  create_date date
);
```

#### 创建嵌套表

``` sql
-- 创建表
create table dept
(
  deptno number(2),
  dname  varchar2(14),
  loc    varchar2(13),
  other  dept_others_nested
) nested table other store as dept_others_table;
```

#### 增删改查

``` sql
-- 插入数据
insert into dept(deptno, dname, loc, other)
values ('10' , 'ACCOUNTING','NEW YORK',dept_others_nested(
                                          dept_others_obj('这是会计部门',1,sysdate),
                                          dept_others_obj('地址在纽约',2,sysdate)));
-- 更新数据
update table (select other from dept where deptno = '10') temp  -- 使用别名,作为临时表
set value(temp) = dept_others_obj('这是会计部门!',1,sysdate)     -- 更新临时表,数据类型为对象数据类型
where temp.create_user = 1;                                     -- 条件,表.对象字段
-- 删除数据
delete from table (select other from dept where deptno = '10') temp
where temp.create_user = 1;
-- 查看数据
select * from dept;
-- 查看数据集
select * from table (select other from dept where deptno = '10' );
```

## 数组类型
### 单数据类型的数组
可变数据类型,与嵌套表相似,只不过只能有一列具有相同数据类型的结构,且长度根据创建时指定

#### 创建
创建可变数组,指定长度为3
``` sql
create or replace type other_varray is varray(3) of varchar2(100);
-- 使用可变数组创建数据表
create table dept(
  deptno number(2),
  dname  varchar2(14),
  loc    varchar2(13),
  other  other_varray
);
```

#### 增删改查

``` sql
-- 插入数据
insert into dept(deptno, dname, loc, other)
values ('10' , 'ACCOUNTING','NEW YORK',other_varray('这是会计部门','地址在纽约'));
-- 更新数据
update dept 
set other = other_varray('这是会计部门!','地址在纽约!')
where deptno = 10;
-- 删除数据
delete from dept 
where deptno = '10';
-- 查询
select * from dept;
select * from table( select other from dept where deptno = '10' );
```

### 复核数据类型的数组
#### 创建
首选需要创建一个对象类型

``` sql
-- 创建对象
drop type dept_others_obj;
create or replace type dept_others_obj as object
(
  depiction varchar2(200),
  create_user number,
  create_date date
);
```

创建可变数组,引用对象类型
``` sql
-- 创建可变数组
create or replace type other_varray is varray(3) of dept_others_obj;
```

创建数据表
``` sql
-- 使用可变数组创建数据表
create table dept(
  deptno number(2),
  dname  varchar2(14),
  loc    varchar2(13),
  other  other_varray
);
```

#### 增删改查

``` sql
-- 插入数据
insert into dept(deptno, dname, loc, other)
values ('10' , 'ACCOUNTING','NEW YORK',other_varray(
                                          dept_others_obj('这是会计部门',1,sysdate),
                                          dept_others_obj('地址在纽约',2,sysdate)));
-- 更新数据
update dept
set other = other_varray( dept_others_obj('这是会计部门',1,sysdate),
                          dept_others_obj('地址在纽约',2,sysdate))
where deptno = '10'; 
-- 删除数据
delete from dept
where deptno = '10';
-- 查看数据
select * from dept;
-- 查看数据集
select * from table (select other from dept where deptno = '10' );

```