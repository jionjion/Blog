---
title: Oracle基础-ALC权限
abbrlink: e8351440
date: 2019-02-03 08:56:58
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---

> Oracle 数据库中 ACL 权限控制

<!--more-->



## 内容简介

介绍了Oracle数据库中，对于访问控制权限（ACL）的控制。


## 示例

#### 创建ACL
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

#### 分配ACL给指定用户
```sql
begin
  dbms_network_acl_admin.add_privilege
    (acl        => '/sys/acls/ACLName.xml',  -- 命名
     principal  => 'USER',                   -- 用户
     is_grant   => true,                     -- true表示赋权，false表示取消赋权
     privilege  => 'connect');               -- 权限限制
end;
```

#### 分配访问权限给配置
```sql
begin
  dbms_network_acl_admin.assign_acl 
    (acl => '/sys/acls/ACLName.xml',  -- 命名
     host => '10.18.110.51',          -- 服务器地址
     lower_port => 1,                 -- 端口从
     upper_port => 10000);            -- 端口到
end;
```

#### 删除访问权利从配置
```sql
begin
  dbms_network_acl_admin.unassign_acl
    (acl        => '/sys/acls/ACLName.xml', -- 命名
     host       => '10.18.110.51',          -- 服务器地址
     lower_port => 1,                       -- 端口从
     upper_port => 10000);                  -- 端口到
end;
```

#### 删除用户从ACL
```
begin
  dbms_network_acl_admin.delete_privilege
    (acl       => '/sys/acls/ACLName.xml', -- 命名
     principal => 'USER',                  -- 用户
     is_grant  => true,                    -- true表示赋权，false表示取消赋权
     privilege => 'connect');              -- 权限限制
end;
```

#### 删除ACL

```sql
begin
  dbms_network_acl_admin.drop_acl
    (acl => '/sys/acls/ACLName.xml'); -- 命名
end;
```

#### 相关查询
查询ACL权限分配
`select * from dba_network_acl_privileges ;`

查询ACL权限明细
`select * from dba_network_acls;`