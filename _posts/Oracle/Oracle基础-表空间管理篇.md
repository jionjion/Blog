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

### 分类

- 临时表空间
    用来存放用户临时数据,需要时被覆盖,关闭数据后自动删除,不存放永久数据.
    例如,当排序时会在GPA中进行,当数据过多时,内存不足,将排序数据分为多份,每次取一份进行GPA排序,其他放到临时表空间中.如此循环.
- 临时表空间组
    由一组临时表空间组成的组,不能与临时表空间同名.不能显示地创建和删除;当把第一个临时表空间分配给某个临时表空间组时,会自动创建临时表空间组;最后一个临时表空间删除时,自动删除临时表空间组.

### 查询

- 查看默认表空间及临时表空间
``` sql
select default_tablespace,temporary_tablespace
from dba_users                                                              -- 管理员字典
where username='SYSTEM';                                                    -- STSTEM用户(大写)

select * from v$tablespace;													-- 查看当前表空间
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
create tablespace  jion                                                		-- 用户名(默认与表空间名一致)
datafile 'F:\ORACLE_Workspace\database\zhangsan.dbf'                        -- 存放路径
size 10 M                                                                   -- 文件大小
autoextend on                                                               -- 开启自动扩展储存空间
next 64 m maxsize unlimited                                                 -- 储存不够时,自动扩展64M,上不设限
extent management local;                                                    -- 本地管理员
```

- 创建临时表空间
``` sql
create temporary tablespace jion                                       		-- 用户名(默认与表空间名一致)
tempfile 'F:\ORACLE_Workspace\database\temp\zhangsan.dbf'                   -- 存放路径
size 10 M                                                                   -- 默认大小
autoextend on;																-- 自动扩展
```

- 修改默认表空间
``` sql
alter user SYSTEM default tablespace system;
```

- 修改临时表空间
``` sql
alter user SYSTEM temporary tablespace temp;

alter tablespace temp02 
add tempfile 'E:\ORACLE\ORADATA\ORCL\TEMP02b.DBF' size 10M autoextend on; 	-- 为临时表空间添加文件
```

- 创建临时表空间组

```sql
create temporary tablespace temp03 tempfile									-- 创建临时表空间,并分配临时表空间组
'E:\ORACLE\ORADATA\ORCL\TEMP03.DBF' 
size 10M 
autoextend on
tablespace group temp_grp;

alter database default temporary tablespace temp_grp;						-- 修改系统临时表空间为一个组

alter database default temporary tablespace temp02;							-- 修改系统临时表空间为一个临时表空间
```



- 修改临时表空间组

```sql
alter tablespace temp02 tablespace group temp_grp;							-- 为临时表空间分配临时表空间组
alter tablespace temp02 tablespace group '';								-- 从临时表空间组中,去除某表空间
```



- 查找永久表空间中的文件信息

``` sql
select * from dba_data_files                                                -- 数据库文件描述表
where tablespace_name='jion';                                         	 	-- 与表空间名称一致
```

- 查找临时表空间中的文件信息
``` sql
select * from dba_temp_files                                                -- 表空间名与用户名默认一致
where tablespace_name='jion';

select * from v$tempfile;													-- 临时表空间

select * from dba_tablespace_groups;										-- 查看临时表空间组

select * from database_properties t where t.PROPERTY_NAME = 'DEFAULT_TEMP_TABLESPACE'; -- 默认的临时表空间
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
alter tablespace jion                                                  		-- 表空间的名字(默认与用户名一致)
read only ;                                                                	-- 只读
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



## undo 表空间

### undo表空间
针对DML语句,修改了数据块,Oracle则会将修改前的数据保留下来,保存在undo segment 中,进而保存在undo表空间中,当不够时,会被覆盖. snapshot to old
  前镜像/后镜像.

* 一致性读   在某一时刻读的数据,为当前读时刻的数据,不会被同时修改的数据而覆盖
* 回滚事务   回滚数据前的数据
* 实例恢复   SMON检查undo segment头部,如果有未提交的,会被回滚.



### 常见变量

| 变量                | 类型    | 值       | 说明                                                         |
| ------------------- | ------- | -------- | ------------------------------------------------------------ |
| undo_management     | string  | AUTO     | undo segment的提交到undo表空间的管理机制                     |
| undo_retention      | integer | 900      | 900S内,undo中保留旧数据时间                                  |
| undo_tablespace     | string  | UNDOTBS1 | undo表空间                                                   |
| rentention garentee |         |          | 当undo数据文件不能自动扩展时,且undo块不够时,直接报错;而不是覆盖原有的undo块 |

```sql
-- 查看系统中的undo表空间
select * from dba_tablespaces;
-- 查看系统中的表空间文件
select * from dba_data_files;
-- 创建一个回滚段表空间
create undo tablespace undotbs2 datafile
'E:\ORACLE\ORADATA\ORCL\undotbs2.DBF'
size 10M
autoextend on;

-- 为回滚段表空间新增临时文件
alter tablespace undotbs2 add datafile
'E:\ORACLE\ORADATA\ORCL\undotbs2b.DBF'
size 10M
autoextend on;

-- 切换undo表空间
alter system set undo_tablespace = undotbs2;
-- 启用rentention garentee;
alter tablespace undotbs1 retention guarantee;
-- 取消rentention garentee;
alter tablespace undotbs1 retention noguarantee;
-- 查看启用情况  retention参数
select * from dba_tablespaces;
-- 查看undo段的使用情况
select * from v$undostat;
```
