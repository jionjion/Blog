---
title: Oralce基础-游标篇
abbrlink: b99c5a9f
date: 2017-09-23 09:44:01
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---

> Oracle 游标的使用

<!--more-->



## 内容简介

对Oracle数据库中游标的使用进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。

## 基础语法
游标大体可以分为:
- 静态游标:结果集已经存在
   隐式游标:所有DML语句为隐式游标,通过隐式游标属性可以获取SQL语句信息
   显示游标:用户显式声明游标,指定结果集.当查询返回多行时,需要显示游标
- REF游标: 动态关联结果集的临时对象

**语法:**
``` sql
cursor 游标名称([参数列表]) [return 返回类型] is
子查询
[for update [of 数据列,数据列,] [nowait]]
```

**示例:**
``` sql
--查询员工姓名的集合并存储为游标形式,游标关闭时默认销毁
cursor c_emp is select ename from emp
```
`open c_emp`打开游标,指针指向第一个
`close c_emp`关闭游标,关闭后失效 
`fetch c_emp into pename`取游标中一行的元素到变量中,同时指针下移动

### 显式游标
####  使用游标循环查询结果集

``` sql
--查询并打印员工姓名和薪水
declare
  --定义一个游标
  cursor c_emp is select ename,sal from emp;
  --为游标定义变量
  pename emp.ename % type;                                                 --变量类型为表中字段的类型
  psal   emp.sal %type;                                                    --对空格不敏感
begin
  --打开游标
  open c_emp;
      loop
          --取第一条记录
          fetch c_emp into pename,psal;                                    --注意,承接的变量和查询出的变量数量,类型相对应
          --退出条件
          exit when c_emp%notfound;
          --打印
          dbms_output.put_line(pename||'的薪水是'||psal);
      end loop;
  --关闭游标
  close c_emp;
end;
```

#### for循环读取游标
``` sql
declare
  cursor cur_emp return emp%rowtype is
    select * from emp;
begin
  for r_emp in cur_emp loop
    dbms_output.put_line( r_emp.ename || '的薪水为:' || r_emp.sal);
  end loop;
end;
```

#### 将游标绑定到索引表

``` sql
declare
  cursor cur_emp return emp%rowtype is 
    select * from emp;
  type emp_index is table of emp%rowtype index by pls_integer;
  t_emp    emp_index;
begin
  -- 循环初始化
  for r_emp in cur_emp loop
    t_emp(r_emp.empno) := r_emp;
  end loop;
  -- 使用
  dbms_output.put_line('员工7369'|| '姓名' || t_emp(7369).ename || '薪资' || t_emp(7369).sal );
end;
```

#### 使用游标进行记录操作
``` sql
--定义员工涨薪水:boos+3;经理+2;其他+1
declare
  --定义游标代表给那些员工涨工资
  cursor c_emp is select empno,job from emp;
  pempno emp.empno%type;
  pjob   emp.job%type;
begin
  --打开游标
  open c_emp;
    loop
      --取出一个员工
      fetch c_emp into pempno,pjob;
      --退出条件
      exit when c_emp%notfound; 
      --判断条件
      if pjob =  'PRESIDENT' then update emp set sal=sal+3 where empno = pempno;
      elsif pjob = 'MANAGER' then update emp set sal=sal+1 where empno = pempno;   -- 判断职位,并涨工资
      else update emp set sal=sal+1 where empno = pempno;
      end if;
    end loop;
  close c_emp;
  --提交数据,oracle数据库的默认事物隔离级别是 read commit.
  --事务的ACID  原子性,一致性,隔离性,持久性
  commit;
  dbms_output.put_line('修改完成');
end;
```
#### 游标遍历更新

``` sql
-- 游标遍历更新
declare
  cursor cur_emp return emp%rowtype is
    select * from emp
  for update of sal;  -- 在需要更新的字段后,使用for update of 字段,字段
begin
  -- 更新字段
  for r_emp in cur_emp loop
    update emp set sal = sal + 1
     where current of cur_emp;
  end loop;
end;

-- 定义弱类型的游标
declare
  type cursor_ref is ref cursor;                -- 定义弱类型游标
  c_cur              cursor_ref;                -- 定义游标变量
  r_emp              emp%rowtype;
  r_dept             dept%rowtype;
begin
  -- 打开游标,并决定游标类型
  open c_cur for select * from emp;
  loop
    -- 获取游标数据
    fetch c_cur into r_emp;
    -- 退出条件
    exit when c_cur%notfound;
    -- 输出数据
    dbms_output.put_line('员工' || r_emp.ename || '的薪资为' || r_emp.sal);
  end loop;
  -- 结束循环,关闭游标
  close c_cur;
  
  -- 可以重复使用
  open c_cur for select * from dept;
  loop
    -- 获取游标数据
    fetch c_cur into r_dept;
    -- 退出条件
    exit when c_cur%notfound;
    -- 输出数据
    dbms_output.put_line('部门' || r_dept.dname || '的位置为' || r_dept.loc);
  end loop;
  -- 结束循环,关闭游标
  close c_cur;  
end;
```
#### sys_refcursor游标
使用 sys_refcursor 代替游标变量的声明
``` sql
declare
  c_cur              sys_refcursor;                -- 定义弱类型游标变量
  r_emp              emp%rowtype;
  r_dept             dept%rowtype;
begin
  -- 打开游标,并决定游标类型
  open c_cur for select * from emp;
  loop
    -- 获取游标数据
    fetch c_cur into r_emp;
    -- 退出条件
    exit when c_cur%notfound;
    -- 输出数据
    dbms_output.put_line('员工' || r_emp.ename || '的薪资为' || r_emp.sal);
  end loop;
  -- 结束循环,关闭游标
  close c_cur;
  
  -- 可以重复使用
  open c_cur for select * from dept;
  loop
    -- 获取游标数据
    fetch c_cur into r_dept;
    -- 退出条件
    exit when c_cur%notfound;
    -- 输出数据
    dbms_output.put_line('部门' || r_dept.dname || '的位置为' || r_dept.loc);
  end loop;
  -- 结束循环,关闭游标
  close c_cur;  
end;
```

#### 游标属性

| 属性  | 解释|
| --------- | ---------------- |
| %isopen   | 判断游标是否打开 |
| %rowcount | 读取过的行数值   |
| %notfound | 没有找到返回真   |
| %found    | 找到返回真       |
|游标的限制.同一个会话中只能打开300个游标|

``` sql
declare
  cursor c_emp is select empno,job from emp;
  pempno emp.empno%type;
  pjob   emp.job%type;
begin
  --打开游标
  open c_emp;
  --游标状态 
  if c_emp %isopen then
    dbms_output.put_line('游标打开');
  else
    dbms_output.put_line('游标没有打开');
  end if;
    loop
      --取出一个员工
      fetch c_emp into pempno,pjob;
      --打印游标指针,默认从1开始
      dbms_output.put_line('游标所在行号为'||c_emp%rowcount);
      --退出条件
      exit when c_emp%notfound;
    end loop;
  close c_emp;
  commit;
  dbms_output.put_line('修改完成');
end;
```

#### 带参数的游标

``` sql
--查询某个部门的员工姓名(带参数的游标)
declare
  cursor c_emp(dno number) is select ename from emp where deptno=dno;              -- 声明一个带参数的游标,并写出SQL查询语句    
  p_ename emp.ename%type;
begin
  --打开时传递参数
  open c_emp(10);
  loop
    fetch c_emp into p_ename;
    dbms_output.put_line(p_ename);                                                 --打印
    exit when c_emp%notfound;
  end loop;
  close c_emp;
end;
```

### 隐式游标

#### 指定次数循环
``` sql
-- 显示工资最高的前3名雇员的名称和工资
declare                                                                            -- 声明变量
    v_ename varchar2(10);
    v_sal   number(5);
    cursor c_emp is select ename, sal from emp order by sal desc;             -- 根据工资,降序排列
begin
    open c_emp;                                                               -- 打开游标
    for i in 1 .. 3 loop                                                           -- for循环控制,只循环前3次
        fetch c_emp into v_ename, v_sal;                                      -- 抓取数据到变量中
        dbms_output.put_line(v_ename || ',' || v_sal);                             
    end loop;
    close c_emp;                                                              -- 关闭游标
end;
```
#### 全部循环

``` sql
-- 隐式游标,使用for循环结迭代,当记录数量过多时,将会内存溢出
declare
    cursor c_emp is select ename, sal from emp where rownum < 3;
begin
    for r_emp in c_emp loop
      dbms_output.put_line(r_emp.ename || ',' || r_emp.sal);
    end loop;
end;
```

#### 匿名游标

``` sql
-- 隐式游标,使用for循环迭代,当记录过多时,会内存溢出
begin  
 for r_emp in (select ename from emp where rownum < 10)  loop  
  dbms_output.put_line(r_emp.ename);  
 end loop;  
end;
```


