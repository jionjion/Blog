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

## 触发器
在执行DML和DDL语句时,进行触发

* 模式DDL触发器
* 数据库级触发器
* DML触发器, 在数据增、删、改时进行执行，通过对前后数据校验，完成数据录入。
  * 行级触发器
  * 语句级触发器
  * instead of 触发器  , 多用来修改视图

### 相关查询

```sql
-- 查看触发器
select * from user_triggers;
select * from all_triggers;
select * from db_triggers;

-- 禁用触发器
alter trigger emp_t disable;
-- 启用触发器
alter trigger emp_t enable;

-- 删除触发器
drop trigger emp_t;
```



### 语法

```sql
create [or replace] trigger trigger_name                 -- 创建或者取代,触发器名
after | before | instead of                              -- 之后 | 之前 | 取代 时触发
[insert] [[or] update [of column_list]] [[or] delete]    -- 引发的动作
on table_or_view_name                                    -- 表|视图对象
[referencing {old [as] old / new [as] new}]              -- 触发前后的行记录效果
[for each row]                                           -- 行级|表级触发器
[when (condition)]                                       -- 条件
pl/sql_block;                                            -- 语句块
```



### 表级触发器

`:old` 和 `:new` 代表一行记录在操作前后的值

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



### 行级触发器

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



## 模式触发器

### 常见的系统变量
| 变量                  | 说明                    |
| --------------------- | ----------------------- |
| ora_client_ip_address | 客户端地址              |
| ora_database_name     | 当前数据库名            |
| ora_login_user        | 当前用户名              |
| ora_dict_obj_name     | 当前ddl操作的对象名     |
| ora_dict_obj_owner    | 当前ddl操作的对象所有者 |
| ora_dict_obj_type     | 当前ddl操作的对象类型   |

### 模式触发器

```sql
create or replace trigger log_t
after drop on schema
begin
  dbms_output.put_line('对象删除了' || ora_dict_obj_name || ora_dict_obj_type || ora_dict_obj_owner);
end ;
```



### 数据库启动/关闭触发器

```sql
create or replace trigger startup_t 
after startup on database      -- startup开启
begin
  dbms_output.put_line('数据库关闭!');
end ;

create or replace trigger startup_t 
before shutdown on database      -- shutdown关闭 
begin
  dbms_output.put_line('数据库关闭!');
end ;
```



### 用户登录/退出 触发器

```sql
create or replace trigger logon_t
after logon on database
begin
  dbms_output(ora_login_user || '登录了');
end ;
create or replace trigger logoff_t
before logoff on database
begin
  dbms_output(ora_login_user || '登录了');
end ;
```

