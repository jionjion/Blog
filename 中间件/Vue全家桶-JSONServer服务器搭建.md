---
title: Vue全家桶-JSONServer服务器搭建
abbrlink: 9db1a59c
date: 2019-12-27 10:30:51
categories: Vue
tags: [Vue, JsonService]
---

> 前端模拟后台数据 , 进行双向开发使用.

<!--more-->



# Json Service 服务

更多参考  [官网](https://github.com/typicode/json-server)

## 安装服务

### 全局安装服务
推荐全局安装,便于直接启动.
`npm install -g json-server`



### 开发安装

新建文件夹.并在该文件夹下执行
`npm install json-server --save-dev`
这种方式需要本地配置程序



### 命令

格式
`json-server [options] <source>`



Options可选目录

| 参数                        | 说明                          | 默认值,参数类型                 |
| --------------------------- | ----------------------------- | ------------------------------- |
| `--config, -c`              | 配置文件的路径                | `[default: "json-server.json"]` |
| `--port, -p`                | 端口                          | ` [default: 3000]`              |
| `--host, -H`                | 访问地址,默认本地访问         | `[default: "localhost"]`        |
| `--watch, -w`               | 启动后随数据文件变化而修改    | `[boolean]`                     |
| `--routes, -r`              | 路由配置文件路径              |                                 |
| `--middlewares, -m`         | 中间件文件路径                | `[array]`                       |
| `--static, -s`              | 配置静态文件的路径            |                                 |
| `--read-only, --ro`         | 只读,只允许Get请求            | `[boolean]`                     |
| `--no-cors, --nc`           | 跨域资源共享                  | `[boolean]`                     |
| `--no-gzip, --ng`           | 开启GZIP                      | `[boolean]`                     |
| `--snapshots, -S`           | 快照目录                      | `[default: "."]`                |
| `--delay, -d`               | 延迟响应(ms)                  |                                 |
| `--id, -i`                  | 主键字段名称                  | `[default: "id"]`               |
| `--foreignKeySuffix, --fks` | 设关联外键前缀,默认为自身的id | `[default: "Id"]`               |
| `--quiet, -q`               | 禁止输出日志消息              | `[boolean]`                     |
| `--help, -h`                | 帮助                          | `[boolean]`                     |
| `--version, -v`             | 版本                          | `[boolean]`                     |

更多内容,参考 [官网](https://github.com/typicode/json-server)



## 运行服务



### 配置文件

在当前目录下,创建文件 `json-server.json`.
在文件中,定义需要配置的属性.属性名与参数名一致.
注意JSON格式的完整性,标准双引号属性,不能有注释.

```json
{
	// 主机
	"host": "127.0.0.1",
	// 端口
	"port": 3000,
	// 监控文件变化
	"watch": true,
	// 延迟,毫秒
	"delay": 10,
	// 详细日志
	"quiet": false,
}
```



### 数据文件

在当前目录下,创建文件 `db.json` 

一个标准的JSON对象,用来存储当前数据

```json
{
	"class": {
		"id": 1,
		"room": "1-1",
		"school": "No.1 experimental primary school"
	},
	"student": [
		{
			"id": 1,
			"name": "Jion",
			"age": "25",
			"address": "ShangHai",
			"score":{
				"Math": 100,
				"English": 60
			}
			
		},
		{
			"id": 2,
			"name": "Arise",
			"age": "18",
			"address": "ShangHai",
			"score":{
				"Math": 100,
				"English": 100
			}
		}
	]
}
```



### 启动

在当前目录下,运行.

运行服务,并指定数据文件.  `--watch` 监视文件修改,随文件修改接口变动

`json-server --watch db.json`

在运行时,通过向命令行输入 `s` ,保存当前数据库文件

## 路由规则

- 默认 `id` 字段为自增序列,新增对象时不用传入该属性.
- `post` , `put `, `delete` 请求会触发数据文件的修改,且其请求类型需添加 `Content-Type: application/json`
- 默认路由规则由 `db.json` 中的对象列表匹配.  通过 `--routes` 配合 `routes.json` 文件用以自定义路由



### 默认路由请求

请求路径和对应格式要求如下

| 请求方式 | 路由       | 参数                                     | 说明                        |
| -------- | ---------- | ---------------------------------------- | --------------------------- |
| GET      | /student   |                                          | 查看`student`  路由返回数据 |
| GET      | /student/1 |                                          | 查看`id` 为 1 的数据        |
| POST     | /student   | `{"name": "Boot","age": "20"}`           | 新增一个数据                |
| PUT      | /student/1 | `{"id": 1, "name": "Boot", "age": "20"}` | 更新一个数据                |
| PATCH    | /student/1 |                                          | 批量操作                    |
| DELETE   | /student/1 |                                          | 删除 `id` 为 1 的数据       |



### 高级查询

| 目的        | 请求方式 | 路由                                     | 参数                | 说明                                           |
| ----------- | -------- | ---------------------------------------- | ------------------- | ---------------------------------------------- |
| 过滤        | GET      | `/student?name=Jion&addresss=ShangHai`   | `name` , `address`  | 条件查询,不同属性间条件为`and`                 |
|             | GET      | `/student?id=1&id=2`                     | `id`                | 条件查询,相同属性为 `or` 条件                  |
|             | GET      | `/student?score.English=100`             | `score.English`     | 通过`.`进行深度的条件查询                      |
| 分页        | GET      | `/student?_page=1`                       | `_page`             | 查询第1页,默认分页纪录为 **10**                |
|             | GET      | `/student?_page=1&_limit=3`              | `_page` , `_limit`  | 查询第1页,且分页数为3                          |
| 排序        | GET      | `/student?_sort=id`                      | `_sort`             | 根据`id` 字段,默认升序排列                     |
|             | GET      | `/student?_sort=id&_order=desc`          | `_sort` , `_order`  | 根据`id`字段降序排列                           |
|             | GET      | `/student?_sort=id,name&_order=desc,asc` | `_sort` , `_order`  | 根据`id`字段降序,`name`字段升序排列            |
| 分片        | GET      | `/student?_start=5&_end=10`              | `_start` , `_end`   | 分片,查询区间 [5, 10]                          |
|             | GET      | `_start=0&_limit=10`                     | `_start` , `_limit` | 查询位置0开始,限制10条                         |
| 大/小于等于 | GET      | `/student?id_gte=1&id_lte=2`             | `_gte` , `_lte`     | 在查询属性后追加, 查询`id` 在范围 [1,2] 之中的 |
| 不等于      | GET      | `/student?id_ne=1`                       | `_ne`               | 在查询属性后追加,不等于                        |
| 正则匹配    | GET      | `/student?name_like=J`                   | `_like`             | 在查询属性后追加,模糊查询,支持正则表达式       |
| 全文检索    | GET      | `/student?q=shanghai`                    | `q`                 | 在全文中检索匹配文本,不区分大小写              |
| 数据库      | GET      | `/db`                                    |                     | 查询全部数据                                   |



### 配置路由请求规则



## 动态响应