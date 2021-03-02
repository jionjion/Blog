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
### 使用闪回恢复表内容到指定时间点
后面的参数为要还原的时间点
``` sql
flashback table dept to timestamp to_timestamp('2017-09-18 00:00:00','yyyy-mm-dd hh24:mi:ss');
```

### 使用快照查询某时间点的数据
查询过去100分钟前的数据

``` sql
select * from dept AS OF TIMESTAMP  (SYSTIMESTAMP - INTERVAL '100' MINUTE)     
```

查询某个点的数据

```sql
select * from dept as of timestamp to_timestamp('2017-09-18 00:00:00','YYYY-MM-DD HH24:MI:SS');
```

### 使用CSN恢复
SCN提供了Oracle的内部时钟机制，可被看作逻辑时钟，在事物提交时，它被赋予一个唯一的标示事物的SCN 。

将删除时间转换为scn ,注意时间不能早于数据库安装时间

``` sql
select timestamp_to_scn(to_timestamp('2017-09-18 10:00:00','YYYY-MM-DD HH:MI:SS')) from dual; 
```

根据CSN时钟查询数据

``` sql
select * from dept AS  OF SCN 2269794;
```

### 查询回收站中删除的表

``` sql
select object_name,original_name,partition_name,type,ts_name,createtime,droptime from recyclebin;
```

### 关闭闪回

``` sql
alter table dept disable row movement
```

### 闪回表

```sql
flashback table dept to before drop ;
flashback table dept to before drop rename to new_dept;
```

