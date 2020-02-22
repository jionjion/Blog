---
title: Oracle基础-内置函数篇
abbrlink: 3b02ae60
date: 2017-09-20 11:09:30
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

----------
## 内容简介

对Oracle数据库中常用的数值函数，日期函数，字符串函数，以及聚合函数进行了介绍，所涉及的表为`scott`用户下的`emp`与`dept`表。


## 数值函数
### 四舍五入函数

``` sql
--1)小数点后取舍   
select round(5555.5555,2)  from dual;                                      -- 返回5555.56                    
select round(5555.5555)    from dual;                                      -- 返回5556
select round(-5555.5555,2) from dual;                                      -- 返回-5555.56                    
select round(-5555.5555)   from dual;                                      -- 返回-5556
 
--2)小数点前取舍   
select round(5555.5555,-2) from dual;                                      -- 返回5600
select round(5555.5555,-6) from dual;                                      -- 返回0
select round(-5555.5555,-2) from dual;                                     -- 返回-5600
select round(-5555.5555,-6) from dual;                                     -- 返回0
```

### 取整函数

``` sql
--1)正无穷方向取整
select ceil(1.5)  from dual;                                               -- 返回2
select ceil(-1.5) from dual;                                               -- 返回-1
--2)负无穷方向取整
select floor(1.5) from dual;                                               -- 返回1
select floor(-1.5)from dual;                                               -- 返回-2
```
### 截取数字
``` sql
-- 截取,1位小数,默认截取整数
select trunc(15.79,1) from dual;
```


### 正负号函数

``` sql
select sign(100),sign(-100),sign(0) from dual;                             -- 正值返回1，负值返回-1，0返回0

```

### 绝对值函数


``` sql
select abs(2.5) from dual;
select abs(-2.5)from dual;                                                 --返回2.5
```

### 取余函数

``` sql
select mod(5,2) from dual;                                                 --5%2为1
```

### 幂函数

``` sql
select power(3,2) from dual;                                               --3的2次方

select exp(1) from dual;                                     -- 自然对数e的0次幂,1次幂,2次幂
```

### 对数函数

``` sql
select log(2,4) from dual;                                                 -- log以2为底,4的对数

select ln(2.714) from dual;                                                -- log以e为底,2.714的对数
```

### 平方根函数

``` sql
select sqrt(4) from dual;
```

### 截取任意精度

``` sql
--截取任意精度  正数:小数点后保留位数   负数:小数点前元整位数    0或缺省:截取整数
 select trunc(12345.6789,2),trunc(12345.6789,-2),trunc(12345.6789)  from dual;
```

### 随机数

``` sql
-- [0,1)之间的随机数
select dbms_random.value from dual;                                        -- [0,1)范围

-- 指定范围的随机数小数
select dbms_random.value(0,100) from dual;

-- 指定范围的随机整数
select trunc(dbms_random.value(0,100)) from dual ;

-- 正态分布的随机数
select dbms_random.normal from dual ;
```
### 中位数,标准差,方差
`median()` 中位数
`stddev()` 标准差
`variance()` 方差
``` sql
select median(sal) ,stddev(sal) , variance(sal) from emp;
```


### 三角函数

``` sql
-- 三角函数(弧度制)
select sin(3.1515926/2) from dual;
select cos(3.1515926/2) from dual;
select tan(3.1515926/2) from dual;
select sinh(2.714) from dual;
select cosh(2.714) from dual;
select tanh(2.714) from dual;
select asin(0.5) from dual;
select acos(0.5) from dual;
select atan(0.5) from dual;
```
### 数字字符转换函数

``` sql
--数字转为字符
select to_char(12345.6789,'999999999999999')from dual;                      --显示整数位,格式化不足则会显示##,位数太多前面空格填充
 
select to_char(12345.6789,'99999.99')from dual;                             --小数点后保留2位,四舍五入保存
 
select trim(to_char(12345.6789,'999,999,999,999.99')) from dual;            --显示千位符,并保留两位小数       
 
select to_char(12345.6789,'$99,999.99')from dual;                           --显示美元符号
 
select to_char(12345.6789,'s99999.99')from dual;                            --显示正号
--将字符转为数字
select to_number('$12,345.67','$99,999.99') from dual;                      --去掉美元符号,千分号
```


## 字符函数

### ASCII转换函数
``` sql
--查看字母的ASCII值
select ascii('A'),ascii(1),ascii('你') from dual;                           -- 字母和数字智能填一个,多填以第一个为主

--将ASCII值转为字符
select chr(65),chr(49),chr(50403) from dual;                               -- 注意数字的位数
```
### 大小写转换函数

``` sql
--小写转大写
select upper ('hello') from dual;                                          --注意字符串需要加引号
 
--大写转小写
select lower ('HELLO') from dual;
 
--首字母大写
select initcap('good') from dual;
```

### 字符索引函数
单行函数,查找字符串索引位置,索引从1开始.
``` sql
-- 存在返回下角标
select instr('a b c d e' , 'a') from dual;
-- 返回下角标
select instr('a b c d e' , 'c') from dual;
-- 不存在,返回0
select instr('a b c d e' , 'f') from dual;
```

### 字符串截取函数

``` sql
--1)从开始索引到最后
select substr('string',3) from dual;                                       --返回 ring ,在字符串位置开始处为1
 
--2)从开始索引获取后面指定长度的元素
select substr('string',3,2) from dual;                                     --返回 ri
 
--3)从结尾索引到最后
select substr('string',-3) from dual;                                      --返回 ing
 
--4)从结尾索引获取到后面指定长度的元素
select substr('string',-3,2)from dual;                                     --返回 in
 
--!)从结尾索引获取到前面指定长度的元素
select substr('string',-3,-2)from dual;                                    --没有这个用法
```

### 字符串长度函数

``` sql
--获取字符串长度
select length('string') from dual;

--获取字符串byte长度
select lengthb('string') from dual;
```

### 字符串连接函数

``` sql
--字符串链接
select concat('hello','word') from dual;
```

### 字符串分隔函数

``` sql
--字符串分隔函数
select * from table(split('Hello,World',','));                             --将字符串根据,逗号分隔
```

### 字符串裁剪函数

``` sql
--去除子字符串,默认去除空格
--1)从两则去除
select trim('a'from'abcd') from dual;                                      --无效,只能在开头去除
select trim('a'from'abba') from dual;

--2)从左侧去除
select ltrim ('abba' , 'a' ) from dual;
 
--3)从右侧去除       
select rtrim ('abba' , 'a' ) from dual;                                    --注意顺序
 
--4)去除字符串中两侧的空格
select trim(' a b c ') from dual;                                          -- 不传参时,为去除两侧空格
```

### 字符串填充函数

``` sql
--填补字符   被填充字符  填充后总长度(若小于当前长度则表示截取指定长度)  填充的字符,默认为空格填充
--1)从左侧填充
select lpad('hello',8,'*') from dual;                                      --将hello左*填充至8位

--2)从右侧填充
select rpad('hello',8,'*') from dual;

--替换字符串
select replace('abcd','ab','ABC') from dual;                               --替换的个数并不受到影响
select replace('abcd','a') from dual;                                      --不指定替换为什么,默认为删除指定字符
```

### 金融格式化

``` sql
-- 金融格式化
select to_char(123456789.99 , 'FM999,999,999,999,999.99') from dual;
-- 带有补零的
select to_char(123456789.99 , 'FM000,000,000,000,000.00') from dual;
select to_char(123456789.99 , 'FM999,999,999,999,990.00') from dual;
-- 货币显示,本地,美元
select to_char(123456789.99 , 'L999,999,999,999,999.99') from dual;
select to_char(123456789.99 , '$999,999,999,999,999.99') from dual;
```


## 日期函数

### 设置系统默认显示日期格式

``` sql
alter session set nls_date_format = 'yyyy-MM-dd hh24:mi:ss';
select sysdate from dual;
```


### 获取当前时间

``` sql
--获取系统日期  年/月/日 时:分:秒
select sysdate from dual;                                                  

--获取会话的时间的时间戳
select localtimestamp from dual;

--获取会话在当前时区的时间戳
select current_timestamp from dual;

--获取当前数据库时区,会话时区
select dbtimezone ,sessiontimezone from dual;
```



### 日期增减

``` sql
-- 增加天数
select (sysdate+1) from dual;
-- 增加小时
select (sysdate+1/24) from dual;
-- 增加分钟
select (sysdate+1/24/60) from dual;
-- 增加秒钟
select (sysdate+1/24/60/60) from dual;
```

### 月份增减函数

``` sql
--增减月数
select add_months(sysdate,2) from dual;                                    --当前时间中月份加2
select add_months(sysdate,-2) from dual;                                   --当前时间中月份减2
```

### 获取下一周日期

``` sql
--获取下一个星期几的日期
select next_day(sysdate,'星期日') from dual;                               --格式必须为星期几（根据系统环境）
```

### 获取本月最后一天

``` sql
--获取这个月的最后一天
select last_day(sysdate) from dual;
```

### 获取月份间隔

``` sql
--获取两个日期之间相隔的月数                                                 --前一个月份减去后一个月份,为小数
--注意格式,字符串自动转为日期的格式为: 日数-X月-年数
select months_between(sysdate,to_date('2015-05-15','yyyy-mm-dd'))from dual;
select months_between(to_date('2015-05-15','yyyy-mm-dd'),sysdate)from dual;
select months_between('1-10月-14','1-10月-15') from dual;        
```

### 四舍五入日期

``` sql
--四舍五入日期
select round(sysdate) from dual;                                            --获取0点最近的天
select round(sysdate,'day') from dual;                                      --获取最近的星期天
select round(sysdate,'month') from dual;                                    --获取最近的月初
select round(sysdate,'q') from dual;                                        --获取最近的季度
select round(sysdate,'year') from dual;	
```

### 随机日期

``` sql
-- 随机日期
select to_char(sysdate,'J') from dual ;                                    -- 将日期转为日期整数
select to_date(2457947+trunc(dbms_random.value(0,365)),'J') from dual ;
```


### 截取日期

``` sql
-- 截取日期到年度开始
select trunc(sysdate,'YYYY') from dual;
-- 截取日期到月份开始
select trunc(sysdate,'MM') from dual;
-- 截取到本周的开始
select trunc(sysdate,'day')+1 from dual;
-- 截取日期到天开始
select trunc(sysdate,'DD') from dual;
select trunc(sysdate) from dual;
-- 截取日期到小时开始
select trunc(sysdate,'HH') from dual;
select trunc(sysdate,'HH24') from dual;
-- 截取日期到分钟开始
select trunc(sysdate,'MI') from dual;
```

### 获取日期信息

``` sql
--获取日期中的年/月/日
select extract (year from sysdate) from dual;                               --获取年
select extract (month from sysdate) from dual;                              --获取月
select extract (day from sysdate) from dual;                                --获取日
select extract (hour from localtimestamp) from dual;                        --获取时,仅限时间戳类型
select extract (minute from localtimestamp) from dual;                      --获取分,仅限时间戳类型
select extract (second from localtimestamp) from dual;                      --获取秒,仅限时间戳类型
```

示例,获取两个日期间的间隔分钟
``` sql
-- 两个日期分钟间隔 , 不连续
select 
  extract (MINUTE from to_timestamp('2018-1-5 00:10:00' ,'yyyy-MM-dd hh24:mi:ss') 
                       - to_timestamp('2018-1-1 00:00:00' ,'yyyy-MM-dd hh24:mi:ss'))
from dual;
```



### 字符串日期转换

``` sql
--字符转为日期
select to_date('2015-1-1','yyyy-mm-dd') from dual;                          --以系统日期格式保存,取出时任然需要格式转化

--日期转换为字符
select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') from dual;                  --获取全部的日期,时刻 格式
select to_char(sysdate,'yyyy-mm-dd') from dual;                             --获取日期
select to_char(sysdate,'mm-dd') from dual;                                  --获取感兴趣的部分        
select to_char(sysdate,'day') from dual;                                    --获得星期
select to_char(sysdate,'AM') from dual;                                     --获得上午|下午时间段
select to_char(sysdate,'ddd') from dual;                                    --获得本年的第几天
select to_char(sysdate,'dl') from dual;                                     --获得日期和星期          2017年7月12日 星期三
select to_char(sysdate,'ds') from dual;                                     --获得日期               2017-07-12
select to_char(sysdate,'iw') from dual;                                     --获得本年度第几周
select to_char(sysdate,'q') from dual;                                      --获得本年度第几季度
select to_char(sysdate,'j') from dual;  
select to_char(sysdate) from dual;                                          --默认格式: 日-X月-年      14-9月-16
```

## 聚合函数

### 平均值函数

``` sql
--平均值
select avg(sal) from emp;
```

### 求和函数

``` sql
--总值
select sum(sal)  from emp;
```

### 极大值函数

``` sql
--极大值
select max(sal) from emp;
```

### 极小值函数

``` sql
--极小值                    
select min(sal) from emp;                                                   -- 不包括空,空值过滤
```

### 标准差函数

``` sql
--标准差
select stddev(sal) from emp;
select stddev(distinct sal) from emp;
```

### 方差函数

``` sql
--方差
select variance(sal) from emp;
select variance(distinct sal) from emp;
```

### 统计函数

`count`一定会有的返回值
``` sql
--计数
select count(*) from emp;
select count(distinct emp.job) from emp;                                    -- 必须指定具体的字段,去重输出
select count(*) , count(empno) , count(comm) , count(distinct job) from emp;
```

## 位置函数
### 相对位置函数

`cume_dist()` 函数,计算一行在分区的相对位置,例如,分区有五条记录,则按照 1 , 0.8 , 0.6, 0.4 , 0.2进行划分
``` sql
select deptno , ename, sal,
    cume_dist() over(partition by deptno order by sal) cume
from emp
```

### 等分函数
`ntile()`函数针对数据分区中的有序集合进行划分,类似于分成等份
``` sql
select deptno , sal, 
  sum(sal) over (partition by deptno order by sal) dept_sum_sal,
  ntile(3) over (partition by deptno order by sal) dept_ntile_sal_3,
  ntile(6) over (partition by deptno order by sal) dept_ntile_sal_6
from emp
```

### 比例函数
`ratio_to_report()` 计算该行所在比例
``` sql
select deptno , sum(sal),
  ratio_to_report(sum(sal)) over () rate
from emp

```

## 分析函数
分许函数的语法如下:
```
函数名([参数,...]) over (
partition by 子句 字段, ...
[order by 子句 字段, ... [asc | desc | nulls first | nulls last]]
[windowing 子句]);
```
其中:
over子句: 为分析函数指明一个查询结果集,此语句在select子句之中使用
partition by子句:将一个简单的集合分为N区,而后按照不同的区对数据进行统计
order by子句:指明数据在区中的排列方式
  nulls first | nulls last 返回数据列中,包含NULL值是出现在排序序列前还是尾
windowing子句:给出变化固定的数据窗口的方法,分析函数将对此数据进行操作


### 常见组合
第一种   函数名称(参数,...) over (partition by 子句 , order by 子句, windowing子句);
第二种   函数名称(参数,...) over (partition by 子句 , order by 子句);
第三种   函数名称(参数,...) over (partition by 子句);
第四种   函数名称(参数,...) over (                   order by 子句, windowing子句);
第五种   函数名称(参数,...) over (                   order by 子句              );
第六种   函数名称(参数,...) over (                                             );

### over子句
```sql
-- 根据查询结果,进行求和
select deptno , ename , sal ,
  sum(sal) over () sum_sal
from emp;
-- 根据查询结果,随后进行分区求和
select deptno , ename , sal ,
  sum(sal) over (partition by deptno) sum_sal
from emp;
-- 根据查询结果,随后进行多条件分区,求和
select deptno , ename , sal , job ,
  sum(sal) over (partition by deptno , job) sum_sal
from emp;
-- 根据查询结果,随后进行多条件分区,求和,随后排序 rank()产生序列,有名次并列
select deptno , ename , sal ,
  rank() over (partition by deptno  order by sal desc , ename asc) rk
from emp;
-- nulls first|last 当为空时,null所在记录排在最前面或者最后面,默认空放在第一位
select deptno , ename , sal , comm , 
  rank() over (order by comm desc nulls last) rk,
  sum(sal) over (order by comm desc nulls last) sum_sal
from emp;
```
### window 分窗子句
分窗子句主要是用于定义一个变化的或者固定的数据窗口方法,主要用于定义分析函数在操作行的集合

分窗子句有两种实现方式:
- 实现一: 值域窗(range window),逻辑便宜.当前分区之中前行的N行到当前行的记录集
-  实现二: 行窗(rows window),物理便宜.以排序的结果顺序计算偏移当前行的起始行记录集

如果想要指定range或rows的偏移量,则可以采用如下的几种排序列:

``` sql
range | rows 数字 preceding
range | rows between unbounded preceding and current row
range | rows between current row and unbounded following
```

以上几种排序之中包含的概念:
preceding 主要设置一个偏移量,可以为数字或者为其他
between...and.. 偏移量范围
unbounded preceding 不限制偏移量大小
current row 当前行
following 如果不写,表示使用N行与当前行指定数据比较,如果编写次语句,表示当前行与下N行数据比较

**示例**
``` sql
-- 验证range  根据部门进行分区,同时按照部门内工资升序排序,sum_sal对当前行之前的记录,如果不超过300则进行累计求和,否则重新求和
select deptno, ename, sal,
  sum(sal) over (partition by deptno order by sal
                 range 300 preceding) sum_sal
from emp;                 

-- 使用向下匹配,向下求和,注意相同值的匹配
select deptno, ename, sal,
  sum(sal) over (partition by deptno order by sal
                 range between 0 preceding and 300 following) sum_sal
from emp;                 

-- 匹配当前行,没有偏移量进行求和
select deptno, ename, sal,
  sum(sal) over (partition by deptno order by sal
                 range between 0 preceding and current row) sum_sal
from emp;        

-- 不设置边界,不限制偏移量,注意相同值的数据
select deptno, ename, sal,
  sum(sal) over (partition by deptno order by sal
                 range between unbounded preceding and current row) sum_sal
from emp;        

-- 物理偏移,当前行等于前两行的求和
select deptno, ename, sal,
  sum(sal) over (partition by deptno order by sal
                 rows 2 preceding) sum_sal
from emp;

-- 设置查询行的范围,不设置下限,并采用下行比较方式.等于对当前分区内所有薪资求和
select deptno, ename, sal,
  sum(sal) over (partition by deptno order by sal
                 rows between unbounded preceding and unbounded following) sum_sal
from emp;
```

### 分区统计函数
用于统计分区后各区的数据,常用语法
``` sql
sum([distinct | all] 表达式)         计算分区中的数据累加和
min([distinct | all] 表达式)         查找分区中的最小值
max([distinct | all] 表达式)         查找分区中的最大值
avg([distinct | all] 表达式)         计算分区中的数据平均值
count(* | [distinct | all] 表达式)    计算分区中的数据量
```
**示例**
```sql
--查询雇员为7369的雇员名称,职位,基本工资,部门编号,部门人数,部门平均工资,最低工资,最高工资,总工资
select e.empno, e.ename, e.job, e.sal, e.deptno,
    sum(e.empno) over (partition by e.deptno) dept_emp_count,
    avg(e.sal) over (partition by deptno) dept_sal_ave,
    min(e.sal) over (partition by deptno) dept_sal_min,
    max(e.sal) over (partition by deptno) dept_sal_max,
    sum(e.sal) over (partition by deptno) dept_sal_sum
from emp e
where e.empno = 7369;

--查询员工的编号,姓名,基本工资,部门,此部门的平均工资,最低和最高工资
select e.empno, e.ename, e.sal, d.dname,
    avg(e.sal) over (partition by e.deptno order by e.sal
                     range between unbounded preceding and unbounded following) dept_sal_ave,
    min(e.sal) over (partition by e.deptno order by e.sal
                     range between unbounded preceding and unbounded following) dept_sal_min,
    max(e.sal) over (partition by e.deptno order by e.sal
                     range between unbounded preceding and unbounded following) dept_sal_max,                 
    sum(e.sal) over (partition by e.deptno order by e.sal
                     range between unbounded preceding and unbounded following) dept_sal_sum
from emp e , dept d
where e.deptno = d.deptno
```
### 等级函数
多用于生成序列,进行排序展示等操作

|   方法  |  说明    |
|---|---|
|	rank()	|          根据order by子句的排序字段,从分区查询一行数据,按照降序生成序号,同分名次相同,下一名次不存在,出现跳号	|
|	dense_rank()	|    根据order by子句的排序字段,从分区查询一行数据,按照降序生成序号,同分名次相同,下一名次顺序排列,不会出现跳号	|
|	first	|           取出dense_rank()的第一行数据	|
|	last   |        取出dense_rank()的最后一行数据	|
|	first_value(列)	|	 取出分区的第一个值	|
|	last_value(列)	|  取出分区的最后一个值	|
|	lag(列名称\[,行数字]\[,默认值])	|  访问分区中指定的前N行记录,如果没有则返回默认值	|
|	lead(列名称\[,行数字]\[,默认值])	| 访问分区中指定的后N行记录,如果没有则返回默认值	|
|	row_number  |    返回每组中的行号	|

**示例**
```sql
-- TOP-N 问题,并列,后取消下一名次
select rownum , rank() over (order by sal asc) rank, 
e.* from emp e;
-- 并列,则顺序排序,不出现跳号
select rownum , dense_rank() over (order by sal asc) dense_rank, 
e.* from emp e;

-- 重新生成rownum,避免分组后紊乱,表示分组中当前行
select rownum , row_number() over (order by sal asc) row_number, 
e.* from emp e;
```

### keep语句
keep语句保留满足条件的数据
在使用dense_rank确定后的集合才可以使用first或者last取得集合中的数据
语法为:

``` sql
分组函数() keep (dense_rank first | last order by 表达式[asc | desc | nulls [first | last]],...) [over() 分区查询]
```

**示例**

``` sql
--first | last 查询每个部门的最高及最低工资
select deptno , 
max(sal) keep (dense_rank first order by sal desc) dept_sal_max,
min(sal) keep (dense_rank last order by sal desc) dept_sal_min
from emp
group by deptno;
```

### 首尾函数

over()声明一个数据集合,而利用first_value()或者last_value()函数取得集合中的首行和尾行中的字段值
``` sql
select deptno , empno , sal,
    -- 查找部门最高工资
    first_value(sal) over (partition by deptno order by sal
                           range between unbounded preceding and unbounded following) sal_first_value,
    last_value(sal) over (partition by deptno order by sal
                           range between unbounded preceding and unbounded following) sal_last_value
from emp
```

### 相邻记录

在满足于一定顺序的基础上,对前|后偏移量的数据进行重现
`lag(字段,偏移量,默认值)`   查找前X行字段数据,没有则返回默认值
`lead(字段,偏移量,默认值)`  查找后X行字段数据,没有则返回默认值

``` sql
-- 相邻记录
select deptno, empno , ename, sal,
    lag(sal,2,null) over (partition by deptno order by sal) dept_lag_sal,
    lead(sal,2,null) over (partition by deptno order by sal) dept_lead_sal
from emp
where deptno = 20;
```

### 报表函数
`cume_dist()`                   计算一行在分区中的相对位置
`ntile(数字)`                    将一个分区分为"表达式"的散列表示
`ratio_to_report(表达式)` 改函数计算表达式的值,它给出相对于总数的百分比

``` sql
-- cume_dist() 函数,计算一行在分区的相对位置,例如,分区有五条记录,则按照 1 , 0.8 , 0.6, 0.4 , 0.2进行划分
select deptno , ename, sal,
    cume_dist() over(partition by deptno order by sal) cume
from emp

-- ntile()函数针对数据分区中的有序集合进行划分,类似于分成= = 份
select deptno , sal, 
  sum(sal) over (partition by deptno order by sal) dept_sum_sal,
  ntile(3) over (partition by deptno order by sal) dept_ntile_sal_3,
  ntile(6) over (partition by deptno order by sal) dept_ntile_sal_6
from emp

-- ratio_to_report() 计算该行所在比例
select deptno , sum(sal),
  ratio_to_report(sum(sal)) over () rate
from emp
group by deptno
```
