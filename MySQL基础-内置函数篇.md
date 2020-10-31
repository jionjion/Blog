---
title: MySQL基础-内置函数篇
abbrlink: 274a540a
date: 2020-06-14 11:03:26
categories:
  - MySQL
tags: [MySQL, SQL]
---

## 简介

介绍了 `MySQL`  相关内置函数. 数据库采用 `8.X` 版本


## 数值函数
### 四舍五入函数

``` mysql
-- 1)小数点后取舍   
select round(5555.5555,2)   from dual;                                    -- 返回5555.56                    
select round(5555.5555)     from dual;                                    -- 返回5556
select round(-5555.5555,2)  from dual;                                    -- 返回-5555.56                    
select round(-5555.5555)    from dual;                                    -- 返回-5556
-- 2)小数点前取舍   
select round(5555.5555,-2)  from dual;                                    -- 返回5600
select round(5555.5555,-6)  from dual;                                    -- 返回0
select round(-5555.5555,-2) from dual;                                    -- 返回-5600
select round(-5555.5555,-6) from dual;                                    -- 返回0
```

### 取整函数

``` mysql
-- 1)正无穷方向取整
select ceil(1.5)  from dual;                                               -- 返回2
select ceil(-1.5) from dual;                                               -- 返回-1
-- 2)负无穷方向取整
select floor(1.5) from dual;                                               -- 返回1
select floor(-1.5)from dual;                                               -- 返回-2
```

### 正负号函数

``` mysql
select sign(100),sign(-100),sign(0) from dual;                             -- 正值返回1，负值返回-1，0返回0
```

### 绝对值函数


``` mysql
select abs(2.5) from dual;
select abs(-2.5)from dual;                                                 -- 返回2.5
```

### 取余函数

``` mysql
select mod(5,2) from dual;                                                 -- 5%2为1
```

### 幂函数

``` mysql
select power(3,2) from dual;                                               -- 3的2次方
select exp(1)     from dual;                                               -- 自然对数e的0次幂,1次幂,2次幂
```

### 对数函数

``` mysql
select log(2,4)  from dual;                                                -- log以2为底,4的对数
select ln(2.714) from dual;                                                -- log以e为底,2.714的对数
```

### 平方根函数

``` mysql
select sqrt(4) from dual;                                                  -- 2
```

### 截取任意精度

``` mysql
select truncate(12345.6789,2)   from dual;                                 -- 正数:小数点后保留位数 
select truncate(12345.6789,-2)  from dual;                                 -- 负数:小数点前元整位数 
select truncate(12345.6789,0)   from dual;                                 -- 0:截取整数
```

### 随机数函数

``` mysql
select rand()              from dual;                                      -- 查询[0, 1)的随机小数
select rand() * 100        from dual;                                      -- 查询[0, 100)的随机小数
select floor(rand() * 100) from dual;                                      -- 查询[0, 100)的随机整数
```

### 三角函数

``` mysql
-- 三角函数(弧度制)
select sin(3.1415926/2) from dual;
select cos(3.1415926/2) from dual;
select tan(3.1415926/2) from dual;
select asin(0.5) from dual;
select acos(0.5) from dual;
select atan(0.5) from dual;
```

## 字符函数

### ASCII转换函数
``` mysql
-- 查看字母的ASCII值
select ascii('A'), ascii(1), ascii('囧') from dual;                        -- 字母和数字智能填一个,多填以第一个为主
-- 将ASCII值转为字符
select char(65), char(49), char(50403) from dual;                          -- 注意数字的位数
```
### 大小写转换函数

``` mysql
-- 小写转大写
select upper ('hello') from dual;                                      
-- 大写转小写
select lower ('HELLO') from dual; 
```

### 字符索引函数
单行函数,查找字符串索引位置,索引从1开始,不存在,返回0 
``` mysql
select instr('a b c d e' , 'a') from dual;
select instr('a b c d e' , 'c') from dual;
select instr('a b c d e' , 'f') from dual;
```

### 字符串截取函数

``` mysql
-- 1)从开始索引到最后
select substr('string',3) from dual;                                      -- 返回 ring ,在字符串位置开始处为1
-- 2)从开始索引获取后面指定长度的元素
select substr('string',3,2) from dual;                                    -- 返回 ri
-- 3)从结尾索引到最后
select substr('string',-3) from dual;                                     -- 返回 ing
-- 4)从结尾索引获取到后面指定长度的元素
select substr('string',-3,2)from dual;                                    -- 返回 in
-- !)从结尾索引获取到前面指定长度的元素
select substr('string',-3,-2)from dual;                                   -- 没有这个用法
```

### 字符串长度函数

``` mysql
-- 获取字符串长度
select length('string') from dual;
```

### 字符串连接函数

``` mysql
-- 字符串链接
select concat('hello','word') from dual;
```

### 字符串裁剪函数

``` mysql
-- 从两则去除指定字符串
select trim('a'from'abcd') from dual;                                      -- 只能在开头去除
select trim('a'from'abba') from dual;                                      -- 两侧被去除
-- 除空格
select trim('  a  ') from dual;
-- 从左侧去除
select ltrim ('  a') from dual;
-- 从右侧去除       
select rtrim ('a  ') from dual; 
```

### 字符串填充函数

``` mysql
-- 填补字符   被填充字符  填充后总长度(若小于当前长度则表示截取指定长度)  填充的字符,默认为空格填充
-- 1)从左侧填充
select lpad('hello',8,'*') from dual;                                      -- 将hello左*填充至8位
-- 2)从右侧填充
select rpad('hello',8,'*') from dual;
```

### 字符串替换函数

```mysql
-- 全部替换
select replace('abcd','ab','ABC') from dual;
```

### 金融格式化

``` mysql
select format(1000.2189,2)               from dual;                        -- 1,000.22
select convert(10000.2189,decimal(12,2)) from dual;                        -- 10000.22 精度12位,小数位2位
```

### 字符串MD5

```mysql
select md5('hello')          from dual;
select md5(unix_timestamp()) from dual;                                    -- 时间戳种子MD5
select md5(rand() * 10000)   from dual;                                    -- 随机数种子MD5
```

## 日期时间函数


### 当前日期时间

``` mysql
-- 获得数据库所在时区的时间戳 2020-06-14 15:17:33
select localtimestamp                   from dual;
-- 获取会话在当前时区的时间戳 2020-06-14 15:17:33
select current_timestamp                from dual;
-- 获得当前系统时间 2020-06-14 15:18:15
select now()                            from dual;
-- 当前日期
select curdate()                        from dual;
-- 当前时间
select curtime()                        from dual;
-- 获得unix当前时区时间戳 1592119124
select unix_timestamp()                 from dual;
-- 将时间戳转为日期
select from_unixtime(unix_timestamp())  from dual;
```

### 日期增减函数

可以使用日期变量,或者标准日期格式进行日期的加减

``` mysql
-- 年
select date_add('2020-01-01', interval 1 year) from dual;
select date_add(now(), interval 1 year)        from dual;
-- 季
select date_add(now(), interval 1 quarter)     from dual;
-- 月
select date_add(now(), interval 1 month)       from dual;
-- 周
select date_add(now(), interval 1 week)        from dual;
-- 天
select date_add(now(), interval 1 day)         from dual;
-- 时
select date_add(now(), interval 1 hour)        from dual;
-- 分
select date_add(now(), interval 1 minute)      from dual;
-- 秒
select date_add(now(), interval 1 second)      from dual;
-- 毫秒
select date_add(now(), interval 1 microsecond) from dual;
```

### 日期间隔函数

``` mysql
-- 两个日期间的天数差
select datediff('2020-01-02', '2020-01-01') from dual;
select datediff(now(),now()) from dual;
-- 两个时间间的时间差
select timediff(now(), now)            from dual;
select timediff('01:00:00','00:00:00') from dual;
-- 两个日期间的小时差
select timediff('2020-02-01 12:30:00', '2020-01-01 00:00:00') from dual;   -- 756:30:00  
```

### 日期信息

``` mysql
-- 年
select extract(year from now())        from dual;
-- 季
select extract(quarter from now())     from dual;
-- 月
select extract(month from now())       from dual;
-- 周
select extract(week from now())        from dual;
-- 天
select extract(day from now())         from dual;
-- 时
select extract(hour from now())        from dual;
-- 分
select extract(minute from now())      from dual;
-- 秒
select extract(second from now())      from dual;
-- 毫秒
select extract(microsecond from now()) from dual;
```

### 日期格式化

``` mysql
-- 年,4位
select date_format(now(), '%Y') from dual;
-- 年,2位
select date_format(now(), '%y') from dual;
-- 年,其中的星期日是周的第一天，4 位，与 %V 使用
select date_format(now(), '%X') from dual;
-- 年,其中的星期一是周的第一天，4 位，与 %v 使用
select date_format(now(), '%x') from dual;
-- 本年第N天 (001-366)
select date_format(now(), '%j') from dual;

-- 月名,英文单词
select date_format(now(), '%M') from dual;
-- 缩写月名
select date_format(now(), '%b') from dual;
-- 月,数值(00-12)
select date_format(now(), '%c') from dual;
-- 月,数值(00-12)
select date_format(now(), '%m') from dual;
-- 带有英文后缀的,本月第N天
select date_format(now(), '%D') from dual;
-- 本月第N天,数值(00-31),有前导零
select date_format(now(), '%d') from dual;
-- 本月第N天,数值(0-31),无前导零
select date_format(now(), '%e') from dual;

-- 星期名
select date_format(now(), '%W') from dual;
-- 缩写星期名
select date_format(now(), '%a') from dual;
-- 本周第N天 （0=星期日, 6=星期六）
select date_format(now(), '%w') from dual;
-- 本年第N周 (00-53) 星期一是一周的第一天
select date_format(now(), '%u') from dual;
-- 本年第N周 (00-53),星期日是一周的第一天
select date_format(now(), '%U') from dual;
-- 周 (01-53) 星期日是一周的第一天，与 %X 使用
select date_format(now(), '%V') from dual;
-- 周 (01-53) 星期一是一周的第一天，与 %x 使用
select date_format(now(), '%v') from dual;

-- 小时,数值(00-23),二十四小时制,有前导零
select date_format(now(), '%H') from dual;
-- 小时,数值(00-23),二十四小时制,没有前导零
select date_format(now(), '%k') from dual;
-- 小时,数值(01-12),十二小时制,有前导零
select date_format(now(), '%h') from dual;
select date_format(now(), '%I') from dual;
-- AM,PM
select date_format(now(), '%p') from dual;
-- %r	时间, 12-小时[hh:mm:ss AM 或 PM]
select date_format(now(), '%r') from dual;
-- 小时,数值(01-12),十二小时制,没有前导零
select date_format(now(), '%l') from dual;

-- 分钟,数值(00-59)
select date_format(now(), '%i') from dual;

-- 秒(00-59),有前导零
select date_format(now(), '%S') from dual;
select date_format(now(), '%s') from dual;
-- 微秒
select date_format(now(), '%f') from dual;
```

### 字符与日期

```mysql
-- 日期转为字符串
select date_format(now(), '%T') from dual;                                 -- 16:52:20
select date_format(now(), '%Y-%m-%d %H:%i:%s') from dual;                  -- 2020-06-14 16:52:31

-- 字符串转为日期
select str_to_date('2020-01-01 07:30:00', '%Y-%m-%d %H:%i:%s') from dual;
```



## 聚合函数

### 平均值函数

``` mysql
-- 平均值
select avg(sal) from emp;
```

### 求和函数

``` mysql
-- 总值
select sum(sal)  from emp;
```

### 极大值函数

``` mysql
-- 极大值
select max(sal) from emp;
```

### 极小值函数

``` mysql
-- 极小值                    
select min(sal) from emp;                                                  -- 不包括空,空值过滤
```

### 标准差函数

``` mysql
-- 标准差
select stddev(sal) from emp;
```

### 方差函数

``` mysql
-- 方差
select variance(sal) from emp;
```

### 统计函数

`count`  未查询数据,返回0返回值

``` mysql
-- 计数
select count(*) from emp;
select count(distinct emp.job) from emp;                                   -- 必须指定具体的字段,去重输出
select count(*) , count(empno) , count(comm) , count(distinct job) from emp;
```



## 查询语句

### 分页查询

```mysql
-- 查询前五条数据,从0开始,查询5条
select * from emp limit 0, 5
```

