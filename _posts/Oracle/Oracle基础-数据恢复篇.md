---
title: Oracle基础-数据恢复篇
abbrlink: efd59e60
date: 2017-09-23 15:38:49
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---
> Oracle 数据恢复解决

<!--more-->



## 内容简介

对Oracle数据库中单表的数据状态进行恢复，利用闪回技术可以实现任意时间点或者时间段的表状态进行回复恢复。

## 数据恢复常用方法

### 打开闪回权限
``` sql
ALTER TABLE dept ENABLE row movement;
```
### 关闭闪回

``` sql
alter table dept disable row movement
```

### 使用闪回恢复表内容到指定时间点
后面的参数为要还原的时间点
``` sql
-- 闪回表 , 前提,启用行迁移
flashback table table_name to timestamp | scn
-- 示例
flashback table dept to timestamp to_timestamp('2017-09-18 00:00:00','yyyy-mm-dd hh24:mi:ss');
```

### 闪回表

```sql
flashback table dept to before drop ;
flashback table dept to before drop rename to new_dept;
```

### 使用快照查询某时间点的数据

查询过去100分钟前的数据

``` sql
select * from dept AS OF TIMESTAMP  (SYSTIMESTAMP - INTERVAL '100' MINUTE)     
```

查询某个点的数据

```sql
select * from dept as of timestamp to_date('2018-3-27 18:30:00','yyyy-MM-dd hh24:mi:ss');
select * from dept as of timestamp to_timestamp('2017-09-18 00:00:00','YYYY-MM-DD HH24:MI:SS');
```

### 使用CSN恢复
SCN提供了Oracle的内部时钟机制，可被看作逻辑时钟，在事物提交时，它被赋予一个唯一的标示事物的SCN
SCN系统变化号,可以根据时间点或者SCN查询数据,依赖回滚段中的前镜像获得,通过设定 `undo_retention` 保留的最长时间

```sql
select ... as of scn 变化号 |timestamp 时间戳;
```

将删除时间转换为scn ,注意时间不能早于数据库安装时间

``` sql
select timestamp_to_scn(to_timestamp('2017-09-18 10:00:00','YYYY-MM-DD HH:MI:SS')) from dual; 
```

根据CSN时钟查询数据

``` sql
-- 获得当前SCN,系统管理员
select dbms_flashback.get_system_change_number from dual;
-- 查询历史数据
select * from dept AS  OF SCN 2269794;
```

### 查询回收站中删除的表

``` sql
select object_name,original_name,partition_name,type,ts_name,createtime,droptime from recyclebin;
```

### 事物版本查询

```sql
-- 闪回版本查询,对每次修改的数据进行查询
select ... from ... versions between   -- select之后可以跟伪列,获得事务的开始.结束.SCN号,ID号
select * from emp version between  scn|timestamp  and scn|timestamp

-- 事务开始时间 结束时间 事务号 操作
select versions_starttime , versions_endtime, versions_xid , versions_operation , 
empno, ename, job, sal 
from emp versions between timestamp  minvalue and maxvalue;

-- 撤销事务,回滚指定时间点,管理员权限 , 通过执行 undo_sql 可以撤销事务修改
select * from flashback_transaction_query t where t.table_name = 'EMP';
```

### 闪回删除的表

```sql
-- 闪回删除.将表从回收站中找到,及其相关索引触发器
drop table emp;
-- 查询回收站信息
select * from recyclebin;
-- 闪回删除到删除之前
flashback table emp to before drop;
-- 闪回指定表
flashback table BIN$MvC9PCFnTRymv7VC8EFv0w==$0 to before drop;
-- 查询回收站中,表名
select * from "BIN$MvC9PCFnTRymv7VC8EFv0w==$0";
-- 清空回收站
purge recyclebin; 
-- 清空某个文件
purge table emp;
-- 修改数据库配置参数,删除表不经过回收站
alter session set recyclebin = off;
```



## 回滚数据库

```sql
最终技能,闪回数据库
1.配置数据库为归档模式
startup mount;               -- mount状态打开数据库
alter database archivelog;   -- 配置归档
alter database openl         -- 打开数据库
2.配置闪回恢复区
show parameter db_recovery_file_dest;
3.配置闪回保留时间
show parameter db_flashback_retention_target; -- 显示保留持续记录数据库变化的时间,默认24h
4.启用数据库闪回
alter database flashback on; -- mount状态下执行
5.进行闪回数据库 
flashback database to timestamp | scn
6.打开数据库
startup force;
alter database open resetlogs;
```

