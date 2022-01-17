---
title: Oracle基础-程序包篇
abbrlink: dcb92d15
date: 2017-09-23 12:53:44
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---
> Oracle 程序包的使用

<!--more-->



## 内容简介

Oracle数据库中，程序包是一系列相关存储过程和函数的集合，通过程序包可以简化语句块的编写，便于对各种过程、函数进行分类。

## 构成
### 包头
声明程序包中各种函数或者存储过程，只有在包头中声明过的才可以通过包直接对外部提供。
包头中声明的函数或存储过程必须在包体中实现，并且参数类型和返回值类型必须一致。

可以使用`constant 常量名`声明常量
``` sql
create or replace package myPackage as
  type myCursor is ref cursor;                                                                -- 自定义数据类型,类似于游标性质
  procedure queryEmpList(dno in number,empList out myCursor);                                 -- 声明存储过程
end myPackage
```

### 包体
对包头中声明的每一个方法进行实现。

``` sql
create or replace package body myPackage as                                                   -- 声明包体
  procedure queryEmpList(dno in number,empList out myCursor) as                               -- 声明存储过程的实现
  begin
    open empList for select * from emp where deptno = dno;                                    -- select语句查询的结果由游标表示  打开游标.for表示代指的结果集
  end queryEmpList;                                                                           -- 表示存储过程的结束
end myPackage;
```

### 维护

``` sql
-- 查看数据类型
select * from user_objects;
-- 查看源代码
select * from user_source;
-- 删除包
drop package 包名;
```

**重新编译**
编译形式
- `package`:重新编译包头和包体
- `specification`:重新编译包头
- `body`:重新编译包体
```sql
alter package 包名 complete [debug] package | specification | body [reuse settings];
```
示例:
```sql
alter package emp_pkg package;
```

### 相关概念
**包的作用域**
在包体中定义的变量为私有变量.
默认情况下,所有的包在第一次调用时才会初始化,随后包的运行状态保存在用户的全局会话之中,在会话期间,该包会被当前用户持续占用,直到会话结束.
因此包中的任何一个变量或者游标可以在一个会话期间一直存在,相当于全局变量,同时可以被所有子程序共享.
如果不希望设置全局变量,可以使用 `serially_reusable` 选项,但是这会导致每次调用包时都会重新加载-释放,高并发下用户性能很差.

``` sql
create or replace 包名
as
   pragma serially_reusable;  -- 不使用全局变量,每次调用重新加载,包头包体同样声明
   变量...
   方法...
end 包名;
```

**方法的重载**
相同方法名,不同参数列表的一类方法

**包的初始化**
在包头中可以定义复杂的变量类型,而在包体中使用  begin...end 直接进行初始化定义

**包的纯度级别**
如果在包中定义了函数,那么可以通过SQL进行调用,如果对包中函数进行限制,则可以使用包纯度限制
`pragma restrict_references(函数名,[WNDS] [,WNPS] [,RNDS] [,RNPS])`
其中
- `WNDS` 函数不能修改数据库表数据,无法使用DML更新
- `RNDS` 函数不能读数据库表,无法使用select语句
- `WNPS` 函数不予许修改包中变量
- `RNPS` 函数不允许读取包中变量

注意:如果用户定义的是公共SQL函数,则必须符合`WNDS`(不能DML),`WNPS`(不能写包变量),`RNPS`(不能读包变量)
满足这三个纯度要求的函数即为公共函数.


## 常见系统包
### dbms_output包
系统自带的输出包,将内容输出到日志或者服务器界面
`select * from all_source s where s.name = 'DBMS_OUTPUT';`
| 方法 | 说明 |
|----|----|
|`dbms_output.enable(buffer_size => )`|           启用缓冲区,指定缓冲区大小字节|
|`dbms_output.disable`|                           关闭缓冲区|
|`dbms_output.put(a => )`|                        将内容防止在缓冲中,不包括换行,等待输出|
|`dbms_output.put_line(a => )`|                   输出指定内容,包括换行|
|`dbms_output.new_line`|                          在输出末尾增加换行,并输出缓冲区|
|`dbms_output.get_line(line => , status => )`|    获得一行数据,status为取回结果,1表示获取到一行数据,0表示没有取到|
|`dbms_output.chararr`|											|
|`dbms_output.get_lines(lines => , numlines => )`|   lines将要取回的行的嵌套表,numlines实际取回的行数|


### dbms_job定时调度包
数据库端实现的定时调度任务
``` sql
dbms_job.any_instance                        
dbms_job.isubmit(job       => ,
                 what      => ,
                 next_date => ,
                 interval  => ,
                 no_parse  => )
dbms_job.submit(job       => ,                提交作业,作业号
                what      => ,                作业任务
                next_date => ,                开始日期
                interval  => ,                间隔,单位天
                no_parse  => ,                是否重复
                instance  => ,
                force     => )                 
dbms_job.remove(job => )                      传入作业号,删除job
dbms_job.change(job       => ,
                what      => ,
                next_date => ,
                interval  => ,
                instance  => ,
                force     => )                
dbms_job.what(job => , what => )                
dbms_job.next_date(job => , next_date => )
dbms_job.instance(job      => ,
                  instance => ,
                  force    => )
dbms_job.interval(job => , interval => )                  
dbms_job.broken(job       => ,
                broken    => ,
                next_date => )
dbms_job.run(job => , force => )                
dbms_job.user_export(job => , mycall => )
dbms_job.background_process
dbms_job.is_jobq

-- 查看任务JOB
select * from user_jobs;
```

### dbms_asser字符串工具包
处理字符,将一些敏感字符进行转换

```sql
-- 查询SQL查询对象是否存在
select dbms_assert.simple_sql_name('emp') from dual;

-- 验证对象名是否符合语法规则
select dbms_assert.qualified_sql_name('EMP_V') from dual;

-- 验证输入是否为有效输入的schema名称
select dbms_assert.schema_name('SCOTT') from dual;

-- 验证SQL查询对象是否存在
select dbms_assert.sql_object_name('EMP') from dual;

-- 为传入字符串增加双引号
select dbms_assert.enquote_name( 'ABC' ) from dual;

-- 传入字符串左右增加单引号
select dbms_assert.enquote_literal(123) from dual
```
