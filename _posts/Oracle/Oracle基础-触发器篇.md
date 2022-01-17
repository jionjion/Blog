---
title: Oracle基础-触发器篇
abbrlink: 87e21b59
date: 2017-09-23 12:54:13
categories:
  - Oracle
  - Trigger
tags: [Oracle, Trigger]
---
> Oracle 触发器的使用

<!--more-->



## 内容简介

介绍了Oracle中触发器的简单使用，实际中触发器的引入回导致数据难以控制，所以还是少用为好。

## 使用
触发器是在数据增、删、改时进行执行，通过对前后数据校验，完成数据录入。
按照作用范围可以分为行级触发器和表级触发器
`:old` 和 `:new` 代表一行记录在操作前后的值
- 表级触发器
``` sql
-- 禁止在非工作时间插入新员工
create or replace trigger new_emp
before insert																	--触发条件
on emp
declare                                                                                       --在没有声明时可以省略
begin
  if to_char(sysdate,'day')  in ('星期六','星期日') or                                         --可以直接调用系统函数的结果                
      to_number(to_char(sysdate,'HH24'))  not between 9 and 17 then                           --逻辑条件
     raise_application_error(-20001,'禁止在非工作时间插入新员工');                   	         --自定义一个异常,指明编号[-20001,-40000]和提示
  end if;                                                                                     --结束判断
end;
```

- 行级触发器

``` sql
-- 涨工资,不能少,只能多
create or replace trigger cherk_sal
before update																--触发条件
on emp
for each row                                                                                  -- 行级触发器
declare
begin
  if :new.sal < :old.sal then                                                                 -- 新值小于旧值
    raise_application_error(-20002,'涨后薪水不能小于之前的,涨前的:'||:old.sal||'涨后的'||:new.sal);
  end if;
end;
```
