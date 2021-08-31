---
title: Oracle基础-查询篇
abbrlink: 5c90839c
date: 2017-09-22 13:20:50
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

> Oracle 查询语句

<!--more-->



## 内容简介

对Oracle数据库中常用单表查询和多表联合查询进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。

## 字段查询

### SQL执行顺序
在Oracle中,SQL子句的执行步骤依次为:
1. 执行From子句,确定数据的来源
2. 执行Where子句,数据过滤
3. 执行Select子句,确定数据列,设置别名
4. 执行Group by子句,对数据进行分组
5. 执行Having子句,对分组后的数据进行过滤
6. 执行Order by子句进行排序

因此,不能再where中使用select的字段别名
但是,可以在Order by子句使用select的别名


### 查询全部字段

``` sql
--查询表中的全部记录
select * from emp;
```

### 查询指定字段

``` sql
-- 部分查询,并置空字段
select null "empno", ename from emp;
```

### 为字段设置别名
Oracle中,通过为字段设置别名,可以便于后续子查询查询,或者文档导出.
对于中文的字段别名,推荐使用`" "`双引号引住,便于包裹的别名使用特殊字符.
``` sql
--给字段设置别名,只体现输出时,不会改变原来已有的结构
select ename as "姓名", job as "工作" from emp ;                                -- 别名,推荐加双引号
 
select deptno 部门号, dname 部门名称 from dept;                             -- 表的别名,不要加as 关键字
```


### 去重查询

``` sql
--去重查询,表中的不重复的记录,根据记录全部字段是否重复进行去重
select distinct * from emp;
```

### 行转列

``` sql
--行转列
--  select deptno 部门号 ,ename 部门员工 from emp group by deptno
select deptno 部门号 ,wm_concat(ename) 部门员工 from emp group by deptno;    -- 将原来不能分组的显示的通过行转列实现
select job 职位,wm_concat(ename) 工作人员 from emp group by job;             -- 将集合中的记录的字段,根据job进行横向分组,每一个字段对应唯一一个组
```

### 滤空函数

``` sql
--非空与滤空 (以求平均奖金为例)
select sum (comm)/count(*)  非空算法 from emp;
select sum (comm)/count(comm) 滤空算法 from emp;
select avg(comm) 滤空算法 from emp;
--nvl(字段名,补值) 用来将空值替换补充,有空的算式计算结果均为空
select  count(*) 总数  , count(comm) 滤空后 ,count(nvl(comm,0))补空后 from emp;

--如果为空,则替换为0
select nvl(null , '0') from dual;
--如果为不为空,则替换为1,否则替换为0
select nvl2(null , '1' , '0') from dual; 
-- 相同返回null , 不相同返回第一个
select nullif(1,1),nullif(1,2) from dual;

-- 多空值, 如果第一个为空,则显示第二个,如果仍未空,则显示下一个,如果最终为空,最终显示为null
select e.* , coalesce(e.comm , e.sal) from emp e;
```

### 比较函数

``` sql
-- 比较函数
select nullif(10,10),nullif(10,100) from dual;								 -- 相同则返回null,不同则返回第一个值
```
### 伪列
#### `rownum`伪列
代表了当前记录在表中的位置.
常用作限制查询数量,TOP-N分析

``` sql
-- 查看rownum
select rownum , e.* from emp e;												 
-- 查询前5行记录
select * from emp
where rownum <= 5;
```


#### `rowid`伪列
代表了当前记录在数据库文件中的位置
`dbms_rowid`程序包声明了对rowid相关的函数
``` sql
select 
rowid ,
dbms_rowid.rowid_object(rowid) 数据对象号, 
dbms_rowid.rowid_relative_fno(rowid) 相对文件号,
dbms_rowid.rowid_block_number(rowid) 数据块号,
dbms_rowid.rowid_row_number(rowid) 数据行号,
e.*
from emp e;
```

**删除重复数据**
根据业务定义的有重复列进行分组,使用rowid范围删除分组后比min(rowid)大的即可

``` sql
-- 删除empno,ename有重复数据的记录
delete from emp
where rowid not in (
  select min(rowid) 
  from emp
  group by empno , ename 
);
```


### 字段解释

``` sql
--对查询结果进行case解释
--1)选择型
select ename 姓名,job 工作,
case job
  when 'CLERK' then '品监部'
  when 'SALESMAN' then '销售部'
  when 'ANALYST' then '分析部'
  else '高管层'
end as 部门                                                                -- 对then后的解释字段名修改名字
from emp;
 
--2)搜索型,仅限于明令行输入
select ename 姓名,job 工作,                                                -- 注意逗号
case
  when job = 'CLERK' then '品监部'
  when job = 'SALESMAN' then '销售部'                                      --表中的字段,必须用单引号引住
  when job = 'ANALYST' then '分析部'
  else '高管层'
end as 部门                                                                --as 部门 可以省略
from emp ;
 
--3)判断型
select ename as 姓名,job as 工作,
decode(job, 'CLERK' , '品监部','SALESMAN','销售部','ANALYST','分析部' ,'高管层') as 部门
from emp;
```

### 伪列查询

``` sql
--找出工资表中工资最高的三位员工的信息
select rownum,ename,empno,sal
from  (select * from emp order by sal desc)
where rownum<=3;
```

### long类型查询
long类型在数据库中,不能直接查询.
这里通过`PL/SQL`,将`long`类型的字段重新查询,并作为函数结果返回.受限于SQL的字段最大查询长度,这里仅将查询结果的前4000长度进行返回.
也可用过 `select long_to_char('AAAAAAA', 'USER', 'TABLE_NAME',  'COLUMN_NAME') from dual` 调用

``` sql
create or replace function long_to_char(in_rowid      rowid    -- 数据ID,
                                        in_owner      varchar  -- 表所有者 ,
                                        in_table_name varchar  -- 表,
                                        in_column     varchar2 -- 字段)
  return varchar as
  text_c1 varchar2(32767);
  sql_cur varchar2(2000);
begin
  sql_cur := 'select ' || in_column 
          ||  ' from' || in_owner || '.' || in_table_name 
          || ' where rowid = ' 
          || chr(39) || in_rowid || chr(39);
  execute immediate sql_cur
    into text_c1;
  text_c1 := substr(text_c1, 1, 4000);
  return text_c1;
end;
```
另外,可以通过在insert语句中,使用`to_lob()`函数,将`long`类型字段以`clob`形式存入表.

## 条件查询

### 模糊查询

``` sql
--模糊查询
--查询以K开头的员工姓名
select * from emp
where ename like 'K_';
 
--查询以G结尾的员工姓名
select * from emp
where ename like '%G';
```

### 高级模糊查询
效率比Like关键字更快.

``` sql
-- 查询以K开头的员工姓名
select * from emp
where instr(ename,'K') = 1;

--查询包含M,但是不以M开头的员工姓名
select * from emp
where instr(ename,'M') > 1;

--查询不包含N的员工姓名
select * from emp
where instr(ename,'N') = 0;
```




### 范围查询

``` sql
--范围查询
select * from emp
where sal between 1000 and 3000;                                           -- 注意小数在前,大数在后
```

### 包含查询

``` sql
--查询部门名称是SALES和ACCOUNTING的员工信息
--方式一
select e.* from emp e,dept d
where e.deptno=d.deptno
and d.dname in('SALES','ACCOUNTING');

--方式二
select e.* from emp e,dept d
where e.deptno=d.deptno
and(d.dname='SALES' or d.dname='ACCOUNTING');
```
注意:使用 `not in`查询时,不能再在范围中出现`NULL`,否则将会不会查询出结果

``` sql
select * from emp e
where e.comm not in (null);
```


### 空值查询
注意,条件查询中不能使用`null`作为查询条件,如`null=null`,`null > 1`等
此时,将会永远返回`false`,使查询结果为空.

``` sql
select * from emp
where comm is not null;									--查询非空的

select * from emp
where comm is null;											--查询为空的
```



### 排序查询

``` sql
--查询排序
--1)升序
select * from emp
order by empno asc;                                                        --asc 升序
 
--2)降序
select * from emp
order by empno desc;                                                         --desc降序,默认
 
--3)组合排序
select * from emp                                                          --中间用逗号分隔,指明降序,可省略升序
order by empno asc,sal desc;

--4)排序,取极值纪录
select * from emp 
order by empno desc 
fetch first row only;

--5)排序,字段值相同,则并列
select * from emp
order by empno desc 
fetch first row with ties;
 
--6)排序,取总记录的百分比数量
select * from emp
order by empno desc 
fetch first 20 percent rows only;										--降序排列,取前20%

--7)排序,取偏移量的纪录
select * from emp
order by empno desc 
offset 10 rows 															--从第10行开始
fetch first 20 rows only;												--再取20条纪录

--8)排序.取百分比偏移量的纪录
select * from emp
order by empno desc 
offset 10 rows 															--从第10行开始
fetch first 20 percent rows only;										--再取20%的纪录
```
### 分组函数

``` sql
--分组函数
select deptno,avg(sal)                                                       --被包含
from emp
group by deptno ;                                                            --包含
     
--依据部门,职位,统计员工的工资总和
select deptno,job,sum(sal)
from emp
group by deptno,job
order by deptno;
```
### 分组条件函数

``` sql
--having后可以跟分组函数
--选出以部门平均工资分组组后,仍大于30号部门工资最大值的部门
select deptno,avg(sal) 平均工资 from emp
group by deptno
having avg(sal) > (select max(sal) from emp where deptno=30);

--查询部门最低工资大于20号部门最低工资的部门号和部门最低工资
select deptno,min(sal)
from emp
group by deptno
having min(sal)>(select min(sal) from emp where deptno=20);
```


### 分组报表

``` sql
--group by的增强(报表功能)
select deptno,job,sum(sal)from emp group by rollup (deptno,job);
=
select deptno,job,sum(sal)from emp groupby deptno,job
+
select deptno,sum(sal) from emp group by deptno
+
select sum(sal) from emp
```

### 分组去重

``` sql
-- 分组统计去重.以job作为唯一条件,取出相同job中的最新(id最大)的记录
select empno, ename, sal, job
  from (select e.empno, e.ename, e.sal, e.job, row_number() OVER(PARTITION BY e.job ORDER BY e.empno desc) as row_flg
          from emp e)
where row_flg = 1;
```

### 比较查询

``` sql
--any 表示与集合中的任何一个值比较
--查询工资比30号部门任意一个员工都高的员工信息  (实际满足大于最小值就好)
select * from emp
where sal>any (select sal from emp where deptno=30);
 
--all 表示与集合中的所有值比较
--查询工资比30号部门所有员工都高的员工信息  (实际满足大于最大值就好)
select * from emp
where sal>all (select sal from emp where deptno=30);
```

### 分页查询

``` sql
--分页查询
--查询工资排名在5到8之间的员工 (为开区间)
select * from (select rownum r, a.* from ( select emp.* from emp order by sal desc )a where rownum<8)b where r>5;
```

### 分片查询
`fetch`函数支持根据记录数或者比例抓取查询结果
语法:

``` sql
select  XXX
from XXX
group by XXX
having XXX
order by XXX
fetch first 行数 | [offset 开始位置 rows fetch next 个数] | [fetch next 百分比 percent] row only
```

**示例**

``` sql
-- 前五条
select * from emp order by sal
fetch first 5 row only;

-- 第4-5条
select * from emp order by sal
offset 3 fetch next 2 row only;

-- 取一半
select * from emp order by sal
fetch next 50 percent row only;
```


### 存在查询

``` sql
-- 查询薪资低于1.5K的员工
select * from emp e                                                        -- 查询的条数,有exists中符合条数确定
where exists(  select * from emp where e.sal <1500   );
```


## 连接查询
### 等值与不等值连接

``` sql
--等值连接
--查询员工号,姓名,月薪,部门名称
select e.empno,e.ename,e.sal,d.dname
from emp e,dept d
where e.deptno=d.deptno;
 
--不等值连接
--查询员工号,姓名,月薪,薪水的级别
select e.empno,e.ename,e.sal,s.grade
from emp e ,salgrade s
where e.sal between 0 and 3000;
```

### 自连接查询
可以在同一个SQL的同一张表使用不同的别名,完成对同一张表的自我关联.

``` sql
--查找员工和员工老板的姓名
select e.ename 员工姓名 , b.ename 老板姓名
from emp e ,emp b                                                            -- 通过别名,将同一张表的字段相关联
where e.mgr=1b.empno;

--或者使用外连接,注意外连接
select e.empno , e.ename , e.mgr , h.empno , h.ename from emp e , emp h
where e.mgr = h.empno(+)
```


### 左外连接查询

``` sql
--左链接
--1)方式一
select e.ename,d.dname
from emp e                                                                   -- 中心表,从左往右连接
left join dept delete                                                        -- 补充表
on e.deptno = d.deptno
 
--2)方式二
select e.ename,d.dname
from emp e,dept d
where e.deptno = d.deptno(+);

--3)方式三
select *
from emp e
left outer join dept d
on ( e.deptno = d.deptno )
```


### 右外连接查询

``` sql
--右链接
--1)方式二
select e.ename,d.dname
from emp e                                                                   -- 中心表,从右往左连接
right join dept delete                                                       -- 补充表
on e.deptno = d.deptno;
 
--2)方式二
select e.ename,d.dname
from emp e,dept d
where e.deptno(+) = d.deptno;

--3)方式三
select *
from emp e
right outer join dept d
on ( e.deptno = d.deptno )
```

### 全连接查询

``` sql
--自连接中的全连接
select e.ename 员工姓名 , b.ename 老板姓名
from emp e
full join emp b
on e.mgr = b.empno;

-- 或者
select *
from emp e
full outer join dept d
on ( e.deptno = d.deptno )
```


### 层次查询

``` sql
--层次查询
select level,empno,ename,sal,idn
from emp
connect by prior empno=idn                                                   --自连接条件
start with empno=7839                                                        --开始的位置
order by level;                                                         	 --以树的深度决定
```

### 交叉查询
交叉查询的结果将会是一个笛卡尔集合
``` sql
select * 
from emp 
cross join dept;
```

### 自然连接
通过两张表的共同字段进行连接

``` sql

-- 自然连接,将指定连接放在开头
select *
from emp
natural join dept;

```

### 指定连接
指定字段关联连接
``` sql
-- 指定连接 using 设置连接字段
select *
from emp
join dept 
using(deptno);
```



## 子表查询
可以出现在select , from , where , having 之中,其中
from    子句一般为多行多列作为一张临时表进行查询
where   子句一般为单行单列,单行多列,多行单列
having  子句一般为单行单列,便于使用统计函数操作

### select子查询

``` sql
select empno,ename,sal ,(select job from emp where empno=e.empno) as job from emp e;
```

### from子表查询

``` sql
--from后的子查询可以看做一个新表
--查询部门编号,员工编号,工资
select a.* from (select empno,ename,sal from emp) a;                         -- 这里a为一个新表
```

### where子查询

``` sql
--查询与scott工资相同的雇员,注意括号
select * from emp e 
where (e.job , e.sal) =
(select job , sal from emp where ename = 'SCOTT')
and e.ename <> 'SCOTT';

--查询与ALLEN同一年入职相同职位人,注意括号
select * from emp e
where (e.job , to_char( e.hiredate , 'yyyy')) = 
(select job , to_char( hiredate , 'yyyy') from emp where ename = 'ALLEN');

-- some 和 any 的效果是相同的,大于子查询的最小值 1250
select * from emp where sal > some (select sal from emp where job = 'SALESMAN');
```

### 相关子查询

``` sql
--找出员工表中薪水大于本部门平均薪水的员工
--方式一:相关子查询
select empno,ename,sal from emp e
where sal > ( select avg(sal) from emp where deptno=e.deptno );               --这里子查询可以通过别名,调用主查询的字段
--方式二:联合查询
select e.empno,e.ename,e.sal,d.avgsal
from emp e,( select deptno, avg(sal) as avgsal from emp group by deptno ) d
where e.deptno = d.deptno
and e.sal > d.avgsal;
```

**注意**
-- in 与 not in,当子查询含有空值时,not in将会不有返回结果. 
```sql
SMITH的子查询结果为空,则整体返回为空值 
select * from emp e
where e.comm not in ( select comm from emp where ename in ('SMITH','ALLEN'));
```

## with关键字查询

### with子表
在SQL中使用`with`关键字定义临时表,便于后续查询
```sql
-- with子句定义临时表,定义多个则使用逗号分隔
with e as ( select * from emp ),
     d as ( select * from dept)
select * from e , d
where e.deptno = d.deptno;
```

### with子程序
`with`关键字定义查询函数,直接在查询SQL中使用
仅限于12C版本

``` sql
-- with子程序
with function get_emp_dept_name(p_empno number) return varchar2 as
  v_dname varchar2;
begin
  select dname into v_dname from dept where deptno = 
  (select deptno from emp where empno = p_empno)
  return v_dname;
end;
-- 查询
select 
get_emp_dept_name(e.deptno) as emp_dept_name,
e.*
from emp e;
```

## 行转列
查询各部门各个岗位工资的总和
``` sql
select deptno , job , sum(sal)
from emp 
group by deptno , job;
```

### decode行转列
decode方式仅适用于转列值较少的字段
查询各部门各个岗位工资的总和
 1. 首先根据job职位进行decode,将对应职位的薪资取出,使用字段别名作为新字段添加.
 2. 随后根据部门汇总

 ``` sql
 -- 各员工对应岗位薪资
select deptno,
  decode(job,'SALESMAN',sal,0) salesman_job,
  decode(job,'PRESIDENT',sal,0) president_job,
  decode(job,'MANAGER',sal,0) manager_job,
  decode(job,'CLERK',sal,0) clerk_job,
  decode(job,'ANALYST',sal,0) analyst_job
from emp

-- 各部门岗位薪资的总和
select deptno,
  sum(decode(job,'SALESMAN',sal,0)) salesman_job,
  sum(decode(job,'PRESIDENT',sal,0)) president_job,
  sum(decode(job,'MANAGER',sal,0)) manager_job,
  sum(decode(job,'CLERK',sal,0)) clerk_job,
  sum(decode(job,'ANALYST',sal,0)) analyst_job
from emp
group by deptno;
 ```

### 子查询行转列
查询各部门各个岗位工资的总和
1.首先使用不同子查询,查询员工所在部门各岗位薪资和,作为新字段,组合为一张临时表
2.再对临时表分组求和

``` sql
-- 各员工对应部门的岗位薪资和
select deptno,
  (select sum(sal) from emp where job = 'SALESMAN' and deptno = e.deptno) salesman_job,
  (select sum(sal) from emp where job = 'PRESIDENT' and deptno = e.deptno) president_job,
  (select sum(sal) from emp where job = 'MANAGER' and deptno = e.deptno) manager_job,
  (select sum(sal) from emp where job = 'CLERK' and deptno = e.deptno) clerk_job,
  (select sum(sal) from emp where job = 'ANALYST' and deptno = e.deptno) analyst_job
from emp e;

-- 各部门岗位薪资的总和
select deptno, 
sum(salesman_job) salesman_job,
sum(president_job) president_job, 
sum(manager_job) manager_job, sum(clerk_job) clerk_job, 
sum(analyst_job) analyst_job from
(select deptno,
    (select sum(sal) from emp where job = 'SALESMAN' and deptno = e.deptno) salesman_job,
    (select sum(sal) from emp where job = 'PRESIDENT' and deptno = e.deptno) president_job,
    (select sum(sal) from emp where job = 'MANAGER' and deptno = e.deptno) manager_job,
    (select sum(sal) from emp where job = 'CLERK' and deptno = e.deptno) clerk_job,
    (select sum(sal) from emp where job = 'ANALYST' and deptno = e.deptno) analyst_job
  from emp e ) temp
group by deptno;
```

### pivot函数行转列
oracle12c使用`prvot`函数进行行转列,语法
``` sql
select *|字段[别名]...
from 子查询
prvot(
  统计函数 for 转换列名称 in(
    内容1[ [as] 别名],
    内容2[ [as] 别名],
      ....
    内容n[ [as] 别名]
  )
)
```
**示例** 各部门岗位薪资的总和
``` sql
select *
from (
  select deptno ,ename, job ,sal from emp
)
-- 进行转换,一般为两个字段需要行列转换,其中一个字段作为聚合函数,另个字段起别名
pivot(
  sum(sal)
  for job in (
    'SALESMAN' as salesman_job,
    'PRESIDENT' as president_job,
    'MANAGER' as manager_job,
    'CLERK' as clerk_job,
    'ANALYST' as analyst_job
  )
)

-- 带有分析函数的行转列
select *
from (
  select deptno ,ename, job ,sal,
  sum(sal) over (partition by deptno) dept_sal_sum,
  min(sal) over (partition by deptno) dept_sal_min,
  max(sal) over (partition by deptno) dept_sal_max
  from emp
)
pivot(
  sum(sal)
  for job in (
    'SALESMAN' as salesman_job,
    'PRESIDENT' as president_job,
    'MANAGER' as manager_job,
    'CLERK' as clerk_job,
    'ANALYST' as analyst_job
  )
)

-- 多字段行转列
select *
from (
  select deptno ,ename, job ,sal from emp
  where deptno = 20
)
pivot(
  sum(sal)
  for (job,deptno) in (
    ('SALESMAN',20) as salesman_job_20,
    ('PRESIDENT',20) as president_job_20,
    ('MANAGER',20) as manager_job_20,
    ('CLERK',20) as clerk_job_20,
    ('ANALYST',20) as analyst_job_20
  )
)
```
**扩展:转为xml**  `pivot`支持转为XML结构,便于数据格式输出

``` sql
select *
from (
  select deptno ,ename, job ,sal from emp
)
pivot xml(
  sum(sal)
  for job in (any)
)
```
#### unpivot函数
取消行转列
语法
``` sql
select *|字段[别名]...
from 子查询
unprvot [include nulls | exclude nulls](
  统计函数 for 转换列名称 in(
    内容1[ [as] 别名],
    内容2[ [as] 别名],
      ....
    内容n[ [as] 别名]
  )
)
....
include nulls  行变为列转换之后保留所有的null数据
exclude nulls  (默认),行转列之后不保留null数据
```
## 层次查询

**示例** 查询7698员工的下级成员
``` sql
select 
connect_by_iscycle,  -- 是否为循环,是则为1,否则为0
connect_by_isleaf,   -- 是否为叶子节点,是则为1,含有子节点则为0,
connect_by_root ename, -- connect_by_root 字段  查找到根节点的记录字段信息
level, 
empno , 
lpad('|-' , level *2 , ' ') || ename , 
sys_connect_by_path(ename , '=>'),  -- sys_connect_by_path(字段,连接符) 显示节点的位置路径
mgr
from emp e
connect by 
  nocycle            -- cycle | nocycle 是否进行循环查询,默认进行循环查询.当出现环状时,抛出异常
  prior empno = mgr  -- 子节点优先,则向下查询
  and empno != 7698  -- 不显示某节点及其之下
start with mgr is null
order siblings by ename;  -- 保持层次后,进行查询
```

## 正则查询

### 正则拆分

```sql
-- 根据,逗号分隔字符, 最多显示999层
select regexp_substr('1,2,3,4','[^,]+', 1, level) from dual connect by level<=999;
-- 根据 空格分隔字符, 最多显示1层
select regexp_substr('2021-05-10 23:59:00','[^ ]+', 1, level) from dual connect by level<=1;
```

### 正则匹配

```sql
-- 查找字段中非数字的记录
select * from emp where not regexp_like(eid, '^[0-9]+[0-9]$');
```

