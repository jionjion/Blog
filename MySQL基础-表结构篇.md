---
title: MySQL基础-表结构篇
abbrlink: d3ab4f85
date: 2020-06-14 11:06:15
categories:
tags:
---



## 简介

介绍 `Mysql` 数据库相关表结构



## 数据类型


### 数值类型

| 类型              | 占用大小(字节 byte) | 范围              | 用途 | `java.sql.JDBCType` 类型 |
| :---------------- | :------------: | :---------------: | ---- | ------------------------ |
| `tinyint`         | 1              | [-2^7^, 2^7^-1]   | 小整数 | `java.sql.Types#TINYINT` |
| `smallint`        | 2              | [-2^15^, 2^15^-1] | 小整数 | `java.sql.Types#SMALLINT` |
| `mediumint`       | 3              | [-2^23^, 2^23^-1] | 小整数 |  |
| `int` / `integer` | 4              | [-2^31^, 2^31^-1]   | 整数 | `java.sql.Types#INTEGER` |
| `bigint`          | 8              | [-2^63^, 2^63^-1]   | 大整数 | `java.sql.Types#BIGINT` |
| `float` | 4 | - | 单精度浮点数 | `java.sql.Types#FLOAT` |
| `real` | 4 | - | 单精度浮点数 | `java.sql.Types#REAL` |
| `double`          | 8              | -  | 双精度浮点数 | `java.sql.Types#DOUBLE` |
| `decimal(M,D)`    | Max[M\|D] + 2  | -                 | 小数值 | `java.sql.Types#DECIMAL`  |
| `numeric(M,D)` | Max[M\|D] + 2 | - | 小数值 | `java.sql.Types#NUMERIC` |

### 字符类型

| 类型         | 占用最大(字节 byte) | 用途         | `java.sql.JDBCType` 类型     |
| ------------ | :-----------------: | ------------ | ---------------------------- |
| `char`       |      2^8^ - 1       | 短定长字符串 | `java.sql.Types#CHAR`        |
| `varchar`    |      2^16^ - 1      | 变长字符串   | `java.sql.Types#VARCHAR`     |
| `tinytext`   |      2^8^ - 1       | 短文本       |                              |
| `text`       |      2^16^ - 1      | 文本         | `java.sql.Types#CLOB`        |
| `mediumtext` |      2^24^ - 1      | 中等文本     |                              |
| `longtext`   |      2^32^ - 1      | 大文本       | `java.sql.Types#LONGVARCHAR` |

### 二进制类型

| 类型         | 占用最大(字节 byte) |    用途    | `java.sql.JDBCType` 类型       |
| ------------ | :-----------------: | :--------: | ------------------------------ |
| `bit`        |         1/8         |   布尔值   | `java.sql.Types#BIT`           |
| `binary`     |      2^8^ - 1       | 定长二进制 | `java.sql.Types#BINARY`        |
| `varbinary`  |      2^16^ - 1      | 变长二进制 | `java.sql.Types#VARBINARY`     |
| `tinyblob`   |      2^8^ - 1       |  短二进制  | `java.sql.Types#LONGVARBINARY` |
| `blob`       |      2^16^ - 1      |   二进制   | `java.sql.Types#BLOB`          |
| `mediumblob` |      2^24^ - 1      | 中等二进制 |                                |
| `longblob`   |      2^32^ - 1      |  大二进制  |                                |


### 日期类型

| 类型        | 占用最大(字节 byte) |         格式          | 用途     | `java.sql.JDBCType` 类型   |
| ----------- | :-----------------: | :-------------------: | -------- | -------------------------- |
| `date`      |          3          |     `YYYY-MM-DD`      | 日期     | `java.sql.Types#DATE`      |
| `time`      |          3          |      `HH:MM:SS`       | 时间     | `java.sql.Types#TIME`      |
| `year`      |          1          |        `YYYY`         | 年份     |                            |
| `datetime`  |          8          | `YYYY-MM-DD HH:MM:SS` | 日期时间 |                            |
| `timestamp` |          4          |   `YYYYMMDD HHMMSS`   | 时间戳   | `java.sql.Types#TIMESTAMP` |



### 更多数据类型

- `enum` 枚举
- `set` 集合
- `linestring` 地址坐标
- `multilinestring` 地理坐标
- `point` 地理坐标
- `multipoint` 地理坐标
- `polygon` 地理坐标
- `multipolygon` 地理坐标
- `geometry` 地理坐标
- `geometrycollection` 地理坐标