---
title: Oracle基础-动态SQL篇
abbrlink: 8114df6e
date: 2017-09-23 12:54:24
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---

-----

## 内容简介
介绍了Oracle数据库中，在存储过程或者匿名语句块中动态执行SQL语句。


## 语法

`into` 保存SQL的执行结果,返回多个则使用bulk collect设置保存变量
`using` 为动态SQL的占位符设置内容,默认模式为IN模式
`returning`|`return` 使用效果相同,获得被影响的行数.通过bulk collect实现批量绑定,默认为OUT模式
**注意** 如果在动态SQL中执行了`DDL`语句,会将当前会话提交事务!
``` sql
execute immediate 动态SQL字符串  [ [bulk collect] into 自定义变量,.. | 记录类型 ]
[using [in | out | in out] 绑定参数,... ]
[[returning | return] [bulk collect] into 绑定参数];
```

## 使用

### 动态SQL返回值
使用`into`关键字获得SQL执行的返回结果字段,注意类型匹配
``` sql
-- 具有返回值的SQL
declare
  v_count         number;
  v_sql           varchar2(2000);                 -- SQL脚本
begin
  v_sql := 'select count(*) ' || ' from emp';
  -- 执行,并返回数据
  execute immediate v_sql into v_count;
  dbms_output.put_line('数据总量' || v_count);
end;
```

### 动态SQL占位符
使用冒号加占位符,如`:1`进行占位,并在随后使用`using`关键字替换占位符,注意,占位符的出现顺序与替换顺序保持一致.
``` sql
-- 具有占位符替换的SQL
declare
  v_count         number;
  v_sql           varchar2(2000);                 -- SQL脚本
  v_deptno        emp.deptno%type;
begin
  -- 变量赋值
  v_deptno := '10';
  -- 具有占位符的SQL,使用冒号: + 形参的方式进行变量占位,这里使用:2实现占位
  v_sql := 'select count(*) from emp where deptno = :p_deptno and rownum <= :2' ; 
  -- 执行,并返回数据
  execute immediate v_sql into v_count  -- 具有返回值的
  using v_deptno , 5;  -- 根据参数列表位置,使用变量或者常量完成赋值
  dbms_output.put_line('数据总量' || v_count);
end;
```

### 动态SQL修改数据

#### 嵌套表进行批量更新
使用动态SQL修改部门名称为中文
``` sql
declare
  v_sql          varchar2(2000);
  type deptno_nested is table of dept.deptno%type not null;
  type dname_nested  is table of dept.dname%type  not null;
  t_deptno       deptno_nested   := deptno_nested(10,20,30,40);        
  t_dname        dname_nested    := dname_nested('财务','研发','销售','生产'); 
  v_deptno       dept.deptno%type;
  v_dname        dept.dname%type;
begin
  -- 动态SQL,使用占位符,并具有返回结果
  v_sql := 'update dept set dname = :1
             where deptno = :2
            return dname into :3 ';
             
  -- 循环
  for i in t_deptno.first .. t_deptno.last loop
    execute immediate v_sql 
    using t_dname(i) , t_deptno(i)
    return into v_dname;  -- 返回影响的字段
    dbms_output.put_line( '修改为部门' || v_dname);
  end loop;
end;
```

#### 获得修改前的的数据
在动态SQL中使用`returning 字段 into :占位符`方式将修改前的字段信息返回,随后执行动态SQL,并使用`returning into 变量`形式获得修改前的字段信息
``` sql
-- 获得删除前的数据
declare
  v_sql          varchar2(2000);
  v_dname        dept.dname%type;
begin
  v_sql := 'delete from dept where rownum <= 1 
            returning dname into :1';
  execute immediate v_sql
  returning into v_dname;
  dbms_output.put_line('删除部门' || v_dname);
end;
```

### 动态SQL批量绑定

#### 批量绑定,设定输入参数

``` sql
-- 批量绑定,设定输入参数
declare
  -- 嵌套表
  type empno_nested is table of emp.empno%type;
  -- 索引表
  type ename_index is table of emp.ename%type index by pls_integer;
  type job_index   is table of emp.job%type   index by pls_integer;
  type sal_index   is table of emp.sal%type   index by pls_integer;
  t_ename          ename_index;
  t_job            job_index;
  t_sal            sal_index;  
  v_sql            varchar2(2000);
  t_empno          empno_nested := empno_nested(7369,7566,7788); -- 要删除的员工编号
begin
  -- SQL,返回删除前的数据
  v_sql := 'delete from emp
             where empno = :eno 
         returning ename, job, sal into :ea, :ej, :es';
  -- 使用forall批量执行
  forall i in 1 .. t_empno.count
    execute immediate v_sql 
      using t_empno(i)
  returning bulk collect into t_ename,t_job, t_sal;
  
  -- 循环获得修改后的数据对象
  for i in 1 .. t_ename.count loop
    dbms_output.put_line('员工' || t_ename(i) || '职位' || t_job(i) || '删除前工资' || t_sal(i));
  end loop;
end;
```

#### 批量绑定,获得返回结果

``` sql
-- 批量绑定,获得返回结果
declare
  -- 索引表
  type ename_index is table of emp.ename%type index by pls_integer;
  type job_index is table of emp.job%type index by pls_integer;
  type sal_index is table of emp.sal%type index by pls_integer;
  t_ename        ename_index;
  t_job          job_index;
  t_sal          sal_index;
  v_sql          varchar2(2000);
begin
  -- SQL,返回修改后的数据
  v_sql := 'update emp set sal = sal + 1
             where deptno = ''10''
            return ename,job,sal into :1,:2,:3';
  -- 执行
  execute immediate v_sql
  return bulk collect into t_ename , t_job , t_sal;
  -- 循环获得修改后的数据对象
  for i in 1 .. t_ename.count loop
    dbms_output.put_line('员工' || t_ename(i) || '职位' || t_job(i) || '修改后工资' || t_sal(i));
  end loop;
end;
```

###  在打开游标时使用动态变量

语法

``` sql
  open 游标名称  for 动态SQL语句 [using 绑定变量,绑定变量,...]
```

#### 动态SQL操控游标

``` sql
-- 动态SQL操控游标
declare
  cur_emp    sys_refcursor;     -- 弱类型的游标
  r_emp      emp%rowtype;
  v_deptno   emp.deptno%type;   -- 查询部门编号
begin
  v_deptno := 10;
  -- 动态SQL打开游标
  open cur_emp for 'select * from emp where deptno = :dno'
  using v_deptno;
  -- 使用loop循环遍历游标数据
  loop
    fetch cur_emp into r_emp;
    exit when cur_emp%notfound;
    dbms_output.put_line('员工' || r_emp.ename || '的薪资为' || r_emp.sal);
  end loop;
  -- 关闭游标
  close cur_emp;
end;
```

#### 动态SQL操控批量集合

``` sql
-- 动态SQL操控批量集合
declare
  cur_emp    sys_refcursor;     -- 弱类型的游标
  type emp_index is table of emp%rowtype index by pls_integer;  -- 索引表
  t_emp      emp_index;         -- 保存数据
  v_deptno   emp.deptno%type;   -- 查询部门编号
begin
  v_deptno := 10;
  -- 动态SQL打开游标
  open cur_emp for 'select * from emp where deptno = :dno'
  using v_deptno;
  -- 批量读取数据
  fetch cur_emp bulk collect into t_emp;
  -- 关闭游标
  
  -- 使用for循环获得集合的内容
  for i in t_emp.first .. t_emp.last loop
    dbms_output.put_line('员工' || t_emp(i).ename || '的薪资为' || t_emp(i).sal);
  end loop;
end;
```