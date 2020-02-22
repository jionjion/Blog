---
title: MongoDB基础
abbrlink: f20a5c88
date: 2019-01-22 20:19:11
categories:
  - MongoDB
tags: [MongoDB]
---

# 安装
访问网站`https://www.mongodb.com/download-center/community`选择需要的版本
这里是CentOS7系统,因此选择RedHat版本的,官网可以使用`rpm`安装包进行下载.或者提供的`yum`源

## Yum下载
创建yum源`/etc/yum.repos.d/mongodb-org-4.0.repo`写入以下配置
``` bash
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

查看是否配置成功
`yum list | grep mongodb`

进行安装最新版本
`sudo yum install -y mongodb-org`
或者指定固定版本
`sudo yum install -y mongodb-org-4.0.5 mongodb-org-server-4.0.5 mongodb-org-shell-4.0.5 mongodb-org-mongos-4.0.5 mongodb-org-tools-4.0.5`

Linux下可能需要创建数据文件夹,并修改权限
`mkdir -p /data/db && sudo chmod 755 /data/db && sudo chown -R mongod:mongod /data/db
`

## 运行
安装完成后创建用户`mongo`,管理数据库
`service mongod start` 启动服务
`service mongod stop` 关闭服务
 `chkconfig mongod on` 设置开机启动
 `mongo` 进入mongo shell交互


## 目录结构
| 目录,文件 | 说明 |
| --- | --- |
|/etc/mongod.conf | 配置文件|
|/var/lib/mongo | （数据目录）|
|/var/log/mongodb |（日志目录）|

# 命令
在`usr/bin/`目录下,创建了很多小工具`ll /usr/bin/ | grep mongo`可以看到
通过`mongo`命令可以进入到交互界面

## 数据库命令

- `show dbs` 显示当前所有数据库情况
- `db`显示当前操作的数据库或者集合
	- admin： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
	- local: 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
	- config: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。
- `use 数据库名` 表示使用当前数据库;如果数据不存在,则新建
- `db.dropDatabase()` 删除当前数据库

## 集合命令
- ` db.集合.insert(文档)`  为集合插入一条数据,集合不存在则创建
例如`db.news.insert({url: "www.baidu.com"})` 向集合news插入一条数据,或者
`db.news.insert({title:'震惊',tags: ['热点','实时'],like: 100});`
- ` db.集合.save(document)`  为集合插入一条数据,集合不存在则创建
- `db.集合.find()` 查询数据
例如`db.news.find()`,查询全部
- `db.集合.update`更新文档
```
db.collection.update(    
	<query>, 
	<update>, 
	{       
		upsert: <boolean>,   
		multi: <boolean>,  
		writeConcern: <document>
	}
)
```
query : update的查询条件，类似sql update查询内where后面的。
update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
writeConcern :可选，抛出异常的级别。
- `db.集合.save(文档)` 将文档保存,或者替换.
- `db.集合.remove()`删除,不填写条件表示删除全部 
```
db.collection.remove(     
	<query>,      
	{       
		justOne: <boolean>,
		writeConcern: <document> 
	} 
)
```
query :（可选）删除的文档的条件。
justOne : （可选）如果设为 true 或 1，则只删除一个文档。
writeConcern :（可选）抛出异常的级别。

- `db.集合.find()` 查询全部
- `db.集合.find().pretty()`查询全部,并格式化
- `db.集合.findOne()` 只查找一条数据

## 操作符
| 操作符 | 解释 |
| --- | --- |
|`$set`| 更新字段 |
|`$unset`| 删除字段 |
|`$inc` | 自增  `{$inc: {money: 10}} | 自减 {$inc: {money: -10}}` |
|`$exists` |  是否存在 |
|`$in` | 是否在...范围 |
|`$and`| 与 |
|`$or`| 或 |
|`$push` | 向数组中尾部添加一个元素|
|`$addToSet` | 向集合中添加元素 |
|`$pop` | 删除数组中的头部或尾部元素|

## 操作命令练习
连接
`mongo`

当前数据库版本
`db.version();`

退出
`quit();`

查看当前数据库
`show dbs;`

使用当前数据库,不存在则创建
`use test;`

当前数据库状态
`db.stats();`

删除当前正在使用的数据库
`db.dropDatabase()`

查看当前表或者集合
`show tables;`或者`show collections;`

使用创建JS对象,仅存在于当前会话
`var user = {"name":"bao","age":14,address:"上海"};`
查看当前对象
`user`

插入数据
`db.users.insert(user);`

查询一条数据
` db.users.findOne();`
统计当前数据总数
`db.users.count()`
查询符合当前条件的数据,默认查询全部
`db.users.find()`
删除当前集合
`db.users.drop()`

插入数据
`db.users.insertOne({name:"jion",age:10});`
插入多条数据
`db.users.insertMany([{name:"arise",age:13,address:"信阳"},{name:"bao",age:15}]);`
插入数据,指定主键,如果存在主键则报错
`db.users.insert({"_id":1,"name":"Hale"});`

保存数据,指定主键,如果主键存在则更新对应数据
`db.users.save({"_id":1,"name":"Hare"})`
保存多条数据
`db.users.save([{"_id":2,"name":"Hare"},{"_id":3,"name":"Faker"}]);`


