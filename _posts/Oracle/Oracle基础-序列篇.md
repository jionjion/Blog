---
title: Oracle基础-序列篇
typora-root-url: ../
date: 2022-07-24 11:43:08
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

## 序列
生成唯一的,连续序号的对象.
序列可以是升序或者降序的

```sql
-- 创建序列
create sequence emp_s
start with 1        -- 开始
increment by 1      -- 步长
maxvalue 2000       -- 最大值
minvalue 1          -- 最小值
nocycle             -- 不循环
cache 10           -- 缓存,默认20

-- 查看用户序列
select * from user_sequences;

-- 序列下一个值
select emp_s.nextval from dual;

-- 序列当前值
select emp_s.currval from dual;

-- 修改序列,不能修改start with,其他均可
alter sequence emp_s increment by 1 maxvalue 3000 minvalue 0 cycle cache 20;

-- 删除序列
drop sequence emp_s;
```