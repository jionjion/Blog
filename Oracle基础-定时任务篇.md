---
title: Oracle基础-定时任务篇
abbrlink: b7efc40d
date: 2017-09-23 12:53:18
categories:
  - Oracle
  - Package
tags: [Oracle, Package]
---
> Oracle 定时器的使用

<!--more-->



## 内容简介

介绍了Oracle数据库中定时任务的创建及启动方式。

## 创建及启动
定时任务的创建：

1. 创建存储过程
2. 创建执行计划,默认创建时开始执行,并返回创建ID

**注意:** 定时任务只能定时执行存储过程,因此需要先声明存储过程,再执行定时任务。

### 创建存储过程
向数据库中`auto_print_log`表插入数据。
``` sql
create or replace procedure auto_print as 
  message varchar2(20);                                                            --声明变量不需要declare关键字
begin
  message := '执行';
  insert into auto_print_log(ID,message) values(auto_print_s.nextval,message);     -- 后台挂起执行插入
  dbms_output.put_line('自动执行语句.....'||message);                              -- 后台挂起不执行消息输出
end;
```

### 创建执行计划
默认计划在创建时即进行执行

``` sql
declare
  job_times number;
begin
  -- 调用系统自动执行函数:参数1:job的主键id,输出参数,代表当前自定义任务  参数2:要执行的存过,注意要加分号  参数3:第一次执行时间 参数4:下次执行时间
  dbms_job.submit(job_times,'auto_print;',sysdate,'sysdate+5/24/60/60');
  dbms_output.put_line('当前准备执行的job的id为:'||job_times);                     -- 输出开始执行的job主键
end;
```

### 查询执行计划
查询当前所有运行的计划,job字段对应唯一编号,broken字段对应运行状态Y(停止)N(开始),interval字段对应运行周期,what字段对应执行的存储过程
``` sql
select * from user_jobs ;
```

### 运行计划
``` sql
declare 
  job_id number ;                                                                  -- 计划编号
  job_pro_name varchar2(20);                                                       -- 计划调用的存过
begin
  job_pro_name := 'auto_print;';                                                   -- 调用的存过名称,注意分号
  select job into job_id from user_jobs where what = job_pro_name;              --获取计划编号
  dbms_job.run(:job_id);
end;
```

### 停止或者推迟执行计划
``` sql
declare 
  job_id number;                                                                   -- 传入Job的ID,代表当前执行的计划任务
begin
  -- 停止计划，不再继续执行 
  dbms_job.broken(:job_id,True); 
  -- 停止计划，并在20秒后继续执行 
  --dbms_job.broken(:job_id,True,Sysdate+(20/24/60/60)); 
end;
```

## 修改执行计划
``` sql
declare 
  job_id number ;
begin
  dbms_job.interval(:job_id, 'sysdate+1/(24*60)');                                 --每分钟执行一次
end;
```

### 删除执行计划
``` sql
declare
  job_id number;
begin
  dbms_job.remove(:job_id);                                                        -- 程序包的执行,默认在begin-end中进行
end;
```
