---
title: Java基础-Redis操作
abbrlink: 4809f3db
date: 2020-01-30 16:29:42
categories:
  - Java
  - Redis
tags: [Java, Redis, Jedis]
---

> Java代码操作Redis数据库.
>

<!--more-->



## 简介

## 相关介绍

`Redis` 数据库是NoSQL的一种,具有高并发读写,高数据量存储访问,高扩展和高可用的优点,根据访问形式和适用场景

**常用非关系数据库**

| 分类         | 产品       | 典型应用                                 | 数据模型                        | 优点                               | 缺点                            |
| ------------ | ---------- | ---------------------------------------- | ------------------------------- | ---------------------------------- | ------------------------------- |
| 键值对       | Redis      | 内容缓存,处理大量数据的高并发            | 键值对                          | 快速查询                           | 数据结构缺少变化                |
| 列储存数据库 | HBase,Riak | 分布式文件系统                           | 列簇式储存,将同一列数据放在一起 | 查找速度快,扩展性强,分布式扩展容易 | 功能相对局限                    |
| 文档型数据库 | MongDB     | Web应用,键值对数据结构(值的数据结构多样) | 一些列键值对                    | 数据结构要求不严格                 | 查询性能不高,缺乏统一的查询语法 |
| 图形数据库   | InfoGrid   | 社交网络                                 | 图结构                          | 图结构关系算法                     | 对整个图分析后做出结论,分布式差 |

**Windows下安装**

 1. [官网][1]下载msi安装包.
 2. 启动本地服务
	 - 安装目录下执行`redis-server.exe`
	 - 安装目录下执行`Redis-cli.exe –h 127.0.0.1 –p 6379`,表示本地开启,6379为默认访问端口
 3. 将其安装/卸载进入Windows服务组
	 - 安装 `E:\Redis\redis-server.exe --service-install redis.windows.conf --loglevel verbose`   
	 - 卸载 `redis-server --service-uninstall`



## 文件

* `RedisConnect`  IP连接数据库,数据库连接池
* `RedisData`  数据结构存放



## 文件描述

### `RedisConnect`连接数据库

- 下载相应Jar包,使用IP或者Pool进行连接

``` java
public class RedisConnect {

	@Test
	/**使用IP端口进行连接*/
	public void connectByIP() {
		//创建数据库连接,指定IP,端口,超时时间
		Jedis jedis = new Jedis("127.0.0.1", 6379, 2000);
		//保存数据
		jedis.set("name", "张谦");
		//获取数据
		String name = jedis.get("name");
		//打印输出
		System.out.println(name);
		//释放资源
		jedis.close();
	}
	
	@Test
	/**使用连接池进行连接*/
	public void connectByPool() {
		//获得连接池配置对象
		GenericObjectPoolConfig genericObjectPoolConfig = new GenericObjectPoolConfig();
		//设置最大连接数
		genericObjectPoolConfig.setMaxTotal(20);
		//设置最大空闲数
		genericObjectPoolConfig.setMaxIdle(10);
		//创建连接池
		JedisPool jedisPool = new JedisPool(genericObjectPoolConfig, "127.0.01", 6379, 2000);
		//获得核心对象
		Jedis jedis = null;
		try {
			//通过连接池获得
			jedis = jedisPool.getResource();
			//保存数据
			jedis.set("name", "JION.JION");
			//获取数据
			String name = jedis.get("name");
			System.out.println(name);
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			//关闭连接
			if (jedis != null) {
				jedis.close();
			}
			if (jedisPool != null) {
				jedisPool.destroy();
			}
		}
	}
}
```



### `RedisData`数据类型操作

| 数据类型      | 方法                                                         | 说明                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| String类型    | set(String key, String value)                                | 保存数据,修改数据                                            |
|               | get(String key)                                              | 获取数据                                                     |
|               | rename(String oldkey, String newkey)                         | 修改键名                                                     |
|               | del(String key)                                              | 删除键值对                                                   |
|               | append(String key, String value)                             | 追加数据                                                     |
| number类型    | set(String key, String value)                                | 保存数据,修改数据                                            |
|               | get(String key)                                              | 获取数据                                                     |
|               | incr(String key)                                             | 自增+1                                                       |
|               | decr(String key)                                             | 自减-1                                                       |
|               | incrBy(String key, long integer)                             | 增加                                                         |
|               | decrBy(String key, long integer)                             | 减少                                                         |
| hset类型      | hset(String key, String field, String value)                 | 保存数据:属性,对象,属性值                                    |
|               | hmset(String key, Map&lt;String, String&gt; hash)            | 保存Map形式对象                                              |
|               | hincrBy(String key, String field, long value)                | 数值属性增加/减少                                            |
|               | hexists(String key, String field)                            | 判断对象属性是否存在                                         |
|               | hget(String key, String field)                               | 获取对象属性                                                 |
|               | hgetAll(String key)                                          | 获取对象所有属性,返回Map                                     |
|               | hdel(String key, String... fields)                           | 删除对象属性,可以传入String[]                                |
|               | del(String key)                                              | 删除对象                                                     |
| list双向链表  | lpush(String key, String... strings)                         | 左侧压入链表                                                 |
|               | rpush(String key, String... strings)                         | 右侧压入链表                                                 |
|               | lrange(String key, long start, long end)                     | 左侧查询链表,end为负数表示查询全部                           |
|               | lpop(String key)                                             | 弹出左侧头部元素                                             |
|               | rpop(String key)                                             | 弹出右侧头部元素                                             |
|               | llen(String key)                                             | 链表长度                                                     |
|               | lpushx(String key, String... string)                         | 左侧头部压入元素                                             |
|               | rpushx(String key, String... string)                         | 右侧头部压入元素                                             |
|               | lset(String key, long index, String value)                   | 修改链表                                                     |
|               | linsert(String key, LIST_POSITION where, String pivot, String value) | 在指定位置前后加入元素.where参数为BinaryClient.LIST_POSITION的枚举 |
|               | rpoplpush(String srckey, String dstkey)                      | 将前者的链表元素弹出到后者                                   |
| set无重复集合 | sadd(String key, String... members)                          | 添加元素                                                     |
|               | srem(String key, String... members)                          | 删除元素                                                     |
|               | smembers(String key)                                         | 查看元素                                                     |
|               | sismember(String key, String member)                         | 判断元素键值是否存在                                         |
|               | sdiffstore(String dstkey, String... keys)                    | 求多个集合差集,赋值dstkey                                    |
|               | sdiff(String... keys)                                        | 求多个集合差集,返回Map                                       |
|               | sinterstore(String dstkey, String... keys)                   | 求多个集合交集,赋值dstkey                                    |
|               | sinter(String... keys)                                       | 求多个集合交集,返回Map                                       |
|               | sunionstore(String dstkey, String... keys)                   | 求多个集合并集,返回Map                                       |
|               | sunion(String... keys)                                       | 求多个集合并集,返回Map                                       |
|               | scard(String key)                                            | 获取集合数量                                                 |
|               | srandmember(String key)                                      | 随机访问集合                                                 |
| zset有序集合  | zadd(String key, Map&lt;String, Double&gt; scoreMembers)     | 添加元素                                                     |
|               | zscore(String key, String member)                            | 查询元素                                                     |
|               | zcard(String key)                                            | 查询集合数量                                                 |
|               | zrem(String key, String... members)                          | 删除集合                                                     |
|               | zrange(String key, long start, long end)                     | 范围查询                                                     |
|               | zrevrange(String key, long start, long end)                  | 倒叙排序                                                     |
|               | zrangeByScore(String key, double min, double max)            | 排序范围内的元素                                             |
|               | zremrangeByScore(String key, double min, double max)         | 删除指定范围元素                                             |
|               | zincrby(String key, double score, String member)             | 修改指定元素                                                 |
|               | zcount(String key, double min, double max)                   | 查询指定范围内的元素数量                                     |
| 通用操作      | del(String key)                                              | 删除元素                                                     |
|               | exists(String key)                                           | 判断元素是否存在                                             |
|               | rename(String oldkey, String newkey)                         | 重命名                                                       |
|               | expire(String key, int seconds)                              | 设置过期时间                                                 |
|               | ttl(String key)                                              | 查看剩余时间                                                 |
|               | type(String key)                                             | 查看存放的数据类型                                           |
|               | flushAll()                                                   | 清空数据库                                                   |
|               | flushDB()                                                    | 清空子数据库                                                 |

#### 示例代码


``` java
public class RedisData {
	
	public GenericObjectPoolConfig genericObjectPoolConfig = null;
	public JedisPool jedisPool = null;
	public Jedis jedis = null;
	
	//构造方法,在类加载时创建
	public RedisData() {
		genericObjectPoolConfig = new GenericObjectPoolConfig();
		//设置最大连接数
		genericObjectPoolConfig.setMaxTotal(20);
		//设置最大空闲数
		genericObjectPoolConfig.setMaxIdle(10);
		//创建连接池
		jedisPool = new JedisPool(genericObjectPoolConfig, "127.0.01", 6379, 2000);
		//创建核心对象
		jedis = jedisPool.getResource();
	}
	
	
	/**String类型的字符获取*/
	@Test
	public void setOrGetString() {
		//保存数据
		jedis.set("name","张谦");
		System.out.println("保存后获取String类型:"+jedis.get("name"));
		//修改数据,值域
		jedis.set("name","JION.JION");
		System.out.println("修改后String类型:"+jedis.get("name"));
		//追加字符串
		jedis.append("name", ".Love");
		System.out.println("追加字符串后为:"+jedis.get("name"));
		//修改数据,键名
		jedis.rename("name", "NAME");
		System.out.println("修改后,键名为:"+jedis.get("NAME"));
		//删除数据
		jedis.del("NAME");
		System.out.println("删除后,NAME值为:"+jedis.get("NAME"));
	}
	
	/**number类型的字符获取,不存在则创建,并保存为0,默认将字符串解析为数字,但要求为整数型*/
	@Test
	public void setOrGetNumber() {
		//保存数据
		jedis.set("count","0");
		//为其自增
		jedis.incr("count");
		System.out.println("自增后数据值为:"+jedis.get("count"));
		//为其自减
		jedis.decr("count");
		System.out.println("自减后数据值为:"+jedis.get("count"));
		//指定数字增加
		jedis.incrBy("count", 10);
		System.out.println("增加后数据值为:"+jedis.get("count"));
		//指定数字减少
		jedis.decrBy("count", 5);
		System.out.println("减少后数据值为:"+jedis.get("count"));
	}
	
	/**Hash类型的使用哈希集合*/
	@Test
	public void setOrGetHash() {
		//保存数据:属性,对象,属性值
		jedis.hset("name", "student", "张谦");
		jedis.hset("age", "student", "23");
		Map<String, String> teacher = new HashMap<String, String>();
		teacher.put("name", "张小谦");
		teacher.put("age", "23");
		//保存数据:对象,属性键值对
		jedis.hmset("teacher", teacher);
		teacher = null;	//置空,删除对象
		//增加或减少数字
		jedis.hincrBy("age", "student", 10);
		//判断属性是否存在
		jedis.hexists("age", "student");
		//读取数据:属性,对象
		String name = jedis.hget("name", "student");
		System.out.println("获取属性值name:"+name);
		//读取数据:对象
		teacher = jedis.hgetAll("teacher");
		System.out.println("获取属性值name:"+teacher.get("name")
						+"\t获取属性值age:"+teacher.get("age"));
		//删除:属性,对象
		jedis.hdel("name", "student");	//删除student对象的name属性
		System.out.println("删除后的属性值name:"+jedis.hget("name", "student"));
		//删除:对象,属性组
		jedis.hdel("teacher", new String[] {"name","age"});
		System.out.println("获取属性值name:"+jedis.hgetAll("teacher").get("name")
		+"\t获取属性值age:"+jedis.hgetAll("teacher").get("age"));
		//删除整个集合
		jedis.del("teacher");
	}
	
	/**List类型的使用双向链表,多用于消息队列*/
	@Test
	public void setOrGetList() {
		//保存对象
		jedis.lpush("左侧链表", "元素一","元素二","元素三");
		jedis.rpush("右侧链表", "元素一","元素二","元素三");
		//查看列表,左侧按照先后顺序,逆向插入,超过越界后会进行循环查询
		System.out.println("查看左侧列表:"+jedis.lrange("左侧链表", 0, 2));
		//查看列表,右侧按照先后顺序,查看全部
		System.out.println("查看右侧列表:"+jedis.lrange("右侧链表", 0, -1));
		//弹出头部元素,弹出后,不再有该元素
		jedis.lpop("左侧链表");
		jedis.rpop("右侧链表");
		//产看链表长度
		System.out.println("左侧链表长度为:"+jedis.llen("左侧链表"));
		System.out.println("右侧链表长度为:"+jedis.llen("右侧链表"));
		//头部压入元素,如果存在该链表,则压入
//		jedis.lpushx("左侧链表", "追加元素一","追加元素二");
//		jedis.rpushx("右侧链表", "追加元素二","追加元素二");
		//修改
		jedis.lset("左侧链表", 0, "开始值");
		jedis.lset("右侧链表", -1,"结束值");
		//在指定元素前进行插入
		jedis.linsert("左侧链表",BinaryClient.LIST_POSITION.BEFORE, "元素二", "追加元素");
		//将前者链表弹出到后者
		jedis.rpoplpush("左侧链表", "右侧链表");
	}
	
	/**Set类型的,不允许出现重复,最多4294967295个,多用来跟踪唯一性数据;维护数据对象的聚合关系*/
	@Test
	public void setOrGetSet() {
		//添加元素
		jedis.sadd("集合一", "a","b","c","d","e");
		jedis.sadd("集合二", "c","d","e");
		//删除元素
		jedis.srem("集合一", "d","e");
		//查看数据
		System.out.println("元素一:"+jedis.smembers("集合一"));
		System.out.println("元素二:"+jedis.smembers("集合二"));
		//判断是否在元素
		System.out.println("元素一种是否有a:"+jedis.sismember("集合一", "a"));
		//集合差集运算,集合1-集合2
		//将集合1和集合2的差集赋予集合一
//		jedis.sdiffstore("集合一", "集合一","集合二");
		Set<String> set = jedis.sdiff("集合一","集合二");
		System.out.print("差集运算：");
		for (String string : set) {
			System.out.print(string+"\t");
		}
		//集合交集运算,集合1+集合2
		set = jedis.sinter("集合一","集合二");
		//将集合1和集合2的差集赋予集合一
//		jedis.sinterstore("集合一", "集合一","集合二");
		System.out.println();
		System.out.print("交集运算：");
		for (String string : set) {
			System.out.print(string+"\t");
		}
		//集合并集运算,集合1+集合2
		set = jedis.sunion("集合一","集合二");
		//将集合1和集合2的差集赋予集合一
//		jedis.sunionstore("集合一", "集合一","集合二");
		System.out.println();
		System.out.print("并集运算：");
		for (String string : set) {
			System.out.print(string+"\t");
		}
		//获得成员数量
		System.out.println("集合一个数:"+jedis.scard("集合一"));
		//随机访问
		System.out.println("随机访问集合一"+jedis.srandmember("集合一"));
	}
	
	@Test
	/**有序set的使用,游戏排序,索引构建*/
	public void setOrGetSortedSet() {
		//增加
		Map<String, Double> set = new HashMap<String,Double>();
		set.put("张三", 95D);set.put("李四", 98D);set.put("王五", 97D);set.put("赵六", 94D);
		jedis.zadd("排序", set);
		//查询
		System.out.println("张三分数:"+jedis.zscore("排序", "张三"));
		//查询当前集合数量
		System.out.println("当前集合数量:"+jedis.zcard("排序"));
		//删除
//		jedis.zrem("排序", "张三","李四");
		//范围查找
		jedis.zrange("排序", 0, -1);		//插叙到最后一个的集合
		//从小到大排序
		jedis.zrevrange("排序", 0, -1);	//倒叙排列
		//按照范围进行删除
		jedis.zremrangeByScore("排序", 0, 4);	//删除0~4范围的
		//按照分数显示前两个
		jedis.zrangeByScore("排序", 0, 100);	
		//指定元素修改
		jedis.zincrby("排序", 3d, "李四");
		//查询指定段的数量
		jedis.zcount("排序", 95, 100);
	}
	
	@Test
	/**通用操作*/
	public void commonOperate() {
		//删除
		jedis.del("对象一");
		//判断否存在
		jedis.exists("对象二");
		//重命名
		jedis.rename("对象一", "新对象一");
		//设置过期时间,单位s
		jedis.expire("对象二", 100);
		//所剩时间
		jedis.ttl("对象二");
		//获取key的类型
		jedis.type("排序");		//返回排序哈希类型
		//清空数据库
		jedis.flushAll();
		//清空当前子数据库
		jedis.flushDB();
	}
}

```



### 事务操作

Redis数据库默认由16个数据库集合构成

- `Multi`:开启事物
- `Exec`:执行语句
- `Discard`:回滚事物



### 持久化本地

将数据从内存到硬盘的过程
**RDB过程**	
每隔一定时间,将快照写入磁盘.只有一个文件,方便恢复转移;不适合及时性和大量性
配置修改安装目录下的,`redis.windows.conf`文件.下面表示每多少秒内有多少发生写入后,进行快照的记录
在136行处添加
``` 
save 900 1
save 300 10
save 60 10000
```
保存位置为当前下文件夹下的dump.rdb文件
182处查看
``` 
dir ./
```

**AOF方式**
日志记录每个过程.更安全,每次变化时进行同步保存;日志为追加格式,避免荡机时未写完;日志过大时,进行重写.
效率更低,文件更大
配置修改,默认没有打开,改为yes即可.产生文件`appendonly.aof`

在509行处
``` 
appendonly no
```

同步策略,每修改后会进行同步.当不使用不同时,仅作为缓存数据库
在573行处

``` 
# appendfsync always
appendfsync everysec
# appendfsync no

```



[1]: https://redis.io/