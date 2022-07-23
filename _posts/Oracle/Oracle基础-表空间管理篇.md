---
title: Oracle基础-表空间管理篇
abbrlink: b5ec7327
date: 2017-09-23 12:52:34
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

> Oracle 表结构管理.

<!--more-->



## 内容简介

对Oracle数据库中表空间的初始进行定义，涉及到对新用户表空间分配及扩展文件夹的创建

## 用户创建及授权

`sys` 			最高管理员		  维护 数据字典,表信息.列信息,动态视图
`system` 	  默认系统管理员  其他一般维护



### 命令行下远程连接数据库

- DOS下连接Oracle
``` sql
sqlplus jion/123456;                                                   -- 普通用户登录
sqlplus system/123456 as sysdba;                                            -- DB管理员登录
```

- 退出命令
``` sql
quit;                                                                       -- 退出命令
```
### 创建用户

``` sql
create user jion                                                       -- 用户名
identified by 123456                                                        -- 密码
default tablespace jion                                                -- 默认表空间名,默认表空间名称与用户名一致
temporary tablespace TEMP                                                   -- 临时表空间名,关闭后数据销毁
profile DEFAULT;                                                            -- 配置方式,默认配置
```
### 创建数据库文件夹及授权用户
- 创建数据库文件夹，存放与数据库相关资料
``` sql
create or replace directory sysload_file_dir                                -- 文件夹名称
as 'F:\ORACLE_Workspace\database';                                          -- 文件夹路径 
```

- 授权用户使用数据库文件夹
``` sql
grant read,write                                                            -- 授权:读,写权限
on directory sysload_file_dir                                               -- 被授权文件夹的名称
to jion;                                                               -- 授权用户
```

## 默认表空间及用户位置
- 查看默认表空间及临时表空间
``` sql
select default_tablespace,temporary_tablespace
from dba_users                                                              -- 管理员字典
where username='SYSTEM';                                                    -- STSTEM用户(大写)
```

- 用户字典
``` sql
select default_tablespace,temporary_tablespace                             
from user_users                                                             -- 用户字段
where username='SYSTEM';                                                    -- STSTEM用户(大写)
```

- 查看用户DB文件位置（BD管理员可见）
``` sql
select name from v$datafile;
```

- 查看用户的schema,默认为用户名
``` sql
select sys_context('userenv','current_schema') as current_schema from dual;
```
## 创建及修改表空间
- 创建永久表空间
``` sql
create tablespace  jion                                                -- 用户名(默认与表空间名一致)
datafile 'F:\ORACLE_Workspace\database\zhangsan.dbf'                        -- 存放路径
size 10 M                                                                   -- 文件大小
autoextend on                                                               -- 开启自动扩展储存空间
next 64 m maxsize unlimited                                                 -- 储存不够时,自动扩展64M,上不设限
extent management local;                                                    -- 本地管理员
```

- 创建临时表空间
``` sql
create temporary tablespace jion                                       -- 用户名(默认与表空间名一致)
tempfile 'F:\ORACLE_Workspace\database\temp\zhangsan.dbf'                   -- 存放路径
size 10 M;                                                                  -- 默认大小
```

- 修改默认表空间
``` sql
alter user SYSTEM
default tablespace system;
```

- 修改临时表空间
``` sql
alter user SYSTEM
temporary tablespace temp;
```

- 查找永久表空间中的文件信息
``` sql
select * from dba_data_files                                                -- 数据库文件描述表
where tablespace_name='jion';                                          -- 与表空间名称一致
```

- 查找临时表空间中的文件信息
``` sql
select * from dba_temp_files                                                -- 表空间名与用户名默认一致
where tablespace_name='jion';             
 
```

- 修改表空间在线状态

``` sql
alter tablespace jion offline;		--脱机                                         
alter tablespace jion online;		--在线
```

- 查看表空间状态
``` sql
select status                                                               -- 字段,表空间的状态
from dba_tablespaces                                                        -- 所有数据库中表空间的状态信息
where tablespace_name = 'ZHANGSAN';                                         -- 表空间的名字(默认与用户名一致)
```

- 修改表空间为只读
``` sql
alter tablespace jion                                                  -- 表空间的名字(默认与用户名一致)
read only ;                                                                 -- 只读
```

- 修改表空间为读写
``` sql
alter tablespace jion
read write;                                                                 -- 读写
```

- 为表空间增加数据文件
``` sql
alter tablespace jion      
add datafile 'F:\ORACLE_Workspace\database\zhangsan_new.dbf'
size 1 M;
```

- 删除表空间
``` sql
alter tablespace  jion
drop datafile 'F:\ORACLE_Workspace\database\zhangsan_new.dbf';
```

- 级联删除表空间
``` sql
drop tablespace jion
including contents;
```



