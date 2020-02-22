---
title: Oracle基础-多线程管理篇
abbrlink: 8f62f15
date: 2017-09-23 12:52:18
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---
------
## 内容简介
Oracle数据中用户调用对象时发生死锁时，通过对用户Session的分析，杀死进程释放数据库资源。

### 程序包调用时发生系统占用

- 查看系统中正在调用该程序包的session
``` sql
select t.sid, t.serial#
from v$session t
where t.sid in (
	select session_id from dba_ddl_locks where name = upper('包名'));
```

- 可能被锁住 查看v$locked
``` sql
select b.sid,b.serial#,b.machine,b.terminal,b.program,b.process,b.status from v$lock a , v$session b
where a.SID = b.SID
```

- 可能被挂起 查看v$session_wait
``` sql
select b.serial#,a.* from v$session_wait a,v$session bwhere a.sid = b.sid
```

- 查看dba_ddl_locks
``` sql
select session_id sid, owner, name, type,mode_held held, mode_requested request from dba_ddl_locks
where name = '&your_package_name'				 
```
