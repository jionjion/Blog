---
title: Oracle基础-权限篇
abbrlink: dc726ee
date: 2017-09-23 12:51:53
categories:
  - Oracle
  - SQL
tags: [Oracle, SQL]
---
> Oracle 权限管理

<!--more-->



# 用户权限

对Oracle中各种权限进行简介。

## 概要文件
数据库概要文件是对各种对象创建时作资源,访问的一组控制的文件

``` sql
-- 概要文件 一组限制用户资源访问的命名文件
create profile zhangqian_profile limit
cpu_per_session 100000
connect_time 60
idle_time 30
sessions_per_user 10
password_lock_time unlimited
password_life_time 60;
-- 修改
alter profile zhangqian_profile limit
cpu_per_session 100000
-- 删除,级联删除
drop profile zhangqian_profile cascade;


-- 查看概要文件
select * from dba_profiles;
-- 创建用户,指定预设文件
create user zhangqian identified by 123456
profile zhangqian_profile;
-- 修改用户,预设文件
alter user zhangqian profile zhangqian_profile;

```

## 用户维护

``` sql
-- 系统用户
select * from dba_users;
-- 数据字典配额
select * from dba_ts_quotas;
-- 修改密码
alter user zhangqian identified by 123456;
-- 账户锁定
alter user zhangqian account lock;
-- 账户解锁
alter user zhangqian account unlock;
-- 账户密码失效
alter user zhangqian password expire;
-- 修改用户配额
alter user zhangqian 
quota 20M on system 
quota 35M on users;
-- 删除用户,级联删除表,索引等对象
drop user zhangqian cascade;
```


## 权限
### 系统权限  
数据库资源操作,创建表,索引等

`public` 表示为公共权限  
`with admin option` 级联授权给其他用户
``` sql
-- 授予权限  
grant XXX to 用户|角色|public [with admin option];
```

查看,回收权限
``` sql
-- 查看用户权限
select * from dba_sys_privs where grantee = 'ZHANGQIAN'
-- 回收权限,级联权限不会被删除
revoke XXX from zhangqian;
```

### 对象权限 
维护数据库对象.由一个用户操作另一个用户对象
``` sql
grant 对象权限 | all | 对象(列)  on 对象 to 用户|角色|public [with admin option]
-- 查看用户对象权限
select * from user_tab_privs_recd where owner = 'ZHANGQIAN';
revoke XXX from zhangqian;
```

## 查看各种权限
- 查看系统权限
``` sql
select * from ROLE_SYS_PRIVS;       
```

- 查看对象权限
``` sql
select * from ROLE_TAB_PRIVS;
```

- 查看各种权限
``` sql
select * from system_privilege_map;
```

- 查看系统默认规则
``` sql
select * from NLS_DATABASE_PARAMETERS;
```

- 查看当前用户拥有的角色

```
select * from user_role_privs;
```

- dba身份查看所有用户的角色

```sql
select * from dba_role_privs;
```

- 查看角色所拥有的权限

```sql
select * from role_sys_privs;
```



## 角色

具有一组权限的集合可以作为一个角色
创建角色
``` sql
create role 角色名
[not identified | identified by 密码;
```
角色查询,维护
```sql
-- 查看角色字典
select * from dba_roles;
-- 查看角色权限
select * from role_sys_privs where role in ('RESOURCE','CONNECT');
-- 修改角色
alter role 角色名 [not identified | identified by 密码];
-- 角色授权
grant XXX to 角色;
-- 角色收权
revoke XXX from 角色;
-- 用户授予角色
grant 角色 to 用户;
revoke 角色 from 用户;
```

## 授权列表


### 授权示例
- 授权用户查看其他用户的表
``` sql
grant select                                                                -- 授权:查询权限
on scott.emp                                                                -- 被授权表的名称
to jion;                                                               		-- 授权用户
```

- 授权debugger
``` sql
-- debug 线程
grant debug connect session to jion;
-- debug 程序包
grant debug any procedure to jion;
```

## 重置密码
如果当数据库忘记密码时,可以这样
1. 打开CMD命令行,输入`sqlplus`进入
2. 在输入的用户名中使用` conn / as sysdba`,即用sysdba登入系统
3. 提示密码直接回车即可,无需输入
4. 然后就可以随意了.比如
5. 解锁,重置`system`用户
```sql
-- 解锁
alter user system account unlock;
-- 重置密码
alter user system identified by 123456;
```



# ACL 权限

介绍了Oracle数据库中，对于访问控制权限（ACL）的控制。


## 操作示例

### 创建ACL

``` sql
begin
  dbms_network_acl_admin.create_acl
    (acl         => '/sys/acls/ACLName.xml',-- 命名
     description => 'some words',           -- 描述
     principal   => 'USER',                 -- 要赋权限的用户
     is_grant    => true,                   -- true表示赋权，false表示取消赋权
     privilege   => 'connect');             -- 权限限制
end;
```

### 分配ACL给指定用户

```sql
begin
  dbms_network_acl_admin.add_privilege
    (acl        => '/sys/acls/ACLName.xml',  -- 命名
     principal  => 'USER',                   -- 用户
     is_grant   => true,                     -- true表示赋权，false表示取消赋权
     privilege  => 'connect');               -- 权限限制
end;
```

### 分配访问权限给配置

```sql
begin
  dbms_network_acl_admin.assign_acl 
    (acl => '/sys/acls/ACLName.xml',  -- 命名
     host => '127.1.1.1',          -- 服务器地址
     lower_port => 1,                 -- 端口从
     upper_port => 10000);            -- 端口到
end;
```

### 删除访问权利从配置

```sql
begin
  dbms_network_acl_admin.unassign_acl
    (acl        => '/sys/acls/ACLName.xml', -- 命名
     host       => '127.1.1.1',          -- 服务器地址
     lower_port => 1,                       -- 端口从
     upper_port => 10000);                  -- 端口到
end;
```

### 删除用户从ACL

```
begin
  dbms_network_acl_admin.delete_privilege
    (acl       => '/sys/acls/ACLName.xml', -- 命名
     principal => 'USER',                  -- 用户
     is_grant  => true,                    -- true表示赋权，false表示取消赋权
     privilege => 'connect');              -- 权限限制
end;
```

### 删除ACL

```sql
begin
  dbms_network_acl_admin.drop_acl
    (acl => '/sys/acls/ACLName.xml'); -- 命名
end;
```

### 相关查询

查询ACL权限分配
`select * from dba_network_acl_privileges ;`

查询ACL权限明细
`select * from dba_network_acls;`



# 数据字典

数据字典表/数据字典视图, 记录了数据库的最基本信息,包括数据字典基本表和数据字典视图

* 数据字典 由 $ORACLE_HOME\RDBMS\ADMIN\sql.bsq 创建 , 用户不能去修改
* 数据字典视图,以$结尾,属于sys用户,存放在system表空间
    分为三类,分别为user_ all_ dba_ 开头
    user_ 当前登录用户的信息
    all_  当前登录用户有权限查看的所有信息
    dba_  数据库管理员可以看到的信息,整个数据库范围的信息

## 动态性能视图

储存数据库每时每刻变化的信息,要是数据库的活动信息.

| 视图                    | 说明                           |
| ----------------------- | ------------------------------ |
| v$parameter             | 初始化参数视图                 |
| v$process               | 当前进程信息                   |
| v$session               | 会话信息                       |
| v$sysstat               | 当前操作会话的系统统计         |
| v$log                   | 控制文件中的日志信息           |
| v$logfile               | 实例重置日志文件名及其位置信息 |
| v$lock                  | 当前进程已获得和请求的锁信息   |
| v$transaction           | 事务相关信息                   |
| v$fixed_view_definition | 所有的动态性能视图的定义信息   |
