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



# 内容简介

Oracle数据库中表空间导出命令,限于当前用户有相应权限,并在命令行窗口下执行,注意结尾处没有分号,并删除注释.操作时避免相关用户登录或者高并发访问

## 数据导出命令

``` sql       
expdp system/123456                                                         -- 导出数据库账号/密码
schemas=system                                                              -- 数据库数据段名称
directory=sysload_file_dir                                                  -- 导出的位置,为数据库使用文件夹的名称,之前已由db定义
dumpfile=system_2017.dmp                                                    -- 导出的文件名称
logfile=system_2017.log                                                     -- 生成的日志名称,结尾处不可以有分号.
```

## 数据导入命令

``` sql
--数据库导入命令
impdp system/123456                                                         -- 导入数据库账号/密码
dumpfile=system_2017.dmp                                                    -- 导入文件名
directory=sysload_file_dir                                                  -- 导入文件的路径,为数据库使用文件夹的名称,之前已由db定义
logfile=system_2017.log                                                     -- 输出日志格式
remap_schema=system:system                                                  -- 数据段更换,system->system.不支持db管理员的重新映射!!!
remap_tablespace=system:system                                              -- 表空间更换,system->system.不支持DB管理员的重新映射!!!
```

# 逻辑备份与恢复

## 备份

* 备份分为物理备份与逻辑备份
* 物理备份对数据库操作系统文件,如数据文件,控制文件,日志文件..的备份
* 逻辑备份是对数据库逻辑组件,如表,视图,存储过程..备份
故障类型
* 语句故障,SQL错误语句,修改SQL语句即可
* 用户进程故障,因为用户自身原因,导致连接不上服务器.
* 实例故障,服务器端出现死机,荡机,重启Oracle解决
* 介质故障,硬盘文件损毁

## 导出
实现对数据库逻辑层次的备份

* 可以在不同版本数据库中迁移
* 从新组织表,降低HWM
  模式选择:
  * 数据库模式,整个数据库的所有对象
  * 表空间模式,一个或者多个表空间内的所有对象
  * 用户模式,用户下的所有对象
  * 表模式,一个或者多个表或者表分区

### 参数列表

- userid       导出用户名/密码
- buffer       导出时缓存字节大小
- file         导出的二进制文件名,默认扩展名 .dmp
- full         以全部数据库方式导出,需要授权
- owner        导出数据库的用户列表
- help         帮助
- rows         是否导出表中数据
- tables       按表方式导出,指定导出表和表分区的名称
- parfile      传递参数时的参数文件名
- tablespaces  按空间方式导出,指定导出空间名

### 语法

```sql
 exp  直接进入交互命令
 exp help=y 显示帮助文档
    通过输入 EXP 命令和您的用户名/口令, 导出
    操作将提示您输入参数:

         例如: EXP SCOTT/TIGER

    或者, 您也可以通过输入跟有各种参数的 EXP 命令来控制导出
    的运行方式。要指定参数, 您可以使用关键字:

         格式:  EXP KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
         例如: EXP SCOTT/TIGER GRANTS=Y TABLES=(EMP,DEPT,MGR)
                   或 TABLES=(T1:P1,T1:P2), 如果 T1 是分区表

    USERID 必须是命令行中的第一个参数。

    关键字   说明 (默认值)         关键字      说明 (默认值)
    --------------------------------------------------------------------------
    USERID   用户名/口令           FULL        导出整个文件 (N)
    BUFFER   数据缓冲区大小        OWNER        所有者用户名列表
    FILE     输出文件 (EXPDAT.DMP)  TABLES     表名列表
    COMPRESS  导入到一个区 (Y)   RECORDLENGTH   IO 记录的长度
    GRANTS    导出权限 (Y)          INCTYPE     增量导出类型
    INDEXES   导出索引 (Y)         RECORD       跟踪增量导出 (Y)
    DIRECT    直接路径 (N)         TRIGGERS     导出触发器 (Y)
    LOG      屏幕输出的日志文件    STATISTICS    分析对象 (ESTIMATE)
    ROWS      导出数据行 (Y)        PARFILE      参数文件名
    CONSISTENT 交叉表的一致性 (N)   CONSTRAINTS  导出的约束条件 (Y)

    OBJECT_CONSISTENT    只在对象导出期间设置为只读的事务处理 (N)
    FEEDBACK             每 x 行显示进度 (0)
    FILESIZE             每个转储文件的最大大小
    FLASHBACK_SCN        用于将会话快照设置回以前状态的 SCN
    FLASHBACK_TIME       用于获取最接近指定时间的 SCN 的时间
    QUERY                用于导出表的子集的 select 子句
    RESUMABLE            遇到与空格相关的错误时挂起 (N)
    RESUMABLE_NAME       用于标识可恢复语句的文本字符串
    RESUMABLE_TIMEOUT    RESUMABLE 的等待时间
    TTS_FULL_CHECK       对 TTS 执行完整或部分相关性检查
    TABLESPACES          要导出的表空间列表
    TRANSPORT_TABLESPACE 导出可传输的表空间元数据 (N)
    TEMPLATE             调用 iAS 模式导出的模板名

    成功终止导出, 没有出现警告。
```

### 示例

* 导出用户下的某张表   tables 需要导出的表   file 导出文件  log 日志位置
`exp zhangqian/123456@127.0.0.1/orcl tables=(emp,dept) file=F:\ORACLE_Workspace\database\zhangqian_tables.dmp log=F:\ORACLE_Workspace\database\zhangqian_tables.log`

* 导出用户下的所有对象
`exp zhangqian/123456@127.0.0.1/orcl owner=zhangqian file=F:\ORACLE_Workspace\database\zhangqian_user.dmp log=F:\ORACLE_Workspace\database\zhangqian_user.log`

* 导出表空间
`exp system/123456@127.0.0.1/orcl tablespaces=(users) file=F:\ORACLE_Workspace\database\tablespaces_users.dmp log=F:\ORACLE_Workspace\database\tablespaces_users.log`

* 导出数据库
`exp system/123456@127.0.0.1/orcl full=y file=F:\ORACLE_Workspace\database\loacl_db.dmp log=F:\ORACLE_Workspace\database\loacl_db.log`



##　导出

### 语法

```sql
  imp help=y 查看帮助
  
    通过输入 IMP 命令和您的用户名/口令, 导入
    操作将提示您输入参数:

         例如: IMP SCOTT/TIGER

    或者, 可以通过输入 IMP 命令和各种参数来控制导入
    的运行方式。要指定参数, 您可以使用关键字:

         格式:  IMP KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
         例如: IMP SCOTT/TIGER IGNORE=Y TABLES=(EMP,DEPT) FULL=N
                   或 TABLES=(T1:P1,T1:P2), 如果 T1 是分区表

    USERID 必须是命令行中的第一个参数。

    关键字   说明 (默认值)        关键字      说明 (默认值)
    --------------------------------------------------------------------------
    USERID   用户名/口令           FULL       导入整个文件 (N)
    BUFFER   数据缓冲区大小        FROMUSER    所有者用户名列表
    FILE     输入文件 (EXPDAT.DMP)  TOUSER     用户名列表
    SHOW     只列出文件内容 (N)     TABLES      表名列表
    IGNORE   忽略创建错误 (N)    RECORDLENGTH  IO 记录的长度
    GRANTS   导入权限 (Y)          INCTYPE     增量导入类型
    INDEXES   导入索引 (Y)         COMMIT       提交数组插入 (N)
    ROWS     导入数据行 (Y)        PARFILE      参数文件名
    LOG     屏幕输出的日志文件    CONSTRAINTS    导入限制 (Y)
    DESTROY                覆盖表空间数据文件 (N)
    INDEXFILE              将表/索引信息写入指定的文件
    SKIP_UNUSABLE_INDEXES  跳过不可用索引的维护 (N)
    FEEDBACK               每 x 行显示进度 (0)
    TOID_NOVALIDATE        跳过指定类型 ID 的验证
    FILESIZE               每个转储文件的最大大小
    STATISTICS             始终导入预计算的统计信息
    RESUMABLE              在遇到有关空间的错误时挂起 (N)
    RESUMABLE_NAME         用来标识可恢复语句的文本字符串
    RESUMABLE_TIMEOUT      RESUMABLE 的等待时间
    COMPILE                编译过程, 程序包和函数 (Y)
    STREAMS_CONFIGURATION  导入流的一般元数据 (Y)
    STREAMS_INSTANTIATION  导入流实例化元数据 (N)
    DATA_ONLY              仅导入数据 (N)

    下列关键字仅用于可传输的表空间
    TRANSPORT_TABLESPACE 导入可传输的表空间元数据 (N)
    TABLESPACES 将要传输到数据库的表空间
    DATAFILES 将要传输到数据库的数据文件
    TTS_OWNERS 拥有可传输表空间集中数据的用户
```

### 示例

  * 导入表  tables 导入表   fromuser 导出用户  touser 导入用户 ignore 忽略错误,重复插入
    `imp system/123456@127.0.0.1/orcl tables=(emp,dept) fromuser=zhangqian touser=zhangqian ignore=y file=F:\ORACLE_Workspace\database\zhangqian_tables.dmp `
  
  * 将整个文件导入
    `imp zhangqian/123456@127.0.0.1/orcl full=y ignore=y file=F:\ORACLE_Workspace\database\zhangqian_user.dmp`



## 可传输表空间

  当数据库数据量很大时,可以使用可传输表空间

### 步骤

1. 检查准备传输的表空间是否为自包含

      ```sql
      -- 创建准备传递的表空间
      create tablespace trans_tablespace datafile 'E:\Oracle\oradata\orcl\trans_tablespace.dbf' size 10M;
      -- 检查表空间是否自包含,即表空间内的对象都在本表空间之内.
      dbms_tts. 
      ```

2. 将表空间设置为只读.
   `alter table trans_tablespace read only;`
3. exp进行可传输表空间模式的导出
   `exp system/123456@127.0.0.1/orcl tablespaces=trans_tablespace transport_tablespace=y file=F:\ORACLE_Workspace\database\trans_tablespace_bak.dmp`
4. 将导出文件和数据文件复制到目标数据库上. 
   `trans_tablespace_bak.dbf 和 trans_tablespace.dbf 文件`
5. 目标数据库上,imp进行可传输表空间模式的导入
   `imp system/123456@127.0.0.1/orcl tablespaces=trans_tablespace transport_tablespace=y file=F:\ORACLE_Workspace\database\trans_tablespace_bak.dmp datafiles=trans_tablespace.dbf`
6. 目标数据库上,把表空间设置为读写状态
   `alter table trans_tablespace read write;`



# 数据泵

`expdp help=y`  查看帮助
 数据泵导出实用程序提供了一种用于在 Oracle 数据库之间传输
 数据对象的机制。该实用程序可以使用以下命令进行调用:

## 导出

### 语法

```bash
expdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp

您可以控制导出的运行方式。具体方法是: 在 'expdp' 命令后输入
各种参数。要指定各参数, 请使用关键字:

   格式:  expdp KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
   示例: expdp scott/tiger DUMPFILE=scott.dmp DIRECTORY=dmpdir SCHEMAS=scott
               或 TABLES=(T1:P1,T1:P2), 如果 T1 是分区表

USERID 必须是命令行中的第一个参数。

------------------------------------------------------------------------------

以下是可用关键字和它们的说明。方括号中列出的是默认值。

ATTACH
连接到现有作业。
例如, ATTACH=job_name。

COMPRESSION
减少转储文件大小。
有效的关键字值为: ALL, DATA_ONLY, [METADATA_ONLY] 和 NONE。

CONTENT
指定要卸载的数据。
有效的关键字值为: [ALL], DATA_ONLY 和 METADATA_ONLY。

DATA_OPTIONS
数据层选项标记。
有效的关键字值为: XML_CLOBS。

DIRECTORY
用于转储文件和日志文件的目录对象。

DUMPFILE
指定目标转储文件名的列表 [expdat.dmp]。
例如, DUMPFILE=scott1.dmp, scott2.dmp, dmpdir:scott3.dmp。

ENCRYPTION
加密某个转储文件的一部分或全部。
有效的关键字值为: ALL, DATA_ONLY, ENCRYPTED_COLUMNS_ONLY, METADATA_ONLY 和 NONE。

ENCRYPTION_ALGORITHM
指定加密的方式。
有效的关键字值为: [AES128], AES192 和 AES256。

ENCRYPTION_MODE
生成加密密钥的方法。
有效的关键字值为: DUAL, PASSWORD 和 [TRANSPARENT]。

ENCRYPTION_PASSWORD
用于在转储文件中创建加密数据的口令密钥。

ESTIMATE
计算作业估计值。
有效的关键字值为: [BLOCKS] 和 STATISTICS。

ESTIMATE_ONLY
计算作业估计值而不执行导出。

EXCLUDE
排除特定对象类型。
例如, EXCLUDE=SCHEMA:"='HR'"。

FILESIZE
以字节为单位指定每个转储文件的大小。

FLASHBACK_SCN
用于重置会话快照的 SCN。

FLASHBACK_TIME
用于查找最接近的相应 SCN 值的时间。

FULL
导出整个数据库 [N]。

HELP
显示帮助消息 [N]。

INCLUDE
包括特定对象类型。
例如, INCLUDE=TABLE_DATA。

JOB_NAME
要创建的导出作业的名称。

LOGFILE
指定日志文件名 [export.log]。

NETWORK_LINK
源系统的远程数据库链接的名称。

NOLOGFILE
不写入日志文件 [N]。

PARALLEL
更改当前作业的活动 worker 的数量。

PARFILE
指定参数文件名。

QUERY
用于导出表的子集的谓词子句。
例如, QUERY=employees:"WHERE department_id > 10"。

REMAP_DATA
指定数据转换函数。
例如, REMAP_DATA=EMP.EMPNO:REMAPPKG.EMPNO。

REUSE_DUMPFILES
覆盖目标转储文件 (如果文件存在) [N]。

SAMPLE
要导出的数据的百分比。

SCHEMAS
要导出的方案的列表 [登录方案]。

SOURCE_EDITION
用于提取元数据的版本。

STATUS
监视作业状态的频率, 其中
默认值 [0] 表示只要有新状态可用, 就立即显示新状态。

TABLES
标识要导出的表的列表。
例如, TABLES=HR.EMPLOYEES,SH.SALES:SALES_1995。

TABLESPACES
标识要导出的表空间的列表。

TRANSPORTABLE
指定是否可以使用可传输方法。
有效的关键字值为: ALWAYS 和 [NEVER]。

TRANSPORT_FULL_CHECK
验证所有表的存储段 [N]。

TRANSPORT_TABLESPACES
要从中卸载元数据的表空间的列表。

VERSION
要导出的对象版本。
有效的关键字值为: [COMPATIBLE], LATEST 或任何有效的数据库版本。

------------------------------------------------------------------------------

下列命令在交互模式下有效。
注: 允许使用缩写。

ADD_FILE
将转储文件添加到转储文件集。

CONTINUE_CLIENT
返回到事件记录模式。如果处于空闲状态, 将重新启动作业。

EXIT_CLIENT
退出客户机会话并使作业保持运行状态。

FILESIZE
用于后续 ADD_FILE 命令的默认文件大小 (字节)。

HELP
汇总交互命令。

KILL_JOB
分离并删除作业。

PARALLEL
更改当前作业的活动 worker 的数量。

REUSE_DUMPFILES
覆盖目标转储文件 (如果文件存在) [N]。

START_JOB
启动或恢复当前作业。
有效的关键字值为: SKIP_CURRENT。

STATUS
监视作业状态的频率, 其中
默认值 [0] 表示只要有新状态可用, 就立即显示新状态。

STOP_JOB
按顺序关闭作业执行并退出客户机。
有效的关键字值为: IMMEDIATE。  
```

### 导出示例

```sql

创建目录对象,将导出文件放入
create directory FILE_DIR as 'F:\ORACLE_Workspace\database\file';
导出表    directory 导出目录     tables 导出表名  
expdp zhangqian/123456@127.0.0.1/orcl directory=FILE_DIR dumpfile=zhangqian_tables.dmp tables=(emp,dept);  

导出表结构 content 设置,只有表结构  reuse_dumpfiles 覆盖导出
expdp zhangqian/123456@127.0.0.1/orcl directory=FILE_DIR dumpfile=zhangqian_tables.dmp tables=(emp,dept) content=metadata_only reuse_dumpfiles=y;

导出表数据 content 设置,只有数据
expdp zhangqian/123456@127.0.0.1/orcl directory=FILE_DIR dumpfile=zhangqian_tables.dmp tables=(emp,dept) content=data_only reuse_dumpfiles=y;

导出表,表主键,但是导出约束   exclude=index 表示排除索引  constraints=n 表示不导出索引
expdp zhangqian/123456@127.0.0.1/orcl directory=FILE_DIR dumpfile=zhangqian_tables.dmp tables=(emp,dept) exclude=index reuse_dumpfiles=y;

导出用户  schemas 导出的用户  job_name 导出文件名,顺便生成同名的MT表
expdp system/123456@127.0.0.1/orcl directory=FILE_DIR dumpfile=zhangqian_tables.dmp schemas=(zhangqian,scott) job_name=users_tablespces;

导出用户 排除某表
expdp system/123456@127.0.0.1/orcl directory=FILE_DIR dumpfile=zhangqian_tables.dmp schemas=(zhangqian,scott) exclude=table:"in ('EMP')";
```



## 导入

### 语法

```sql
  impdp help=y  查看帮助
    数据泵导入实用程序提供了一种用于在 Oracle 数据库之间传输
    数据对象的机制。该实用程序可以使用以下命令进行调用:

         示例: impdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp

    您可以控制导入的运行方式。具体方法是: 在 'impdp' 命令后输入
    各种参数。要指定各参数, 请使用关键字:

         格式:  impdp KEYWORD=value 或 KEYWORD=(value1,value2,...,valueN)
         示例: impdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp

    USERID 必须是命令行中的第一个参数。


    ------------------------------------------------------------------------------

    以下是可用关键字和它们的说明。方括号中列出的是默认值。

    ATTACH
    连接到现有作业。
    例如, ATTACH=job_name。

    CONTENT
    指定要加载的数据。
    有效的关键字为: [ALL], DATA_ONLY 和 METADATA_ONLY。

    DATA_OPTIONS
    数据层选项标记。
    有效的关键字为: SKIP_CONSTRAINT_ERRORS。

    DIRECTORY
    用于转储文件, 日志文件和 SQL 文件的目录对象。

    DUMPFILE
    要从中导入的转储文件的列表 [expdat.dmp]。
    例如, DUMPFILE=scott1.dmp, scott2.dmp, dmpdir:scott3.dmp。

    ENCRYPTION_PASSWORD
    用于访问转储文件中的加密数据的口令密钥。
    对于网络导入作业无效。

    ESTIMATE
    计算作业估计值。
    有效的关键字为: [BLOCKS] 和 STATISTICS。

    EXCLUDE
    排除特定对象类型。
    例如, EXCLUDE=SCHEMA:"='HR'"。

    FLASHBACK_SCN
    用于重置会话快照的 SCN。

    FLASHBACK_TIME
    用于查找最接近的相应 SCN 值的时间。

    FULL
    导入源中的所有对象 [Y]。

    HELP
    显示帮助消息 [N]。

    INCLUDE
    包括特定对象类型。
    例如, INCLUDE=TABLE_DATA。

    JOB_NAME
    要创建的导入作业的名称。

    LOGFILE
    日志文件名 [import.log]。

    NETWORK_LINK
    源系统的远程数据库链接的名称。

    NOLOGFILE
    不写入日志文件 [N]。

    PARALLEL
    更改当前作业的活动 worker 的数量。

    PARFILE
    指定参数文件。

    PARTITION_OPTIONS
    指定应如何转换分区。
    有效的关键字为: DEPARTITION, MERGE 和 [NONE]。

    QUERY
    用于导入表的子集的谓词子句。
    例如, QUERY=employees:"WHERE department_id > 10"。

    REMAP_DATA
    指定数据转换函数。
    例如, REMAP_DATA=EMP.EMPNO:REMAPPKG.EMPNO。

    REMAP_DATAFILE
    在所有 DDL 语句中重新定义数据文件引用。

    REMAP_SCHEMA
    将一个方案中的对象加载到另一个方案。

    REMAP_TABLE
    将表名重新映射到另一个表。
    例如, REMAP_TABLE=EMP.EMPNO:REMAPPKG.EMPNO。

    REMAP_TABLESPACE
    将表空间对象重新映射到另一个表空间。

    REUSE_DATAFILES
    如果表空间已存在, 则将其初始化 [N]。

    SCHEMAS
    要导入的方案的列表。

    SKIP_UNUSABLE_INDEXES
    跳过设置为“索引不可用”状态的索引。

    SOURCE_EDITION
    用于提取元数据的版本。

    SQLFILE
    将所有的 SQL DDL 写入指定的文件。

    STATUS
    监视作业状态的频率, 其中
    默认值 [0] 表示只要有新状态可用, 就立即显示新状态。

    STREAMS_CONFIGURATION
    启用流元数据的加载

    TABLE_EXISTS_ACTION
    导入对象已存在时执行的操作。
    有效的关键字为: APPEND, REPLACE, [SKIP] 和 TRUNCATE。

    TABLES
    标识要导入的表的列表。
    例如, TABLES=HR.EMPLOYEES,SH.SALES:SALES_1995。

    TABLESPACES
    标识要导入的表空间的列表。

    TARGET_EDITION
    用于加载元数据的版本。

    TRANSFORM
    要应用于适用对象的元数据转换。
    有效的关键字为: OID, PCTSPACE, SEGMENT_ATTRIBUTES 和 STORAGE。

    TRANSPORTABLE
    用于选择可传输数据移动的选项。
    有效的关键字为: ALWAYS 和 [NEVER]。
    仅在 NETWORK_LINK 模式导入操作中有效。

    TRANSPORT_DATAFILES
    按可传输模式导入的数据文件的列表。

    TRANSPORT_FULL_CHECK
    验证所有表的存储段 [N]。

    TRANSPORT_TABLESPACES
    要从中加载元数据的表空间的列表。
    仅在 NETWORK_LINK 模式导入操作中有效。

    VERSION
    要导入的对象的版本。
    有效的关键字为: [COMPATIBLE], LATEST 或任何有效的数据库版本。
    仅对 NETWORK_LINK 和 SQLFILE 有效。

    ------------------------------------------------------------------------------

    下列命令在交互模式下有效。
    注: 允许使用缩写。

    CONTINUE_CLIENT
    返回到事件记录模式。如果处于空闲状态, 将重新启动作业。

    EXIT_CLIENT
    退出客户机会话并使作业保持运行状态。

    HELP
    汇总交互命令。

    KILL_JOB
    分离并删除作业。

    PARALLEL
    更改当前作业的活动 worker 的数量。

    START_JOB
    启动或恢复当前作业。
    有效的关键字为: SKIP_CURRENT。

    STATUS
    监视作业状态的频率, 其中
    默认值 [0] 表示只要有新状态可用, 就立即显示新状态。

    STOP_JOB
    按顺序关闭作业执行并退出客户机。
    有效的关键字为: IMMEDIATE。  

```



### 导入示例

导入  directory 使用目录    dumpfile 导入的文件    remap_schema 将XX的对象导入YY  remap_tablespace 将users表空间转储为zhangqian表空间

```bash
impdp zhangqian/123456@127.0.0.1/orcl directory=FILE_DIR dumpfile=zhangqian_tables.dmp remap_schema=zhangqian:zhangqian remap_tablespace=users:zhangqian
```



# 数据与装载

数据加载,将外部的文件导入数据库中

`sqlldr` 使用方式

1. 只使用一个控制文件,在这个控制文件中包含数据
2. 使用一个控制文件作为模板,和一个数据文件作为数据

## 示例

控制文件

```sql
-- 创建表
create table address(sno int , address varchar2(50));
```

执行脚本

```bash
options(skip=0)                                              -- 跳过0行
load data                                                    -- 加载数据
infile 'F:\ORACLE_Workspace\database\file\data_file.txt'     -- 数据文件位置
into table address insert                                    -- 插入表   insert 空表插入,非空表不能执行    append 追加操作  replace --- 删除后插入  truncate 截断后插入
(    
sno       char terminated by ',',                          -- 格式分隔
address   char terminated by ','    
)
```

数据文件的数据格式

```csv
1,Jion, 上海                                                 
2,Arise,信阳
```

执行 userid 登录用户   control 控制文件

```sql
sqlldr userid=zhangqian/123456@127.0.0.1/orcl  control=F:\ORACLE_Workspace\database\file\control_file.txt  log=F:\ORACLE_Workspace\database\file\sqlldr.log
```

数据导出

```bash
spool F:\ORACLE_Workspace\database\file\data_file\data_file.txt
select sno || ',' || address from address;
spool off
```



# 外部表

数据库只存储表结构,但是数据具体内容保存在操作系统中,外部表是只读的

## 数据泵生成外部表

```sql
1.创建数据泵文件
create table emp_txt(empno , ename , sal )
organization external                           -- 表示这是一个外部表
(
  type oracle_datapump                          -- 使用数据泵生成外部表
  default directory FILE_DIR                    -- 表所在的目录
  location('external.dmp')                      -- 外部表存放文件,没有则创建
)
parallel as
select empno , ename , sal from emp;            -- 数据来源

-- 查询外部表
select * from emp_txt;  
```

## 根据已有数据泵文件,生成外部表

```sql
create table emp_txt(empno number , ename varchar2(10) , sal number)
organization external                           -- 表示这是一个外部表
(
  type oracle_datapump                          -- 使用数据泵生成外部表
  default directory FILE_DIR                    -- 表所在的目录
  location('external.dmp')                      -- 外部表存放文件
);

-- 查询外部表
select * from emp_txt;
```

## 根据文本创建外部表

```sql
create table address_txt(sno number , address varchar2(20))
organization external
(
  type oracle_loader                            -- 使用加载器生成外部表
  default directory FILE_DIR                    -- 表所在的目录
  access parameters
  (
    records delimited by newline
    fields  terminated by ',' 
  )
  location('data_file.txt')                      -- 外部表存放文件,相对于所在目录位置
)
-- 查询外部表
select * from address_txt;
```

