---
title: Oracle基础-自定义数据类型篇
abbrlink: b8124b40
date: 2017-09-23 12:54:54
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---
----

## 内容简介
对oracle数据库中基本数据类型进行扩展，实现自定义数据类型，封装对象多属性。


## 对象类型

``` sql
-- 自定义对象类型,
create type employee_object as object(                                                          -- 使用as关键字
ename varchar2(20),                                                                             -- 自定义的对象的属性
empno number
);

```

## 记录类型
- 单条记录类型

``` sql
-- 自定义记录类型
declare 
  type  employee_record  is record(                                                             -- 使用is关键字
    ename varchar2(20) ,                                                                        -- 自定义的记录字段
    empno number(8));  
  emp_01 employee_record;                                                                       -- 创建一条记录
begin
  emp_01.ename := 'Jion';                                                                       -- 为记录字段赋值
  emp_01.empno := '1230';
  dbms_output.put_line(emp_01.ename||'员工的工号是'||emp_01.empno);                              -- 调用记录
end;
```

- 多条记录类型，录表(可以充当List集合)
``` sql
declare
  type employee_record  is record(                                                             -- 使用is关键字
  ename varchar2(20) ,                                                                         -- 自定义的记录字段
  empno number(8));  
  
  type employee_table is table of employee_record indexby BINARY_integer;                      -- 自定义表,表结构与记录一致
  emp_01 employee_record;                                                                      -- 创建一条记录
  table_01   employee_table;                                                                   -- 创建一个自定义表
begin
  emp_01.ename := 'Jion';                                                                       -- 为记录字段赋值
  emp_01.empno := '1230';
  
  dbms_output.put_line('First index:'||'   '|| mytab(1) ||'   '); 
end;
--
declare
     type t_studenttable is table of students%rowtype indexby binary_integer; 
     v_students t_studenttable;
begin
select * into v_students(1100)
from students 
where id=1100; 
     dbms_output.put_line( v_students(1100).ouusrnm); 
end;
```


