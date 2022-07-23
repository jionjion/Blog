---
title: Oracle基础-体系结构
typora-root-url: ../
date: 2022-07-20 10:10:54
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---

> 介绍 Oracle 最基础的体系结构及目录



# 环境工作

## 前端控制台

开启 `OracleDBConsoleorcl` , 开启服务控制台
https://localhost:1158/em/console/logon/logon



## 监听连接

### 服务端数据库服务

监听文件,位于服务端 `listener.ora`
至少启动
`OracleOraDb11g_home1TNSListener` 服务  没有开启时,可以通过 `SQL/Plus` 命令行进入,网络TCP连接无效
`OracleSericeORCL` 服务启动数据库

### 客户端监听服务

客户端连接文件 `tesname.ora`
至少启动
`OracleSericeORCL服务` 保证连接
使用命令行 `lsnrctl start/stop/status` 启动监听程序

上述三个文件位于  位置 `Oracle` 的安装目录
`E:\Oracle\product\11.2.0\dbhome_1\NETWORK\ADMIN` 

### **`listener.ora` 文件** 

记录了连接的地址信息

```
LISTENER = 
(DESCRIPTION_LIST = 
  (DESCRIPTION = 
	(ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1))                # 调用外部过程的监听地址
	(ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521)) # 监听本地服务的地址信息
  ) 
)
```

### `sqlnet.ora` 文件

```bash
SQLNET.AUTHENTICATION_SERVICES = (NTS)  允许不输入密码,直接通过sql/plus连接数据库
NAMES.DIRECTORY_PATH = (TNSNAMES,EZCONNECT)  连接方式:指明服务名连接,简单连接(类似jdbc地址)
```

### 启动命令

#### `tnsping`

测试服务名能否连接
例如: `tnsping orcl`

#### `lsnrctl`

使用命令行 `lsnrctl start/stop/status` 启动监听程序

#### `netca`

打开网路配置界面

- 对 `sqlnet.ora` 文件进行修改,设定允许客户端连接的方式
- 对 `listener.ora` 监听文件进行创建修改,可以有多个端口监听服务
- 对 `tnsnames.ora` 文件 可以监听多个服务名,服务网地址,计算机名



## 注册

将数据库作为服务,注册到监听程序中,使监听器知道服务器中数据库信息, 修改 `listener.ora` 文件

### 静态注册

在`listener.ora` 文件中注册

```bash
SID_LIST_LISTENER = 
(SID_LIST = 
  (SID_DESC = 
	(SID_NAME = PLSExtProc) 
	(ORACLE_HOME = e:\oracle\product\10.2.0\db_1) 
	(PROGRAM = extproc) 
  ) 
  (SID_DESC =                                               # 监听的数据库描述
	(GLOBAL_DBNAME = orcl)                                  # 服务名
	(ORACLE_HOME = e:\oracle\product\10.2.0\db_1)           # 数据db文件主目录
	(SID_NAME = orcl)                                       # 注册id
  ) 
)
```

**另外可以使用 net manager可视化工具进行配置**

### 动态注册

`tnsnames.ora` 文件可以不需要, 由 `PMON` 进程动态注册



## 数据库状态
`shutdown`  实例, 数据库都关闭
`no mount`  实例启动,读取初始参数文件,分配物理内存,启动后台进程,但是数据库未打开
`mount`     数据库装载完成,打开控制文件,可以找到database信息,但是用户表内容不可读
`open`      数据库打开并成功加载,可以访问用户表

查看打开方式
`select open_mode from v$database`



# 事务管理

## Oracle 数据锁

### 要求

1. 一致性,一次只允许一个用户修改
2. 完整性,修改后,多个用户修改后可见
3. 并行性,允许多个用户修改数据



### 锁的类型

- 共享锁

  同一个对象可以由其他用户继续加锁

- 排它锁

  同一个对象只能有一个用户加锁

- 死锁

  循环等待另一个锁,不过会被Oracle自动解决,杀死某个锁



#### 行级锁(TX)

是一种排它锁,防止其他事物修改该行.

`insert` ; `delete` ; `update` ; `select..for update` 执行时, 使用 `commit` / `rollback` 释放行级锁.

`select * .... for update`   			锁定整张表,允许增加,但是不允许修改删除

`select ... for update wait 5` 		查询数据,只能5秒,如果5秒内获得不到锁,则抛出异常
`select ... for update nowait` 		查询数据,如果此刻被其他锁获得,则抛出异常,该资源正忙



#### 表级锁(TM)

限制对其他用户的访问

`lock table 表名 in 类型名 mode `

| 模式                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| row share           | 行共享,禁止排他锁定表[exclusive]                             |
| row exclusive       | 行排他,禁止使用排他锁和共享锁[share , exclusive]             |
| share               | 共享锁,锁定表,仅允许用户查询表中的行;禁止其他用户插入,更新,删除行;多个用户可以同时在同一表上应用此锁 |
| share row exclusive | 共享行排他锁,比共享锁更多的限制,禁止使用共享锁及更高的锁     |
| exclusive           | 排他锁,限制最高的锁,仅允许其他用户查询该表的行.禁止修改和锁定表 |



#### 相关查询

查看系统当前锁  `ID` 对应 `dba_objects` 表主键

```sql
select * from v$lock;

select * from dba_objects t where t.OBJECT_ID = 100;
```



# 体系结构

## 目录

```bash
Oracle体系结构
       |- Oracle实例:管理数据的后台线程和内存结构的结合.
       |    |- 用户进程 需要与Oracle服务器进行交互的程序.当用户运行一个应用程序准备向服务器发送请求时,创建改用户进程,位于客户端
       |    |- 内存结构.分为两部分,
       |       |- 程序全局区(PGA),服务器进程启动时,分配20%,包含单个服务器进程所需要的数据和控制信息,每个连接单独占有一个PGA
       |          |-服务器进程 , 一一对应用户进程
       |       |- 系统全局区(SGA),在实例启动时,分配80%
       |          |- 共享池 
       |             * 对SQL,PL/SQL程序进行语法分析,编译,生成执行计划并执行的内存区域
       |             * 可以共享的执行计划,其SQL要求大小写,缩进书写一致.
       |             * 由库缓存和数据字典组成
       |             * 共享池的大小直接影响数据库的性能
       |          |- 数据缓冲区
       |             * 用于储存从硬盘数据文件中读入的数据,所有用户共享
       |             * 增删改查会将数据首先变更/保存在缓存区中操作,随后再写入文件
       |             * 服务器进程将读入的数据保存在数据缓冲区中,当后续的请求需要这些数据时可以在内存中找到
       |             * 数据缓冲区的大小读数据库的读取速度有直接影响
       |          |- 日志缓冲区
       |             * 产生日志,记录数据库的所有操作
       |             * 当缓存到达一定数量后(1/3内存占用;每隔3S;发出commit;积累1M空间),由后台进程将日志写入在线日志文件中
       |          |- Large池
       |             * 为了进行大的后台进程而分配的内存空间,主要是:数据备份/回复,大型IO操作,并行查询等
       |          |- Steam池
       |             * 为了stream流应用而分配的内存空间
       |          |- JAVA池
       |             * 为了java应用而分配的内存空间
       |    |- 后台进程
       |       |- PMON 进程监控进程
       |          * 清理出故障的进程
       |          * 释放当前挂起的锁
       |          * 释放故障进程使用的资源
       |       |- SMON 系统监控进程
       |          * 在实例失败(服务器断电,重启)之后,重新打开数据库时自动恢复实例
       |          * 整理数据文件的自由空间,将相邻的区域结合起来
       |          * 释放不再使用的临时段
       |       |- DBWR 数据写进程
       |          * 管理数据缓冲区,将最近使用的块保留在内存中
       |          * 数据从数据缓冲区写入数据文件的进程
       |       |- LGWR 日志写进程
       |          * 负责将日志缓冲区的日志数据写入日志文件
       |          * 系统有多个日志文件时,循环写入
       |       |- CKPT 检查点进程
       |          * 防止实例崩溃,尽快进行实例恢复的进程.分两类,在对应情况下触发
       |          - 完全检查点
       |            1. alter system checkpoint;
       |            2. 除了 shutdown abort 以外的其他方式正常关闭数据库
       |          - 增量检查点
       |            1. 每隔3S
       |            2. 日志切换
       |       |- MMAN 自动管理SGA(系统全局区)进程
       |       |- MMON 自动统计信息收集
       |       |- 其他
       |- Oracle数据库,是一个数据的集合,改集合视为一个逻辑单元.
           |- 主要物理文件  位置: E:\Oracle\oradata\orcl
               |- 控制文件 记录数据库结构的二进制文件
               |- 数据文件 储存数据库的数据,表,索引等
               |- 日志文件 记录对数据库的所有修改信息,用以故障恢复
           |- 非主要物理文件 位置 ORACLE_HOME  E:\Oracle\product\11.2.0\dbhome_1\database
               |- 参数文件 位置:E:\Oracle\product\11.2.0\dbhome_1\database
               |- 密码文件 位置:E:\Oracle\product\11.2.0\dbhome_1\database
               |- 告警和跟踪文件 位置:
               |- 归档日志文件
           |- 逻辑结构   数据库 -> 表空间 -> 段 -> 区 -> 数据块
               |- 表空间 
                  * 数据库最大的逻辑单位,至少拥有一个system系统表空间
                  * 每个表空间由一个或者多个数据文件组成,一个数据文件只能与一个表空间关联
                  * 表空间的大小等于该表空间的所有数据文件大小之和
                  select * from v$tablespace; -- 查看表空间大小,默认有6个表空间
                     SYSTEM   系统表空间,存放系统的最基本信息
                     SYSAUX   作为辅助表空间,减少系统表空间的负荷
                     UNDOTBS1 撤销表空间,UNDO类型.保存用户在增删改查之前的数据
                     USERS    数据库默认的永久表空间,当创建用户不指定用户表空间时,默认将新用户的表空间内容 挂载在USERS表空间中
                     TEMP     临时表空间,当排序不能在分配的空间中完成时,就会使用磁盘排序的方式,在临时表空间中进行.
                     EXAMPLE  数据库测试用例所涉及到的表的所属表空间
               |- 段
                  * 由区构成,通过增加区数而扩展段
                  * 按照储存数据的特征,可以分为数据段,索引段,回滚段,临时段
               |- 区
                  * 由连续的数据块构成
                  * 当段中的空间使用完后,系统会自动为段分配一个新区
                  * 区不能跨数据文件存在,只能存在于一个数据文件中.
               |- 数据块
                  * Oracle所能分配,读取,写入的最小储存单元
                  * 默认大小为 8K
                  show parameters db_block_size
               |- 模式 对用户创建的数据库对象的总称,模式名与用户名一致,等同于用户
                  * 包括 表 , 视图 , 索引 , 同义词 , 序列 , 过程 , 程序包 ...
```

## 设置

### 配置查询

```sql
-- 查看进程
select * from v$process

-- 查看 DBWR的线程数量c
show parameters db_wr

-- 修改线程数量为1,下次启动生效
alter system set db_writer_processes=1 scope=spfile
```

### PGA(程序全局区)管理
设置 `workarea_size_policy` 参数为 `auto`
`DBA` 根据数据库的负载情况,估计所有的session大概需要消耗的PGA的总的大小,然后把改值设置成初始化参数 `pga_aggregate_target` ,oracle会自动调整每个session的 `PGA` 的大小
    

### SGA(系统全局区)管理    
设置 `statistices_level`为 `typical` 或者 `all`
只需为 `SGA` 分配一个总大小,初始化参数 `sga_target` 设定
    

### 自动内存管理
设置 `statistices_level` 为 `typical` 或者 `all`
只需为 `Oracle` 的使用分配一个整体大小 , 设置初始化参数 `memory_target` , 另外 `memory_max_target` 定义了最大可以达到的值
注意:如果启用自动内存管理,需要将 `sga_target` 和 `pga_aggregate_target` 设置为 `0`

### 读取参数文件
`pfile`   静态参数文件  
`spfile`  动态参数文件  位置 `E:\Oracle\product\11.2.0\dbhome_1\database` 中 `SPFILEORCL.ORA` 文件



