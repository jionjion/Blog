---
title: Oralce基础-异常篇
abbrlink: 7604d4a6
date: 2017-09-23 09:44:27
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---

## 内容简介
对Oracle数据库中视图的使用进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。

### 常见异常

| 异常                | 说明                               |
| ------------------- | ---------------------------------- |
| No_data_found       | 没有找到数据                       |
| Too_many_rows       | select...into...语句匹配了多条语句 |
| Zero_Divide         | 被零除                             |
| Value_error         | 算数异常或类型转换异常             |
| Timeout_on_resource | 在等待资源时发生超时异常           |
| INVALID_CURSOR      | 非法游标操作                       |
| DUP_VAL_ON_INDEX    | 违反唯一性约束                     |


### 系统内置异常
 - `No_data_found`异常

``` sql
declare
  pename emp.ename%type;
begin
  --查询一个不存在的员工
  select ename into pename from emp where empno=123; 
exception
  when no_data_found then dbms_output.put_line('没有找到用户');                       --when后可以看做一个大括号,可以跟多条语句,中间用分号分隔
                         dbms_output.put_line('请检查');                              
  when others then dbms_output.put_line('捕获其他意外');                              --将其他可能的意外都进行捕获处理.
end;
```

 - `Too_many_rows`异常

``` sql
declare
  pename varchar2(20);
begin
  --查询多条记录,赋值给一个变量,由于参数个数不匹配而抛出异常
  select ename into pename from emp where deptno = 10;
exception
  when too_many_rows then dbms_output.put_line('查询出多条记录');
                         dbms_output.put_line('无法具体匹配');
  when others then dbms_output.put_line('捕获其他意外');
end;
```

 - `zero_divide`异常

``` sql
-- 异常
declare
  pnumber number;
begin
  pnumber := 1/0;
exception
  when zero_divide then dbms_output.put_line('零不能做除数');
  when others then dbms_output.put_line('捕获其他意外');
end;
```

 - `value_error`数值转化异常

``` sql
declare
  pnumber number;
begin
  pnumber := 'abc';                                                                   --将字符串的赋值给数值变量
exception
  when value_error then dbms_output.put_line('算数或类型转换出错');
  when others then dbms_output.put_line('捕获其他意外');
end;
```

### 自定义异常

``` sql
-------------自定义异常
--查询50号部门的员工姓名
declare
  cursor cemp is select ename from emp where deptno=50;                               --定义游标为结果集
  pename emp.ename%type;
  no_emp_found exception;                                                             --自定义的一个异常
begin
  open cemp;
  fetch cemp into pename;                                                             --直接取得一个员工的姓名
  if cemp%notfound then
    raise no_emp_found;                                                               --抛出自定义异常
  end if;
  close cemp;                                                                          --oracle自动关闭因意外终止的进程所占资源,自动启动pmom(process monitor 进程监视器)
exception
  when no_emp_found then dbms_output.put_line('没有找到员工');
  when others then dbms_output.put_line('捕获其他意外');
end;
```
