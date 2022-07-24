---
title: Oralce基础-存储过程篇
abbrlink: 61c0c6c7
date: 2017-09-23 09:44:39
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---
> Oracle 存储过程的使用

<!--more-->



## 内容简介

对Oracle数据库中的存储过程进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。

- `PL/SQL`  过程语言和结构化查询语言的结合
- `PL/SQL` 引擎   执行过程语句执行,随后交由SQL引擎执行过程
- 开启服务器输出 `set serverout on;`



## 数据常量
- 数字类型
  - binary_integer 整型
     - natural
     - naturaln
     - positive
     - positiven
     - signtype
  - number  整数,负数,浮点型
     - decimal
     - float
     - integer
     - real
  - pls_integer 有符号的整数
  - simple_interger 有符号整数
- 字符
  - char
  - varchar2
  - long
  - raw
  - long raw
- 布尔
   -    true
    -   false
     -  null
   - 日期
    -   date
     -  timestamp
- LOB类型: 大的非结构化文件,使用 DBMS_LOB程序包操纵LOB数据
- BFILE  大型二进制对象存在操作系统文件
- BLOB   大型二进制对象存在数据库中
- CLOB   大型字符数据储存在数据库中
- NCLOB  大型UNICODE字符数据
- 属性类型:
  - %type    某一列的类型
  - %rowtype 某一行的类型

## 基础语法
语法

``` sql
create or replace procedure 名称(参数列表)
is|as PL/SQL子程序体

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



## 二进制文件操作

数据库目录准备

```sql
-- 创建目录
create directory file_dir as 'F:\ORACLE_Workspace\database\file';
-- 目录赋予权限,读写权限
grant read , write on directory file_dir to zhangqian;
-- 查看所有目录
select * from all_directories;
-- 删除目录 
drop directory file_dir;
```

在创建有数据库目录后, 将文件写入到数据库文件夹中

```sql
-- 二进制文件操作
create table image_file(img_id number primary key , img blob);

-- 存储文件
declare
  v_img_id         number;                    -- 主键
  v_img_name       varchar2(200);             -- 文件路径
  v_img_file       bfile;                     -- 文件对象,指向操作系统下的文件
  v_img_blob       blob;                      -- 二进制
  v_length         number;                    -- 长度
begin
  -- 插入空对象
  insert into image_file values(1,empty_blob());
  
  -- 获得插入的对象,放入内存中,进行修改
  select img into v_img_blob from image_file where img_id = 1;
  
  v_img_name := 'TIM.jpg';
  -- 读取文件,读取为二进制文件对象
  v_img_file := bfilename(directory => 'IMAGE_DIR', filename => v_img_name);  -- 传入数据库目录和改目录下的文件名
  
  -- 打开二进制文件对象 
  dbms_lob.open(v_img_file);
  
  -- 获得文件长度[有多个重载方法]
  v_length := dbms_lob.getlength(v_img_file);
  
  -- 装载文件
  dbms_lob.loadfromfile(dest_lob    => v_img_blob,  -- 目标文件,为加载到内存的表字段
                        src_lob     => v_img_file,  -- 源文件,为打开的当前文件
                        amount      => v_length,    -- 文件长度
                        dest_offset => 1,
                        src_offset  => 1);
  -- 关闭文件[有多个重载方法]
  dbms_lob.close(v_img_file);
  
  -- 提交事务
  commit;
end ;
-- 查看效果
select * from image_file ;
```

读取数据库目录中的文件

```sql
-- 读取文件
declare
  v_file_path         varchar2(2000);    -- 文件路径
  v_file_name         varchar2(200);     -- 文件名
  v_file              utl_file.file_type;-- 文件类型
  v_str               varchar2(2000);    -- 输出
begin
  v_file_path := 'FILE_DIR';
  v_file_name := 'file1.txt';
  -- 打开文件,文件路径,文件名,读模式打开,缓存长度
  v_file := utl_file.fopen(v_file_path , v_file_name ,'r' , 200);
  -- 循环读取文件
  loop
    utl_file.get_line(v_file,v_str);
    dbms_output.put_line(v_str);
  end loop;
  -- 关闭文件
  utl_file.fclose(v_file);
exception
  when no_data_found then            -- 结尾不做处理
    null;
  when others then
    v_str := sqlerrm(sqlcode);       -- 获取错误信息
    dbms_output.put_line(v_str);
end ;
```



## 过程和函数的区别
过程
* 作为PL/SQL语句执行
* 在规则说明中不包括return子句
* 不返回任何值
* 可以包括return语句,但是与函数不同,不能用于返回值

函数
* 作为表达式的一部分
* 必须在规则说明中方包含return子句
* 必须返回单个值
* 必须包含至少一条return语句
