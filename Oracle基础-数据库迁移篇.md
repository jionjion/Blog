---
title: Oracle基础-数据库迁移篇
abbrlink: 3e54b8ba
date: 2017-09-23 12:52:55
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---
> Oracle 数据库迁移

<!--more-->



## 内容简介

Oracle数据库中表空间导出命令,限于当前用户有相应权限,并在命令行窗口下执行,注意结尾处没有分号,并删除注释.操作时避免相关用户登录或者高并发访问

### 数据导出命令

``` sql       
expdp system/123456                                                         -- 导出数据库账号/密码
schemas=system                                                              -- 数据库数据段名称
directory=sysload_file_dir                                                  -- 导出的位置,为数据库使用文件夹的名称,之前已由db定义
dumpfile=system_2017.dmp                                                    -- 导出的文件名称
logfile=system_2017.log                                                     -- 生成的日志名称,结尾处不可以有分号.
```

### 数据导入命令

``` sql
--数据库导入命令
impdp system/123456                                                         -- 导入数据库账号/密码
dumpfile=system_2017.dmp                                                    -- 导入文件名
directory=sysload_file_dir                                                  -- 导入文件的路径,为数据库使用文件夹的名称,之前已由db定义
logfile=system_2017.log                                                     -- 输出日志格式
remap_schema=system:system                                                  -- 数据段更换,system->system.不支持db管理员的重新映射!!!
remap_tablespace=system:system                                              -- 表空间更换,system->system.不支持DB管理员的重新映射!!!
```
