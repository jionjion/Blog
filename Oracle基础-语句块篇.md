---
title: Oracle基础-语句块篇
abbrlink: c425ce9d
date: 2017-09-22 23:12:44
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---

## 内容简介
对Oracle数据库中约束PL/SQL语句块的格式，逻辑进行了介绍，以及相关的数据存放方式,进行扩展
所涉及的表为`scott`用户下的`emp`与`dept`表。注意，需要打开控制台输出`set serveroutput on`

## 命名约定

### 变量约定:
- `v_`   表示自定义变量
- `p_`   表示传入的参数
- `e_`   表示自定义的异常
- `r_`   表示行变量
- `c_`   表示游标,结果集
- `arr_` 表示自定义的数组



### SQL代码块的属性

- `sql%rowcount`    整型           代表dml语句成功执行的数据行数  
- `sql%found`       布尔型         值为true代表插入、删除、更新或单行查询操作成功  
- `sql%notfound`    布尔型         与sql%found属性返回值相反  
- `sql%isopen`      布尔型         dml执行过程中为真，结束后为假 



## 语句结构

``` sql
-- 基本结构
declare                                                                    -- 声明(变量说明,游标申明,例外说明)
begin                                                                      -- 语句序列(DML语句)
exception                                                                  -- 异常处理语句 
end;
 
declare                                                                    -- 声明
  p_number number(7,2);                                                    -- 定义基本数据类型,默认为8位整型
  p_name varchar2(20);                                                     -- 字符串变量
  p_date date;                                                             -- 日期变量
begin 
  p_number:=1;                                                             -- 赋值方式
  dbms_output.put_line(p_number);
  p_name:='Tom';
  dbms_output.put_line(p_name);
  p_date:=sysdate;
  dbms_output.put_line(p_date);                                            -- 计算明天的日期
  dbms_output.put_line(p_date+1);                                          -- 输出计算
exception	-- 异常处理
  when others then  -- 捕获异常并处理
     dbms_output.put_line('发生异常...')
end;
```

## 逻辑控制
### if语句

``` sql
--判断用户从键盘中输入的语句
--num:是一个地址值,含义是在该地址上保存了输入的值
accept num prompt'请输入一个数字';                                          -- 提示用户
declare                                                                    -- 定义键盘输入的数字
pnum number :=&num;                                                        -- 地址符号,表示从该地址处取值
begin                                                                      -- 执行条件判断
   
0   pnum = 0 then dbms_output.put_line('您输入的数字为0');
     elsif pnum=1 then dbms_output.put_line('您输入的数字为1');
     elsif pnum=2 then dbms_output.put_line('您输入的数字为2');
     else dbms_output.put_line('您输入的为其他数字');
  end if;
end;
```

### while循环

``` sql
--使while循环打印1-10
declare
  p_num number := 1 ;
begin
  while p_num<=10
  loop      --循环体
      dbms_output.put_line(p_num);
      p_num:= p_num + 1;                                                     --变量自增,不能使用++
  end loop;
end;
```

### loop循环

``` sql
declare
  p_num number := 1;
begin
  loop         --循环体
  dbms_output.put_line(p_num);
  p_num := p_num+1;
  exit when p_num >10;                                                      --退出条件 ;在循环体中的判断条件不固定
  end loop;
end;
```

### for循环

``` sql
declare
  p_num number := 1;
begin
  for p_num in 1..10                                                        --循环1到10,共10次
  loop
    dbms_output.put_line(p_num);
  end loop;
end;
```


### case语句

``` sql
-- 语句块逻辑判断
begin
case 变量
  when 值1 then  ..... ;
  when 值2 then  ..... ;
  else .....;
end ;
```

### 异常处理

抛出一个oracle平台异常,交由后端框架捕获
``` sql
declare
begin
  raise_application_error(-20002,'自定义错误信息!');
end;
```

抛出一个自定义异常,并在数据库端进行捕获
`sqlcode`获得错误代码
`sqlerrm`获得错误的内容信息
`pragma exception_init(自定义异常, 异常代码)`将异常进行包装,向外抛出
``` sql
declare
  e_err exception;  -- 自定义错误
  pragma exception_init(e_err , -20001);  -- 用户自定义异常编码
begin
  raise e_err;
exception
  when e_err then
    dbms_output.put_line('错误编码>>' || sqlcode );
    dbms_output.put_line('错误信息>>' || sqlerrm );
end;
```

## 变量类型

###  %type引用类型
`emp.ename%type`引用类型
``` sql
-- 例:定义引用型数据变量,查询并打印7879员工号的姓名和薪水
declare
  pename emp.ename%type;                                                   -- emp表的ename字段的数据类型
  psal emp.sal%type;
begin
  select ename,sal into pename,psal from emp where empno=7839; 
  dbms_output.put_line(pename||'的薪水是'||psal);                          -- 打印姓名,薪水
end;
```

### %rowtype记录类型
`emp%rowtype` 记录型变量(一行中的元素类型)
通过`emp_rec.ename`进行引用或者赋值
```sql
declare
  emp_rec emp%rowtype;  -- 引用了一整行的数据
begin
  select * into emp_rec from emp where empno=7839;    -- 查询一整行,并赋值,要求结果必须为单行
  if sql%found then       -- 如果成功找到相应的员工,则进行打印
     dbms_output.put_line(emp_rec.ename||'薪水是'||emp_rec.sal);   --取出一行中的具体字段;打印姓名,薪水
  else
     dbms_output.put_line('没有找到相应的员工...'); 
  end if;
end;
```

### record记录类型

``` sql
declare
  -- 定义记录类型
  type emp_type is record(
    empno     emp.empno%type,
    ename     emp.ename%type,
    job       emp.job%type,
    sal       emp.sal%type
  );
  -- 定义复合变量
  r_emp   emp_type;
begin
  select empno , ename, job, sal into r_emp from emp where empno = '7782';
  dbms_output.put_line(' 编号:' || r_emp.empno || ' 姓名' || r_emp.ename || ' 职位' || r_emp.job || ' 薪水' || r_emp.sal);
end;
```

### 嵌套记录类型
在多个`record`间进行嵌套,作为数据对象保存
``` sql
declare
   -- 定义子类型
   type dept_type is record(
     deptno   dept.deptno%type,
     dname    dept.dname%type
   );
   
   -- 定义父类型
   type emp_type is record(
     empno    emp.empno%type,
     ename    emp.ename%type,
     job      emp.job%type,
     sal      emp.sal%type,
     dept     dept_type  -- 引用拼接复核变量
   );
   
   -- 定义复核变量
   r_emp      emp_type;    
begin
  select e.empno , e.ename , e.job , e.sal , d.deptno , d.dname 
  into r_emp.empno , r_emp.ename , r_emp.job , r_emp.sal , r_emp.dept.deptno , r_emp.dept.dname
  from emp e , dept d where e.deptno = d.deptno(+) and e.empno = '7782';
  dbms_output.put_line( ' 部门编号' || r_emp.dept.deptno || ' 部门' || r_emp.dept.dname || 
                        ' 编号:' || r_emp.empno || ' 姓名' || r_emp.ename || ' 职位' || r_emp.job || ' 薪水' || r_emp.sal);
end;
```

### 记录类型更新表数据
可以通过记录类型对表数据进行插入或者更新,但是要求记录类型与表数据结构一致
``` sql
-- 记录更新,定义类型必须与rowtype类型一致,即表记录类型一致
declare 
   type dept_type is record(
     deptno  dept.deptno%type,
     dname   dept.dname%type,
     loc     dept.loc%type
   );
   r_dept    dept_type;
begin
   r_dept.deptno := 50;
   r_dept.dname  := 'STUDENT';
   r_dept.loc    := 'ALASKA';
   insert into dept values r_dept;            -- 插入
   update dept set row = r_dept where dept.deptno = r_dept.deptno;  -- 更新
end;      
```

### 索引表类型
索引表,类似于数组,通过指定的索引键进行访问数据,不需要索引初始化,且索引可以为数字也可以为字符,数字不限制连续与正负
语法
``` sql
type 类型名称 is table of 数据类型[not null]
index by [pls_integer | binary_integer | varchar2(长度)]
```
#### 数字类型的索引表

``` sql
declare
  -- 定义索引表
  type info_index is table of varchar(20)
  index by pls_integer;    -- 使用数字作为索引键
  -- 定义复合变量
  t_info   info_index;
begin
  -- 使用索引
  t_info(1) := 'Hello';
  -- 索引不需要连续
  t_info(5) := 'World'; 
  -- 获得索引数据
  dbms_output.put_line( '获得数据 >> ' || t_info(1) );
  -- exists()函数判断数据是否存在
  if t_info.exists(2) then
    -- 不存在则抛出no_data_found异常
    dbms_output.put_line( '获得数据 >> ' || t_info(2) );
  end if;
  -- 遍历数据
exception
  when no_data_found then
    dbms_output.put_line('数据不存在!');
end;
```

#### 字符串类型的索引表

``` sql
declare
  -- 定义索引表
  type info_index is table of varchar(20)
  index by varchar2(10);    -- 使用字符作为索引键
  -- 定义复合变量
  t_info   info_index;
begin
  -- 使用索引
  t_info('Say') := 'Hello';
  -- 获得索引数据
  dbms_output.put_line( '获得数据 >> ' || t_info('Say') );
end;
```

#### rowtype记录类型索引表

在索引表中,通过使用`rowtype`记录类型,可以定制临时表的结构

``` sql
declare
  -- 定义索引表
  type dept_index is table of dept%rowtype
  index by pls_integer;    -- 使用数字作为索引键
  -- 定义复合变量
  t_dept   dept_index;
begin
  -- 使用索引
  t_dept(0).deptno  := '50';
  t_dept(0).dname   := 'STUDENT';
  t_dept(0).loc     := 'ALASKA';
  -- 获得索引数据
  dbms_output.put_line( ' 编号' || t_dept(0).deptno || ' 部门' || t_dept(0).dname || ' 位置' || t_dept(0).loc );
end;
```

#### record记录类型索引表

使用记录类型作为索引表的内容,自定义临时表的字段和记录
``` sql
declare
   -- 定义记录类型
   type dept_type is record(
     deptno  dept.deptno%type,
     dname   dept.dname%type,
     loc     dept.loc%type
   );
  -- 定义索引表
  type dept_index is table of dept_type   -- 使用自定义数据类型
  index by pls_integer;                   -- 使用数字作为索引键
  -- 定义复合变量
  t_dept   dept_index;
begin
  -- 使用索引
  t_dept(0).deptno  := '50';
  t_dept(0).dname   := 'STUDENT';
  t_dept(0).loc     := 'ALASKA';
  -- 获得索引数据
  dbms_output.put_line( ' 编号' || t_dept(0).deptno || ' 部门' || t_dept(0).dname || ' 位置' || t_dept(0).loc );
end;
```

### 嵌套表类型
#### 单数据类型嵌套表
定义一个只有一列数据的嵌套表,可以使用`1...集合.count`进行遍历
``` sql
declare
  type name_nested is table of varchar2(200) not null;     -- 简单数据结构嵌套表
  v_names          name_nested := name_nested('Jion','Arise'); -- 数据对象,并赋初值
begin 
  -- 获得长度,并遍历     使用 .count获得长度
  for n in 1.. v_names.count loop
    dbms_output.put_line( '遍历获得>> ' || v_names(n) );
  end loop;
end; 
```
相对应的,也可以使用`集合.first .. 集合.last`进行遍历
```sql
-- 使用 集合.first - 集合.last方式访问
declare
  type name_nested is table of varchar2(200) not null;     -- 简单数据结构嵌套表
  v_names          name_nested := name_nested('Jion','Arise'); -- 数据对象,并赋初值
begin 
  -- 获得长度,并遍历     使用 .count获得长度
  for n in v_names.first .. v_names.last loop
    dbms_output.put_line( '遍历获得>> ' || v_names(n) );
  end loop;
end; 
```

##### 示例

查询几个变量的中的极值

```plsql
-- 首先声明一个全局的数据类型
create or replace type mytable as table of varchar2(100);

-- 示例,查询最大值
declare
  my_list     mytable;
  v_max       varchar2(50);
begin
  -- 字符串
  my_list := mytable('a','c','d');      
  select max(column_value) into v_max from table(my_list);
  dbms_output.put_line(v_max);
  
  -- 数字
  my_list := mytable(1024, 4096, 512);      
  select max(to_number(column_value)) into v_max from table(my_list);
  dbms_output.put_line(v_max);  
  
  
  -- 日期
  my_list := mytable('2010-01-01','2015-01-01','2020-01-01');
  select max(to_date(column_value,'yyyy-MM-dd')) into v_max from table(my_list);
  dbms_output.put_line(v_max);
end;
```





#### 复核数据类型嵌套表

这里我们先声明一种对象类型,并在SQL中引用

``` sql
-- 创建对象数据类型
create or replace type dept_others_obj as object
(
  depiction varchar2(200),
  create_user number,
  create_date date
);
```

在PL/SQL中使用自定义的数据类型
``` sql
-- 使用已定义的数据类型
declare
  v_others         dept_others_nested  := dept_others_nested(dept_others_obj('PL/SQL录入描述',3,sysdate),
                                                             dept_others_obj('PL/SQL继续录入',4,sysdate));
  r_dept           dept%rowtype;
begin
  -- 记录赋值
  r_dept.deptno := '20';
  r_dept.dname  := 'RESEARCH';
  r_dept.loc    := 'DALLAS';
  r_dept.other  := v_others;
  -- 插入
  insert into dept values r_dept;
  -- 更新
  update dept set other = v_others
  where deptno = '20';
  -- 循环遍历
  for n in v_others.first .. v_others.last loop
    dbms_output.put_line( '描述' || v_others(n).depiction || ' 创建人' || v_others(n).create_user || ' 创建日期' || v_others(n).create_date);
  end loop;
end;
```



### 数组类型
#### 单数据类型的数组

``` sql
declare 
  -- PL/SQL 使用 is
  type other_varray is varray(3) of varchar2(50);
  v_other_varray  other_varray := other_varray(null , null , null);
begin
  -- 赋值
  v_other_varray(1) := 'A';
  v_other_varray(2) := 'B';
  v_other_varray(3) := 'C';
  -- 循环
  for i in v_other_varray.first .. v_other_varray.last loop
    -- 判断是否存在
    if v_other_varray.exists(i) then
       dbms_output.put_line('数组第'|| i ||'个元素:' ||v_other_varray(i));
    end if;
  end loop;
end ;
```

#### 复核数据类型的数组
声明对象数据类型
``` sql
-- 创建对象
drop type dept_others_obj;
create or replace type dept_others_obj as object
(
  depiction varchar2(200),
  create_user number,
  create_date date
);
```

引用对象数据类型,在PL/SQL中使用
``` sql
-- PL/SQL中使用符合类型
declare
  type other_varray is varray(3) of dept_others_obj;
  v_other_varray  other_varray := other_varray(dept_others_obj('这是会计部门',1,sysdate),
                                               dept_others_obj('地址在纽约',2,sysdate));
begin
  -- 循环
  for i in v_other_varray.first .. v_other_varray.last loop
    -- 判断是否存在
    if v_other_varray.exists(i) then
       dbms_output.put_line('数组第'|| i ||'个元素:' || v_other_varray(i).depiction 
                             || '创建者' || v_other_varray(i).create_user 
                             || '创建时间' || to_char(v_other_varray(i).create_date,'yyyy-MM-dd'));
    end if;
  end loop;
end;
```

## 集合操作
### 集合运算符
oracle提供了对集合对象的运算,常用方法有

| 方法 | 说明 |
| ---- | ---- |
|`cardinality(集合)`  |            获得集合内的元素个数|
|`变量 is [not] empty`  |          判断集合是否为NULL|
|`变量 member of 集合`  |          判断某一数据是否为集合中的成员|
|`集合1 multiset except 集合2`  |    从一个集合中删除另一个集合中的相同部分数据,并返回新集合|
|`集合1 multiset intersect 集合2`  | 取出两个集合之中相同的部分,返回一个新集合|
|`集合1 multiset union 集合2`  |   将两个集合合并为一个集合 |
|`set`                         |  1.删除集合中的重复数据,类似于distinct操作;  2.删除重复数据 set(集合);   3.判断集合是否为空 `变量 is [not] a set`|
|`集合1 submultiset of 集合2`|    判断集合1是否是集合2的子集合|

``` sql
declare
  -- 定义嵌套表
  type list_nested is table of varchar2(100) not null;
  v_list_a            list_nested;
  v_list_b            list_nested;
  v_list_except       list_nested;  -- 差集 A-B
  v_list_intersect    list_nested;  -- 交集 A∩B
  v_list_union        list_nested;  -- 并集 A∪B
  
begin 
  v_list_a := list_nested('A','B','C','D','D','C','B','A');
  v_list_b := list_nested('C','D','E','F');
  dbms_output.put_line('去重后,集合长度:' || cardinality(set(v_list_a))); -- 长度4
  dbms_output.put_line('去重前,集合长度:' || cardinality(v_list_a));  -- 长度8
  -- 集合是否不为空
  if v_list_a is not empty then
    dbms_output.put_line('判断集合不为空!');
  end if;
  -- 判断元素在集合是否存在
  if 'A' member of v_list_a then
    dbms_output.put_line('A存在于集合中!');
  end if;
  -- 集合运算,差集
  v_list_except := v_list_a multiset except v_list_b;
  for i in v_list_except.first .. v_list_except.last loop
    dbms_output.put_line('差集:' || v_list_except(i));
  end loop;
  -- 集合运算.交集
  v_list_intersect := v_list_a multiset intersect v_list_b;
  for i in 1 .. v_list_intersect.count loop
    dbms_output.put_line('交集:' || v_list_intersect(i));
  end loop;
  -- 集合运算,并集
  v_list_union := v_list_a multiset union v_list_b;
  -- A是否为集合
  if v_list_a is not a set then
    dbms_output.put_line('A不是一个集合');
  end if;
  -- B是否为A的子集
  if v_list_b submultiset of v_list_a then
    dbms_output.put_line('B是A的子集!');
  else
    dbms_output.put_line('B不是A的子集!');
  end if;
end;
```

### 集合函数

| 集合函数 | 说明 |
| ------ | ---- |
|`集合.count`|                  获得集合长度|
|`集合.delete(m)`|              置空索引m的元素|
|`集合.delete(m,n)`|            置空索引\[m,n\]索引范围内的元素|
|`集合.exists(m)`|              判断索引位是否存在,当元素被置空或者索引所在位置尚未初始化时,返回为false,存在且不为空则返回true|
|`集合.extend(m[,n])`|        扩充集合长度,参数m表示扩充的长度,n表示扩充时以第n位的元素进行扩充,不写默认扩充一个空内容,长度+1|
|`集合.first`|                  返回集合最小的索引|
|`集合.last`|                   返回集合最大的索引|
|`集合.limit`|                  返回集合中允许访问到的最大索引值,没有限制则返回空|
|`集合.next(m)`|                返回集合指定索引位置的下一个索引值,没有则返回false|
|`集合.prior(m)`|               返回集合的前一个索引值,没有则返回false|
|`集合.trim([m])`|              释放集合内的元素,如果没有参数,则表示释放集合内最高元素;否则表示释放指定数量的元素|


``` sql
declare
  -- 定义嵌套表
  type list_nested is table of varchar2(100) not null;
  v_list            list_nested;
begin
  -- 赋值
  v_list := list_nested('A','B','C','D');
  -- 置空第一个元素,第一个元素置空
  v_list.delete(1);
  -- 置空第[1,2]个元素,1,2元素置空
  v_list.delete(1,2);
  -- 集合长度
  dbms_output.put_line('集合长度:' || v_list.count);
  -- 判断第一位是否存在,置空,不存在
  if v_list.exists(1) then
    dbms_output.put_line('第一位存在!');
  end if;
  -- 扩展元素,使用第四位的D扩充为第五位
  v_list.extend(1,4);
  v_list.extend(1,4);
  -- 判断第五位是否存在
  if v_list.exists(5) then
    dbms_output.put_line('第五位存在:' || v_list(5));
  end if;  
  -- 返回集合索引的开始和结束  [3,5]
  dbms_output.put_line('集合开始索引:' || v_list.first || ' 集合结束索引:' || v_list.last || ' 集合最大访问长度:' || v_list.limit);
  -- 返回集合下一个元素
  dbms_output.put_line('集合下一个元素索引:' || v_list.next(1));
  -- 返回集合上一个元素
  dbms_output.put_line('集合下一个元素索引:' || v_list.prior(1));
  -- 释放集合内的元素
  v_list.trim();
  dbms_output.put_line('释放后集合长度:' || v_list.count);
end;
```

### 集合异常
| 异常 | 说明 |
| ----- | ---- |
|`collection_is_null`|                集合未初始化,直接使用时触发|
|`no_data_found`|                     数据未找到|
|`subscript_beyond_count`|            访问索引超过集合中元素个数时触发|
|`subscript_outside_limit`|           访问索引超过定义时的最大长度时触发|
|`value_error`|                       不能转为PLS_INTEGER索引时触发,即索引只能为数字|


`collection_is_null` 集合未初始化异常
``` sql
declare
  type                 list_array is varray(10) of varchar2(50);
  v_list               list_array;  -- 集合尚未初始化
begin
  v_list(0) := 'A';
exception
  when collection_is_null then
    dbms_output.put_line('集合尚未初始化!');
end;
```

`subscript_beyond_count` 访问集合长度超过集合元素
```sql
declare
  type                 list_array is varray(10) of varchar2(50);
  v_list               list_array := list_array('A','B');
begin
  dbms_output.put_line('访问集合元素:' || v_list(4)); -- 越界
exception
  when subscript_beyond_count then
    dbms_output.put_line('下标超出数量!');
end;
```

`subscript_outside_limit 访问长度超过集合长度限制
``` sql
declare
  type                 list_array is varray(10) of varchar2(50);
  v_list               list_array := list_array('A','B');
begin
  dbms_output.put_line('访问集合元素:' || v_list(100));  -- 超长
exception
  when subscript_outside_limit then
    dbms_output.put_line('下标超出最大限制长度!');
end;
```

`value_error` 索引结构异常
``` sql
declare
  type                 list_array is varray(10) of varchar2(50);
  v_list               list_array := list_array('A','B'); 
begin
  dbms_output.put_line('访问集合元素:' || v_list('A'));
exception
  when value_error then
    dbms_output.put_line('索引数据结构异常!');
end;
```

`no_data_found` 获取数据为空

``` sql
declare
  -- 索引表
  type list_table is table of varchar2(100) index by pls_integer;
  v_list            list_table;
begin
  v_list(1) := 'A';
  v_list.delete(1);
  dbms_output.put_line('访问集合元素:' || v_list(1));
exception
  when no_data_found then
    dbms_output.put_line('数据未找到!');
end;
```

### 集合遍历

#### for循环遍历
```sql
-- 使用for循环进行处理
declare
  type emp_array is varray(3) of emp.empno%type;
  v_empno_arr    emp_array := emp_array(7369,7499,7521);
begin
  for i in v_empno_arr.first .. v_empno_arr.last loop
    update emp set sal = sal + 1
     where empno = v_empno_arr(i);
  end loop;
end;
```

#### forall批量读取
```sql
-- 使用FORALL进行批处理
declare
  type emp_array is varray(3) of emp.empno%type;
  v_empno_arr    emp_array := emp_array(7369,7499,7521);
begin
  forall i in v_empno_arr.first .. v_empno_arr.last
    update emp set sal = sal + 1
     where empno = v_empno_arr(i);
     
  for i in v_empno_arr.first .. v_empno_arr.last loop
    dbms_output.put_line('雇员编号:' || v_empno_arr(i) || '更新操作影响行数:' || sql%bulk_rowcount(i));
  end loop;
end;
```

#### bulk collect批量读取

``` sql
declare 
  type ename_array is array(3) of emp.ename%type;
  v_ename_arr      ename_array;
begin
  select ename 
  bulk collect into v_ename_arr
  from emp where rownum <= 3;
  
  for i in v_ename_arr.first .. v_ename_arr.last loop
    dbms_output.put_line('前三名员工为:' || v_ename_arr(i));
  end loop;
end;
```