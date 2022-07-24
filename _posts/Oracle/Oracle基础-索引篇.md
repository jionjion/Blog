---
title: Oracle基础-索引篇
typora-root-url: ../
date: 2022-07-24 12:13:06
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

## 索引

索引在逻辑和物理上都独立于表的数据
oracle自动维护索引: 当对表进行增删改时,会将索引自动维护

## 索引分类

- 平衡树索引,倒立的树形结构(根节点块->分支节点块->叶子节点块->表的数据行)
              通过根据数据列的值范围进行分块,层层指向,直到表的数据行
- 树的平衡 随着数据的删除,更改,某些索引块指向会减少,产生亏空,降低查询效率.因此,经常进行索引碎片.

### 普通索引
```sql
-- 创建
create index emp_no_idx on emp(eno);  -- 一般选择具有唯一性的字段作为索引创建字段

-- 查询
select * from user_indexes;       -- 用户创建的索引
select * from user_ind_columns;   -- 索引涉及到的索引
-- 分析索引
analyze index emp_no_idx validate structure;
select * from index_stats ; -- pct_used 值过低,说明索引碎片过多,最大90%

-- 重建索引
alter index emp_no_idx rebuild;    

-- 删除索引
drop index emp_no_idx; 
```

### 唯一索引
```sql
-- 创建
create unique emp_no_idx  on emp(eno 才才 );  -- 只能插入空值,否则不能重复
```

### 组合索引
```sql
-- 创建
create index emp_no_name_idx on emp(eno , ename);    -- 以组合作为索引依据
```

### 反向键索引
避免某些区间内的B树条目不平衡,因此对数值进行反转.  101,102,103,104 -> 反转 -> 401,301,201,101 ,适合某些字段在开头不变的
```sql
-- 创建
create index emp_no_idx on emp(eno) reverse;
-- 修改为正向的普通索引
alter index emp_no_idx rebuild noreverse;
```

### 函数索引,对默写字段经常进行函数过滤,而创建的索引

```sql
-- 创建
create index emp_no_idx on emp(upper(ename)); -- 这样,在进行大写字段查询时,效率会更高
```

### 位图索引 
使用0101进行存放,而不是指针,适用于低基序列(如'男','女'选项,指取值非常少的),减少空间占用,适合放在数据仓库中
```sql
-- 创建
create bitmap index emp_no_idx on emp(sax);
```

### 重建索引

`online` 在重建索引过程中,用户可以对原来的索引进行修改,避免锁表

`nologging` 在创建中生成最少重做条目,加快重做时间

`compute statistics` 重建过程中,生成了数据库所需要的统计信息,避免重复分析

``` sql
alter index index_name rebuild [online] [nologging] [compute statistics] 
```

### 索引分区
 局部分区索引,在分区表上创建索引,索引分区范围与表分区范围一致
 ```sql
 -- 创建
 create index emp_no_idx on emp(eno) local;
 -- 查看
 select * from user_indexes;         -- parririoned 字段表示是否分区
 select * from user_ind_partitions;  -- 分区的索引信息
 ```
### 全局分区索引
在分区表或者非分区表上创建的索引,索引单独指向分区范围,与表是否分区无关
```sql
-- 创建
create index emp_no_idx on emp(eno) global partition by range(eno)
( partition emo_no_idx1 values less than (2000) ,
 partition emo_no_idx1 values less than (maxvalue)
)
```

### 全局非分区索引

在分区表上创建的全局普通索引,索引没有被分区

```sql
-- 创建
create index emp_no_idx on emp(eno) global; 
```

