---
title: Oralce基础-存储过程篇
abbrlink: 61c0c6c7
date: 2017-09-23 09:44:39
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---
## 内容简介
对Oracle数据库中的存储过程进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。

## 基础语法
语法

``` sql
create or replace procedure 名称(参数列表)
is|as PL/SQL子程序题

```

## 存储过程维护

``` sql
-- 查询存储过程
select * from user_procedures;
-- 查询所有对象
select * from user_objects;
-- 查询依赖,比如程序依赖表
select * from user_dependencies;
-- 重新编译
alter procedure get_emp_dept_name compile;
-- 查询子程序代码
select * from user_source;
-- 删除过程
drop procedure 过程名称;
-- 删除函数
drop function 函数名称;
```

权限相关维护
```sql
-- 授权,只有执行权限
grant execute on 存储过程 to 用户|角色| public;
```

## 无参的存储过程

``` sql
create or replace procedure say_HelloWorld
--说明部分(不能省略)
as
begin
  dbms_output.put_line('hello world');
end;
```


## 有参的存储过程

``` sql
-----------带参数的存储过程
--为指定工号的员工的涨100元,并打印前后的薪水
create or replace procedure raise_salary(eno in number)
as
  p_sal emp.sal%type;                                                                     --定义一个变量保存之前的薪水
begin
  select sal into p_sal from emp where empno = eno;
  update emp set sal=sal+100 where empno=eno;                                             --注意,这里一般不提交数据库
  dbms_output.put_line('涨前'||p_sal||'涨后'||(p_sal+100));
end;
```

``` sql
-----------无参的存储函数
--查询某个员工的年收入
create or replace function query_year_salary(p_empno in number) return number
as
 p_sal emp.sal%type;
 p_comm emp.comm%type;
begin
  select sal,comm into p_sal,p_comm from emp where empno=p_empno;
  return p_sal*12+nvl(p_comm,0);                                                            --滤空,在有空的表达式中,默认结果为空,因此需要滤空函数将其替换为0
end;
```

``` sql
--------------out参数 (如果有一个返回值就用存储函数,如果有多个返回值就用存储过程)
--查询某个员工的姓名,月薪,职位
-- 调用时传入参入进入,入参有值,出参自动返回值
create or replace procedure query_info(p_empno in number,p_ename out varchar2,p_sal out number,p_job out varchar2)  
as
begin
  select ename,sal,job into p_ename,p_sal,p_job from emp where empno= p_ename;
end;
```


## 调用过程
 - 调用  `setserveroutput on` 在命令窗口中执行，打开输出控制。
 - 命令窗口中 `execute sayhelloworld()` 或`exec sayhelloworld()`。
 - 在另外一个程序的begin-end中调用  `sayHelloWorld()`。
``` sql
begin
  say_helloworld();                                                                       -- 存储过程只能在begin-end语句块中调用                                                               
  say_HelloWorld();
end;
```

## 嵌套子程序
将存储过程或者函数作为参数在定义中创建
如果子程序间发生了相互引用,可以通过前导声明解决!
``` sql
-- 前导声明
declare
  procedure pro_b ;  -- 前导声明
  procedure pro_a as 
  begin
    dbms_output.put_line('我是A过程');
    pro_b;        -- 相互依赖
  end;
  
  procedure pro_b as 
  begin
    dbms_output.put_line('我是B过程');
    pro_a;        -- 相互依赖
  end;
begin
  pro_a;
  pro_b;
end;
```

## 具有迁移能力的过程
在数据库中,如果我们编写某些具有通用功能的存储过程,为了在迁移后将该存储过程所操做的表变为新数据库空间下的资源对象,我们需要编写一些具有迁移能力的存过
这里使用`authid current_user as`语法表示,该存储过程所用到的对象为当前数据库所有的对象,并不是来源数据库或者用户下的对象.
```sql
-- 具有迁移能力的过程
create or replace procedure auth_curr_func 
  authid current_user as  -- 操控的dept表为当前用户下的dept表,便于迁移对象
begin
  insert into dept(deptno,dname,loc)
         values ('10','ACCOUNTING','NEW YORK');
  commit;    -- 提交自治事务,当然也可以回滚
end;
```

## nocopy选项
在默认情况下,PL/SQL程序之中会对`IN`模式传递的参数采用引用传递方式,其性能较高
对于`OUT`或者`IN OUT`模式采用数值传递,在传递时将实参数据拷贝一份给形参,这样主要是方便形参
对数据的操作,在过程结束之后,被赋予`OUT`或 `IN OUT`形参赏的值会赋予对应的实参
由于`OUT`和`IN OUT`将会对操作的数据进行复制,所以当传递数据较大时(例如集合,记录),该过程会变长,
消耗大量的内存空间,为了防止这种情况的出现,在定义过程参数时,可以使用`nocopy`选项,将`OUT`或者`IN OUT`的值变为引用传递
`参数名称 [参数模式] nocopy 数据类型;`
当抛出异常时,nocopy修饰的参数可以不会被回滚,停留在抛出异常前的状态.

## 自治事务
子程序的事务并不受主程序事务影响,切不影响主事务.
常用于日志记录
```sql
create or replace procedure auto_tran_func as
  pragma autonomous_transaction;  -- 自治事务
begin
  insert into dept(deptno,dname,loc)
         values ('10','ACCOUNTING','NEW YORK');
  commit;    -- 提交自治事务,当然也可以回滚
end;
```