---
title: Redis基础
typora-root-url: ../../
abbrlink: fe29e710
date: 2019-01-22 21:09:38
categories:
  - Redis
tags: [Redis]
---

> Redis 操作记录

<!--more-->



# 安装

官网下载对应的tar包,然后解压,编译,安装

```bash
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
tar xzf redis-5.0.3.tar.gz
cd redis-5.0.3
make
make install
```

安装完成后,将目录转为到合适的位置
递归创建文件夹
`mkdir -p /usr/local/redis/bin`
`mkdir -p /usr/local/redis/ect`
移动配置文件
`mv /home/jionjion/下载/redis-5.0.3/redis.conf /usr/local/redis/ect/`
移动src目录下的命令文件
`mv redis-benchmark redis-check-aof  redis-cli redis-server /usr/local/redis/bin`
创建命令启动别名,指定启动的配置文件
`alias redis='redis-server /usr/local/redis/ect/redis.conf'`

## 命令
- 启动 自建命令`redis`;
- 启动 系统命令`redis-server /usr/local/redis/ect/redis.conf`,指明启动的配置文件
- 关闭 `redis-cli shutdown nosave` 或者 `redis-cli shutdown`

## 远程连接并作为服务
修改配置文件
注释本地IP,并允许远程连接,并允许后台运行,设置登录密码
``` bash
# bind 127.0.0.1
protected-mode no
daemonize yes
requirepass 123456
```
将redis服务脚本`/usr/local/redis/utils/redis_init_script`移动到   /etc/init.d目录下
`cp /usr/local/redis/utils/redis_init_script /etc/init.d/`
并修改名称
`mv /etc/init.d/redis_init_script /etc/init.d/redis`

修改配置信息
`vim /etc/init.d/redis`
修改为如下信息
```bash
### BEGIN INIT INFO
# Provides:     redis_6379
# Default-Start:        2 3 4 5
# Default-Stop:         0 1 6
# Short-Description:    Redis data structure server
# Description:          Redis data structure server. See https://redis.io
### END INIT INFO

REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
# CONF="/etc/redis/${REDISPORT}.conf"
CONF="/usr/local/redis/ect/redis.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    status)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running"
        else
                echo "Service not running !"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

为文件增加执行权限
`chmod +x /etc/init.d/redis`
加入开机启动

 

##  业务使用场景
数据库,缓存,消息队列,批量任务的中间结果


# 数据库操作
手册参考网站`http://redisdoc.com/`,仅对自己常用的进行记录

如果配置了密码,则需要在登录后执行 `auth password`
或者使用登录命令
 `redis -cli -h 127.0.0.1 -p 6379 -a <your_password>`

## `STRING` 操作

将对象序列化为二进制,存在缓存中.
线程安全,自带原子性

### `SET` 
语法 `SET key value [EX seconds] [PX milliseconds] [NX|XX]`
将数据以键值对的形式存入

 - `key`为字符型,`value`格式不限制
例如:`set name "jionjion"` 将jionjion存入,key为name
- `[EX seconds]`过期时间,单位秒
例如:`set name "jionjion" ex 5` 将jionjion存入,key为name,且过期时间为5秒
- `[PX milliseconds] `设置过期时间,单位毫秒
例如:`set name "jionjion" px 5000` 将jionjion存入,key为name,且过期时间为5秒
- `[NX]` 当key不存在时,存入;否则报错.类似新增操作(set if not exists)
例如:`set name "jionjion" NX` 当name不存在,新增
- `[XX]` 当key存在是,更新;否则不做处理.类似更新操作(set  exists)
例如:`set name "jionjion" XX` 当name存在,更新


### `GET`
语法 `GET key`
通过键,获得以键值对形式存在的值.如果值不存在,则返回`nil`;另外,键`key`只能为字符串.

- `key`只能为字符型,返回`value`不做限制
例如:`get name` 获得name对应的值

### `GETSET`
 语法 `getset key value`
 将`key`对应的值替换为`value`,并将替换之前的`value`返回.如果是新建,则返回`nil`

 - `key`只能为字符型,`value`格式不做限制,
 例如:`getset name "jionjion"` 将`key`为name的值,替换为`jionjion`,并返回原来的值

### `STRLEN` 
 语法 `STRLEN key`
 获得指定`key`对应的`value`的长度,仅限在`value`类型为字符类型时才有长度,否则返回0

 - `key`只能为字符串,`value`不做限制,但仅有字符类型有长度,否则返回0
 例如:`strlen name` 获得`name`对应的值的长度

### `APPEND` 
语法 `APPEND key value`
如果`value`存在,则将其拼在原有的值后面,不存在则存入新值.并返回追加后的长度.

- `key`只能为字符类型
例如:`append name " is student"` 将字符串追加到值的后面,并返回当前总共长度.

### `SETRANGE`
语法 `SETRANGE key offset value`
其中,`key`为字符类型,`offset`为非负数,`value`为字符类型.其目的是将`key`所对应的值的`offset`位置以后的内容替换为`value`所对应的值,类似于截取替换.
当`key`不存在时,将`value`写入.
当`offset`大于字符串的最大长度时,填充`\x00`
当`offset`过大时,会消耗过多内容,容易造成阻塞!

- `key`只能值为字符串,`offset`非负数,`value`字符串
例如`setrange name 0 "jionjion"` 将`name`对应的值,从开始位置,字符替换为`jionjion`;当`name`不存在时,直接写入

### `GETRANGE` 
语法 `GETRANGE key start end`
其中`key`为字符类型,`start`和`end`类型为数字,正数表示位置从左往右,负数表示从右往左,`-1`为最后一个字符的位置.`start`和`end`的过大或者过小只会返回到极限位置对应的字符串

- `key`为字符串,`start`和`end`必须为数字
例如`getrange name 0 1`获得`name`对应的字符串的[0,1]位置的字符,即前两个字符串

- `start`和`end`为负数时,表示从右往左索引
例如:`getrange name -2 -1` 获得后两个字符串

- `start`和`end`过大或者过小,超过部分会被忽略,不会影响返回字符串的内容,最大返回原字符串
例如:`getrange name -100 100` 获得指定区间内的字符串,过大或过小,不会影响原字符串的格式
例如:`getrange name 0 -1` 获得整个字符串

### `INCR` 
语法 `INCR key`
其中,`key`对应的值必须为数字类型,表示并将其自增+1.当`key`不存在时,默认返回1
注意,`key`对应的值在redis中以数字字符串进行保存!
注意,值必须为整数,否则报错.

- `key`对应的值必须为数字
例如:`incr age` 对`age`对应值进行自增,不存在返回1

### `INCRBY` 
语法 `INCRBY key increment`
其中,`key`对应的值必须为整数类型,`increment`为整数,表示将`key`对应的值其加上`increment`增量

- `key`为字符串,`increment`为整数
例如:`incrby age 10` 将`age`对应的值增加10
- `increment`可以为负数,表示相减
例如:`incrby age -10` 将`age`对应的值减少10

### `INCRBYFLOAT`
语法 `INCRBYFLOAT key increment`
其中,`key`对应的值必须为数字,但不限制精度.`increment`为数字类型,不限精度.表示对`key`对应的相加`increment`增量.
注意:存储时会自动将尾数抹零.

- `key`对应的值必须为数字,精度不限.`increment`可以为负数,表示相减
例如:`incrbyfloat age 1.5` 将`age`对应的值增加1.5

### `DECR`
语法 `DECR key`
将`key`对应的值进行自减.
注意,值必须为整数,否则报错.


### `DECRBY`
语法 `DECRBY key decrement`
其中,`key`对应的值必须为整数,`decrement`必须也为整数.表示`key`对应的值减去`decrement`增量

### `MSET`

语法`MSET key value [key value …]`
其中,`key`为字符类型,`value`不做限制,表示对一个或者多对键值对进行赋值,如果`key`重复,则更新对应的`value`

- `key`字符类型,`value`不做显示,`key`重复则更新
例如:`mset name jionjion age 10`

### `MSETNX`
语法 `MSETNX key value [key value …]`
其中,`key`为字符类型,`value`不做限制.当`key`不存在时,执行赋值操作.其中,如果任意一个`key`存在,则整个赋值操作执行失败.事务回滚
返回1,表示执行成功;返回0,表示执行失败.

- `key`字符类型,`value`不做显示,`key`重复则更新
例如:`msetnx name jionjion age 10`

### `MGET`
语法 `MGET key [key …]`
其中,`key`为字符类型.获得一个或者多个`key`对应的值.不存在返回`nil`

- `key`字符类型
例如:`mget name age`.获得`name`和`age`对应的值

## `HASH` 操作

`STRING` 类型的 `key` 和  `value` ,适合存储键和值类型的数据.

### `HSET`
语法 `HSET hash field value`
其中,`hash`为字符类型,代表哈希表的名字,`field`为域,同为字符类型,`value`为域中存放的值,类型不限
当不存这个`hash`或者`field`时,则自动创建;`field`重复时,则更新`value`.
返回,如果创建成功,返回1.更新则返回0

- `hash` 字符类型, `field`字符类型, `value`类型不限
例如`hset user name "jionjion"` 创建`user`的哈希表,将域`name`的值设置为`jionjion`.

### `HSETNX`
语法 `HSETNX hash field value`
其中,`hash`为字符类型,代表哈希表的名字,`field`为域,同为字符类型,`value`为域中存放的值,类型不限
当`field`不存在时,则新增`value`如果`field`存在,则放弃操作
返回,创建成功返回1;更新失败返回0

- `hash` 字符类型, `field`字符类型, `value`类型不限
例如`hsetnx user name "jionjion"` 更新`user`的哈希表,将域`name`的值设置为`jionjion`.如果存在,则放弃操作.

### `HGET`
语法 `HGET hash field`
其中,`hash`为字符类型,为已存在的哈希表,`field`字符类型,哈希表对应的域.命令为获得指定哈希表中的字段的值.
返回,如果哈希表或则字段不存在,则返回`nil`.

- 获得指定哈希表的字段值
例如 `hget user name`

### `HEXISTS`
语法 `HEXISTS hash field`
检查`hash`哈希表中是否含有`field`域.其中,`hash`字符类型,哈希表的名字.`field`字符类型.
返回,如果存在返回1;否则返回0

- 判断是否含有指定字段
例如:`hexists user name` 判断`user`表中是否含有`name`域

### `HDEL`
语法 `HDEL key field [field …]`
删除`key`对应哈希表的一个或者多个`field`域.`key`字符类型,`field`字符类型
如果`field`域不存在,不影响
返回,被移除的域数量

- 删除指定字段
例如:`hdel user name age` 删除`user`哈希表中的域`name`和`age`

### `HLEN`
语法 `HLEN key`
获得当前哈希表中的域值数量.
返回,当前`key`对应的哈希表不存在,或者没有域,返回0

- 获得域的数量
例如`hlen user`

### `HSTRLEN`
语法 `HSTRLEN key field`
获得`key`哈希表中`field`域的字符串的值的长度.
返回,当前`key`或者`field`不存在,返回0

- 获得指定哈希表中域值的长度
例如`hstrlen user name`

### `HINCRBY`
语法 `HINCRBY key field increment`
为`key`哈希表中的`field`域的值加上增量`increment`,其中`field`对应的值必须为整数,不限正负,若不存在则创建,`increment`也必须为整数,正数为加上增量,负数为减去增量

- 指定字段增加增量
例如 `hincrby user age 10`
- 指定字段减去减量
例如 `hincrby user age -100` 结果可以为负数

### `HINCRBYFLOAT`
语法 `HINCRBYFLOAT key field increment`
为`key`哈希表中的`field`域加上增量`increment`,其中`field`对应的值必须为数字,不限制正负数.`increment`也必须为数字.程序会为运算后的结果自动抹去尾零,并以数字字符串的形式存入.

- 指定字段增加
例如 `hincrbyfloat user age 1.1`	

### `HMSET`
语法 `HMSET key field value [field value …]`
为`key`哈希表中,创建多个`field`域并赋值.如果`field`域已经存在,则覆盖值;如果`field`域不存在,则创建并赋值

- 创建多个域
例如 `hmset user name "jionjion" age 20`

### `HMGET`
语法 `HMGET key field [field …]`
获得`key`哈希表中的多个`field`的值.如果`field`值不存在,则返回`nil`.

- 获得多个域
例如 `hmget user name age`

### `HKEYS`
语法 `HKEYS key`
获得`key`哈希表中所有的`field`的信息.`key`不存在时,返回空集合.

- 获得所有的域
例如 `hkeys user`

### `HVALS`
语法 `HVALS key`
获得`key`哈希表中的所有域的值.`key`不存在时,返回空集合.

- 获得所有的值
例如 `hvals user`


### `HGETALL` 
语法 `HGETALL key`
获得`key`哈希表中的所有域和值,如果`key`不存在,返回空集合.

- 获得所有域和值
例如 `hgetall user`

### `HSCAN`
语法 `HSCAN key cursor [MATCH pattern] [COUNT count]`

暂定!

## `LIST` 操作

### `LPUSH`
语法 `LPUSH key value [value …]`
将一个或者多个`value`值顺次插入到列表`key`的头前.`key`必须为列表类型,否则报错;如果`key`列表不存在,则自动创建.`key`列表的顺序由插入时的顺序决定,并且允许重复插入.
返回修改后的列表长度

- 顺次插入列表多个值
例如 `lpush users jion arise`

### `LPUSHX`
语法 `LPUSHX key value [value …]`
将一个或者多个`value`值顺次插入到列表`key`的头前.`key`必须存在且为列表类型,否则不做任何处理.
返回修改后的列表长度

- 对已经存在列表顺次插入多个值
例如 `lpushx users hacket packie`

### `RPUSH`
语法 `RPUSH key value [value …]`
将一个或者多个`value`值顺次追加到列表尾.`key`必须为列表类型,否则报错.如果`key`列表不存在,则自动创建.
返回修改后的列表长度

- 对一个列表追加多个值
例如 `rpush users jion arise`

### `RPUSHX`
语法 `RPUSHX key value [value …]`
将一个或者多个`value`值顺次追加到列表为,`key`必须存在且为列表类型,否则不做任何修改.
可以重复相同值

- 对存在的列表追加值
例如 `rpushx users jion arise`

### `LPOP`
语法 `LPOP key`
将列表头元素删除,并返回.`key`不存在或者列表为空时,返回`nil`.

- 弹出列表第一个元素
例如 `lpop uses`

### `RPOP`
语法 `RPOP key`
将列表尾元素删除,并返回.`key`不存在或者列表为空时,返回`nil`.

- 弹出列表最后一个元素
例如 `lpop users`

### `RPOPLPUSH`
语法 `RPOPLPUSH source destination`
`source`和`destination`必须列表类型,命令将`source`列表的最后一个元素弹出,并返回给客户端.同时,将弹出的元素重新插入到`destination`列表的第一个位置.
`source`列表如果不存在,则不进行动作.
`destination`列表如果不存在,则创建.
`source`和`destination`可以为同一个列表,此时命令等同于`rotation`旋转操作.
整体操作具有原子性,列表修改同时成功或者同时失败.

- 将列尾弹出,并插入到另一个列表的列首
例如 `rpoplpush users class`

- 将列尾弹出,并放置到列首
例如 `rpoplpush users users`

### `LREM`
语法 `LREM key count value`
`key`必须为列表,`count`必须为整数,其绝对值表示将要从`key`列表中移除的`value`的个数,当`count`为正整数时,表示从左往右移除指定`count`个与`value`相同的元素;`count`为负数时,移除顺序相反;`count`为0,表示移除全部
返回,被移除的数量

- 从左往右移除指定个数的值
例如 `lrem users 1 jionjion`

- 从右往左移除指定个数的值
例如 `lrem users -1 jionjion`

- 移除全部的值
例如 `lrem users 0 jionjion`

### `LLEN`
语法 `LLEN key`
返回类表的长度,如果列表不存在,或者为空列表,则返回0

- 获得列表长度
例如 `llen users`

### `LINDEX`
语法 `LINDEX key index`
返回列表`key`中,索引位置为`index`的元素.其中,`key`必须为列表类型,`index`为必须为整数,正整数表示索引从前往后,0为列表第一个元素;负整数表示索引从后往前,-1为列表最后一个元素.如果索引超过列表长度,返回`nil`.

- 获得列表第一个元素
例如 `lindex users 0`

- 获得列表最后一个元素
例如 `lindex users -1`

### `LINSERT`
语法 `LINSERT key BEFORE|AFTER pivot value`
将`value`值插入到列表`key`的最先匹配的值`pivot`的`before`前面或者`after`后面.
返回,插入成功返回修改后列表长度;没有匹配的`pivot`返回-1,列表`key`为空返回0

- 在指定值前面插入一个元素
例如 `linsert users before jion arise`
- 在指定值后面插入一个元素
例如 `linsert users after jion arise`

### `LSET`
语法 `LSET key index value`
将列表`key`的索引位置为`index`的元素的值替换为`value`.

- 替换列表第一个元素的值
例如 `lset users 0 jion`

- 替换列表最后一个元素的值
例如 `lset users -1 jion`

### `LRANGE`
语法 `LRANGE key start stop`
返回列表`key`中,从`start`到`stop`区间内的元素,索引从0开始,表示第一个元素;-1表示最后一个元素.`start`或`stop`过小或者过大均自动截取列表的极值

- 获得列表前两个元素
例如 `lrange users 0 1`
- 获得列表后两个元素
例如 `lrange users -2 -1`
- 获得列表全部元素
例如 `lrange users 0 -1`

### `LTRIM`
语法 `LTRIM key start stop`
对列表`key`进行修剪,只保留`start`和`stop`的区间内的元素.`start`和`stop`超过列表长度,返回错误.

- 修剪,保留前两个
例如 `ltrim users 0 1`
- 截取,保留后两个
例如 `ltrim users -2 -1`

### `BLPOP`
语法 `BLPOP key [key …] timeout`
阻塞式弹出列表`key`的列首第一个元素,如过列表为空,等待`timeout`秒,如果`timeout`设置为0,表示一直等待到列表具有元素时才进行弹出.
当有多个`key`时,会根据`key`的先后顺序,检测每一个列表,直到有一个能弹出列首元素为止.
返回,被弹出的列表名称和弹出的元素

- 持续等待,直到弹出列首
例如 `blpop users 0`

- 等待多个列表10秒,直到弹出列首
例如 `blpop users names 10`

### `BRPOP`
语法 `BRPOP key [key …] timeout`
阻塞式弹出列表`key`的列尾最后一个元素,如过列表为空,等待`timeout`秒,如果`timeout`设置为0,表示一直等待到列表具有元素时才进行弹出.
当有多个`key`时,会根据`key`的先后顺序,检测每一个列表,直到有一个能弹出列尾元素为止.
返回,被弹出的列表名称和弹出的元素

- 持续等待,直到弹出列尾
例如 `brpop users 0`

- 等待多个列表10秒,直到弹出列尾
例如 `brpop users names 10`

### `BRPOPLPUSH`
语法 `BRPOPLPUSH source destination timeout`
将`source`列表的列尾元素弹出,并插入`destination`列表的列首,如果`source`列表为空,则等待`timeout`秒,`timeout`为0,则持续等待.
`source`和`destination`可以为同一个列表.
返回,弹出的元素和等待的时间

- 等待10秒,从列表弹出到另一个列表
例如 `brpoplpush users names 10`

- 循环等待10秒,将列尾弹出并放置在列首
例如 `brpoplpush users uesrs 10`

## `SET` 集合

### `SADD`
语法 `SADD key member [member …]`
将一个或者多个`member`成员加入到集合`key`当中,已经存在于集合的将会被忽略.如果`key`不存在,则创建;如果`key`不为集合类型,则报错.
返回,成功添加的成员个数.

- 添加成员`jion`到集合`users`中
`sadd users jion`

- 添加多个成员到集合`users`中
`sadd users users arise`

### `SISMEMBER`
语法 `SISMEMBER key member`
判断成员`member`是否为集合`key`的成员.
返回,如果属于该集合,返回1;如果不存在或者`key`不是集合,返回0

- 判断成员`jion`是否为`users`集合成员
`sismember users jion`

### `SPOP`
语法 `SPOP key [count]`
从集合`key`中随机移除`count`个成员,并返回.当集合不存在时,返回`nil`.

- 随机弹出`user`集合1个成员
例如 `spop users`

- 随机弹出`user`集合的2个成员
例如 `spop users 2`

### `SRANDMEMBER`
语法 `SRANDMEMBER key [count]`
从集合`key`中随机获得指定个数的成员.若集合为空,则返回`nil`.
`count`为非零整数,若`count`为正数,则随机返回指定个数的成员,如果`count`大于集合长度,则最多返回集合全部成员;如果`count`为负数,同返回指定个数的成员,如果`count`绝对值大于集合长度,则随机填充成员并返回指定长度的成员.

- 随机获得一个集合成员
例如 `srandmember users`

- 随机获得多个集合成员,若`count`大于集合长度,则返回全部集合全部成员
例如 `srandmember users 10`

- 随机获得指定长度的集合成员,若`count`绝对值大于集合长度,则返回随机填充到指长度的集合成员
例如 `srandmember users -10`

### `SREM`
语法 `SREM key member [member …]`
移除集合`key`中的一个或者多个成员,不存在的`member`则忽略.
返回,被移除的成员数量.

- 移除单个成员
例如 `srem users jion`
- 移除多个成员
例如 `srem users jion arise`

### `SMOVE`
语法 `SMOVE source destination member`
将`member`成员从`source`集合删除,并添加到`destination`集合中.两个集合必须存在.
如果集合`source`不存在`member`成员,则操作结束.
如果集合`source`和`destination`同时存在`member`成员,则只删除`source`中的`member`成员,`destination`不做处理.
返回值,成功返回1;失败返回0.

- 移除`users`集合中指定成员`jion`到另一个集合`group`中
例如 `smove users group jion`

### `SCARD`
语法 `SCARD key`
获得集合`key`中的成员数量.如果集合不存在或者为空,返回0

- 获得集合成员数量
例如 `scard users`

### `SMEMBERS`
语法 `SMEMBERS key`
获得集合中的所有成员,`key`或者为空,返回错误

- 获得集合所有成员
例如 `smembers users`

### `SSCAN`
语法 `SSCAN key cursor [MATCH pattern] [COUNT count]`


### `SINTER`
语法 `SINTER key [key …]`
获得多个`key`集合之间的交集.当`key`存在空集时,返回空集

- 获得多个集合的交集
例如 `sinter users1 users2`

### `SINTERSTORE`
语法 `SINTERSTORE destination key [key …]`
获得多个`key`集合之间的交集,并将结果赋值给集合`destination`.如果集合`destination`存在,则覆盖.
返回,交集中的成员个数

- 获得`users1`,`users2`多个集合的交集,并赋值给集合`users`
例如 `sinterstore users users1 users2`

### `SUNION`
语法 `SUNION key [key …]`
获得多个`key`集合共同的并集,并返回集合对象

- 获得多个集合的并集
例如 `sunion users1 users2`

### `SUNIONSTORE`
语法 `SUNIONSTORE destination key [key …]`
获得多个`key`集合共同的并集,并赋值给`destination`.如果集合`destination`存在,则覆盖.

- 获得`users1`,`users2`多个集合的并集,并赋值给集合`users`
例如 `sunionstore users users1 users2`

### `SDIFF`
语法 `SDIFF key [key …]`
获得`key`集合之间的差集,根据`key`的顺序依次相减,并返回差集成员.

- 获得`users1`,`users2`集合的差集
例如 `sdiff users1 users2`

### `SDIFFSTORE`
语法`SDIFFSTORE destination key [key …]`
获得多个`key`集合的差集,根据`key`的顺序相减,并赋值给`destination`.如果集合`destination`存在,则覆盖.

- 获得`users1`,`users2`集合的差集
例如 `sdiffstore users users1 users2`

## `ZSET` 有序集合

### `ZADD`
语法 `ZADD  [NX|XX] [CH] [INCR] score member [score member …]`
将成员`member`和指定排序顺序`score`放入有序集合`key`中.其中,`score`必须为数字,精度不限,正负不限,越小,排序越靠前.`score`数字相同,则根据字典字母A-Z顺序排序.

- 插入有序
例如 `zadd result 5 jion`

### `ZSCORE`
语法 `ZSCORE key member`
返回成员`member`在有序集合`key`中的顺序.如果`key`不是有序集合,或者`key`不存在,返回`nil`

- 获得集合中成员的顺序
例如 `zscore result jion`

### `ZINCRBY`
语法 `ZINCRBY key increment member`
为有序集合`key`中的成员`member`的`order`顺序加上增量`increment`.其中,`key`必须为有序集合,`increment`增量可正可负,精度不限.
返回,修改后的成员所在集合的位置

- 增加集合成员顺序
例如 `zincrby result 1 jion`
- 减少集合成员顺序
例如 `zincrby result -0.5 jion`

### `ZCARD`
语法 `ZCARD key`
返回有序集合`key`的长度.

- 获得集合`key`的长度
例如 `zcard result`

### `ZCOUNT`
语法 `ZCOUNT key min max`
获得有序集合`key`中,排序值在`min`和`max`闭区间内的成员数量.`min`和`max`必须为数字,大小不限制.

- 获得指定区间内成员数量
例如 `zcount result -10 10`

### `ZRANGE`
语法 `ZRANGE key start stop [WITHSCORES]`
获得有序集合`key`的指定索引区间内的成员.索引从0开始,-1表示最后一个.当`start`和`stop`超出索引边界后,只回返回对应边界的极值所在的值.可选参数`withscores`表示返回对应成员的排序大小.
返回值,按照排序值,从小到大排列.相同排序值成员根据字典顺序排序.


- 获得有序集合的前两个成员
例如 `zrange result 0 -1`

- 获得有序集合的后两个成员
例如 `zrange result -1 -2`

- 获得所有的有序集合内的成员,并返回对应的排序大小
例如 `zrange result 0 -1 withscores`

### `ZREVRANGE`
语法 `ZREVRANGE key start stop [WITHSCORES]`
获得有序集合`key`的指定索引区间内的成员.索引从0开始,-1表示最后一个.
当`start`和`stop`超出索引边界后,只回返回对应边界的极值所在的值.可选参数`withscores`表示返回对应成员的排序大小.
返回值,按照排序值,从大到小排列.相同排序值成员根据字典顺序排序.

- 获得所有有序集合内的成员,并返回对应的排序大小,排序从大到小
例如 `zrevrange result 0 -1 withscores`

### `ZRANGEBYSCORE`
语法 `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`
获得有序集合`key`中,所有的成员的排序值在`min`和`max`闭区间内的成员.
可以通过`(`,`)`限制区间为开区间.
返回结果根据排序值从小到大排列,相同的成员根据字典顺序排列.

- 获得大于0,小于等于10的排序值的成员
例如 `zrangebyscore result (0 10`

- 获得大于0,小于10的排序值的成员,及其排序值
例如 `zrangebyscore result (0 (10 withscores`

- 获得大于负无穷,小于正无穷的显示整个集合
例如 `zrangebyscore result -inf +inf withscores`

### `ZREVRANGEBYSCORE`
语法 `ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]`

获得有序集合`key`中,所有的成员的排序值在`min`和`max`闭区间内的成员.
可以通过`(`,`)`限制区间为开区间.
返回结果根据排序值从大到大小排列,相同的成员根据字典顺序排列.

### `ZRANK`
语法 `ZRANK key member`
获得有序集合`key`的成员`member`的排名位置.排序规则为排序值从小到大排列.排名从0开始.如果成员`member`不存在,则返回`nil`

- 获得成员`jion`在有序集合`result`的正序排名
例如 `zrank result jion`

### `ZREVRANK`
语法 `ZREVRANK key member`
获得有序集合`key`的成员`member`的排名位置.排序规则为排序值从大到小排列.排名从0开始.如果成员`member`不存在,则返回`nil`

- 获得成员`jion`在有序集合`result`的降序排名
例如 `zrevrank result jion`

### `ZREM`
语法 `ZREM key member [member …]`
移除有序集合`key`中一个或者多个成员`member`,若该成员不存在,则忽略.
返回被移除的成员个数

- 移除单个成员
`zrem result jion`

- 移除多个成员
`zrem result jion arise`

### `ZREMRANGEBYRANK`
语法 `ZREMRANGEBYRANK key start stop`
移除有序集合`key`中指定排名`rank`在`start`和`stop`闭区间内的所有成员.`start`从0开始,表示第一个元素;`-1`表示最后一个元素.
返回被移除成员的数量

- 移除第一个,第二个成员
例如 `zremrangebyrank result 0 1`

- 移除最后一个成员
例如 `zremrangebyrank result -1 -1`

- 移除全部成员
例如 `zremrangebyrank result 0 -1`

### `ZREMRANGEBYSCORE`
语法 `ZREMRANGEBYSCORE key min max`
移除有序集合`key`中排序值`score`在`min`和`max`闭区间内的所有成员.

- 移除排序值在闭区间内的所有成员
例如 `zremrangebyscore result -100 100`

### `ZRANGEBYLEX`
语法 `ZRANGEBYLEX key min max [LIMIT offset count]`
当有序集合`key`具有相同的排序值`score`时,根据字母的先后顺序进行排列,命令检索`min`字母到`max`字母之间的成员.
必须使用区间限定符,`(`和`)`表示开区间;`[`和`]`表示闭区间;`+`和`-`表示极大值和极小值

- 获得从`a`到`c`的,左开右闭的成员
例如 `zrangebylex result (a [c`

- 获得到`ccc`的所有成员
例如 `zrangebylex result - [ccc`

### `ZLEXCOUNT`
语法 `ZLEXCOUNT key min max`
当有序集合`key`具有相同的排序值`score`时,根据字母的先后顺序进行排列,命令返回在`min`字母和`max`字母之间的成员总数.

- 获得`a`到`c`的,左开右闭的成员总数
例如 `zlexcount result (a [c`

- 获得到`ccc`的成员个数
例如 `zlexcount result - [ccc`

### `ZREMRANGEBYLEX`
语法 `ZREMRANGEBYLEX key min max`
当有序集合`key`具有相同的排序值`score`时,根据字母的先后顺序进行排列,命令移除在`min`字母和`max`字母之间的成员,并返回被移除的成员数量

### `ZSCAN`


### `ZUNIONSTORE`
语法 `ZUNIONSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]`
集合并集并保存
返回新集合的长度

### `ZINTERSTORE`
语法 `ZINTERSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]`
集合交集并保存
返回新集合的长度

## 数据库命令

### `EXISTS`
语法 `EXISTS key [key …]`
判断`key`是否存在对应的数据.
返回,存在的`key`的个数

- 判断`key`是否存在
例如 `exists users`

### `TYPE`
语法 `TYPE key`
获得`key`所对应的数据类型.
返回,`none` key不存在;`string`键值类型;`list`列表;`set`集合;`zset`有序集;`hash`哈希表;`stream`流.

- 判断`key`的类型
例如 `type users`

### `RENAME`
语法 `RENAME key newkey`
将`key`修改为`newkey`;如果`newkey`已经存在,则覆盖.

- 重命名`key`,存在则覆盖
例如 `rename users uesrs1` 将`users`重命名为`users1`

### `RENAMENX`
语法 `RENAMENX key newkey`
将`key`修改为`newkey`,仅当`newkey`不存在时,才进行.

- 重命名`key`,存在则取消
例如 `renamenx users users1` 将`users`重名为`users1`

### `MOVE`
语法 `MOVE key db`
将`key`的值移动到数据库`db`中.

- 移动数据库
例如 `move users 1` 将`users`移动到数据库1

### `DEL`
语法 `DEL key [key …]`
删除一个或多个`key`,不存在的`key`被忽略.
返回,被删除的`key`数量

- 删除
例如 `del users`

### `RANDOMKEY`
语法 `RANDOMKEY`
从数据库中随机返回获得一个`key`.

- 随机获得一个`key`
例如 `randomkey`

### `DBSIZE`
语法 `DBSIZE`
获得当前数据库中,`key`的数量.

- 获得当前数据`key`数量
例如 `dbsize`

### `KEYS`
语法 `KEYS pattern`
获得符合匹配模式`pattern`的`key`,并返回.`pattern`支持正则表达式

- 获得通配符
例如 `keys user*` 获得`user`开头的`key`.

### `SCAN`


### `SORT`

### `FLUSHDB`
语法 `FLUSHDB`
删除当前数据库中,所有的`key`

- 删除数据库
例如 `flushdb`

### `FLUSHALL`
语法 `FLUSHALL`
删除当前redis数据库中,全部的子库中的`key`

- 删除服务器中所有数据库
例如 `flushall`

### `SELECT`
语法 `SELECT index`
切换到指定数据库`index`.默认当前使用数据库索引为0

- 切换数据库
例如 `select 0` 使用索引位`0`的数据库,即默认数据库

### `SWAPDB`
语法 `SWAPDB db1 db2`
将`db1`和`db2`两个数据库内的数据进行调换.

- 调换数据库
例如 `swapdb 0 1` 将`0`和`1`两个数据库内的值进行调换

## 过期策略

### `EXPIRE`
语法 `EXPIRE key seconds`
设置,或者重置`key`的过期时间`seconds`秒.
当`key`被删除或者重新覆盖后,过期时间可以被重写;而修改`key`并不会修改时间.

- 修改过期时间
例如 `expire user 10`

### `EXPIREAT`
语法 `EXPIREAT key timestamp`
设置,或者重置`key`的过期时间,指定格式为时间戳`timestamp`.
可以通过`time`命令获得当前系统的时间戳和当前秒已经过去的微秒数.

- 设置过期时间
例如 `expireat user 1552446392`

### `TTL`
语法 `TTL key`
返回`key`的剩余生命周期,单位秒,并返回.
如果`key`不存在,则返回-2,`key`未指定过期策略,则返-1

- 获得剩余生命秒数
例如 `ttl use`

### `PERSIST`
语法 `PERSIST key`
移除`key`的过期策略.
返回,移除成功返回1;如果`key`不存在,或者未设置过期策略,返回0

- 移除过期策略
例如 `persist user`

### `PEXPIRE`
语法 `PEXPIRE key milliseconds`
设置,或者重置`key`的过期时间`milliseconds`毫秒.
当`key`被删除或者重新覆盖后,过期时间可以被重写;而修改`key`并不会修改时间.

- 设置过期时间,1秒 = 1000毫秒
例如 `pexpire user 1000`

### `PEXPIREAT`
语法 `PEXPIREAT key milliseconds-timestamp`
设置,或者重置`key`的过期时间,指定格式为时间戳`milliseconds-timestamp`,设置过期时间精度为毫秒数.
可以通过`time`命令获得当前系统的时间戳和当前秒已经过去的微秒数.

- 设置过期时间
例如 `pexpireat user 1566660000`

### `PTTL`
语法 `PTTL key`
返回`key`的剩余生命周期,单位毫秒,并返回.
如果`key`不存在,则返回-2,`key`未指定过期策略,则返-1

- 获得剩余生命时间,毫秒数
例如 `pttl user`

## 事务

### `MULTI`
语法 `MULTI`
标记一个事物的开始,以后的所有操作,都是在一个事务中使用.

### `EXEC`
语法 `EXEC`
执行事务内的所有命令.
若,事物中有`watch`监视的变量,则变量如果在事物中被修改,则该事物执行失败.

### `DISCARD`
语法 `DISCARD`
取消事务内的所有命令.

### `WATCH`
语法 `WATCH key [key …]`
监视一个或者多个`key`.如果在事务发生时间内被修改,则事务执行失败.

### `UNWATCH`
语法 `UNWATCH`
取消对所有`key`的监视.如果事物已经提交执行或者执行失败被回滚,则不需要再被`unwatch`.

## 持久化

### `SAVE`
语法 `SAVE`
将当前内存中的变量信息存入磁盘.

### `BGSAVE`
语法 `BGSAVE`
将当前内存中的变量,以异步后台的方式存入磁盘.

### `BGREWRITEAOF`
语法 `BGREWRITEAOF`
将文件写入到`AOF`文件.

### `LASTSAVE`
语法 `LASTSAVE`
返回上次修改时间的时间戳.

## 发布与订阅

### `PUBLISH`

### `SUBSCRIBE`

### `PSUBSCRIBE`

### `UNSUBSCRIBE`

### `PUNSUBSCRIBE`

### `PUBSUB`


## 模式

### 安全队列
[参考](http://redisdoc.com/list/rpoplpush.html)

### 循环列表 
[参考](http://redisdoc.com/list/brpoplpush.html)

#### 事件提醒
[参考](http://redisdoc.com/list/blpop.html)

#### 导航会话
[参考](http://redisdoc.com/expire/expire.html)



# 附录:
## 配置文件信息

| 参数                              | 默认值/参考值                                    | 说明                                                         |
| --------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| `daemonize`                       | no                                               | 是否以守护进程启动,默认非守护进程                            |
| `bind`                            | 127.0.0.1                                        | 绑定的IP地址,默认本地                                        |
| `port`                            | 6379                                             | 绑定的端口                                                   |
| `databases`                       | 16                                               | 数据库数量,默认16个,即 0~15 库                               |
| `save <second> <changes>`         | `save 900 1`  ; `save 300 10`  ; `save 60 10000` | 多少时间、有多少次更新操作，就将数据同步到数据文件           |
| `dbfilename`                      | dump.rdb                                         | 持久到本地数据文件名                                         |
| `dir`                             | `./`                                             | 本地数据库文件存放路径.                                      |
| requirepass                       |                                                  | 是否启用密码                                                 |
| `pidfile`                         | `var/run/redis.pid`                              | 程序PID进程文件                                              |
| `timeout`                         | 300                                              | 客户端闲置多久后关闭连接, `0` 为不关闭                       |
| `loglevel`                        | `verbose`                                        | 日志级别,  `debug`, `verbose`, `notice`, `warning`           |
| `logfile`                         | `stdout`                                         | 日志纪录方式,这里为标准输出;如果以守护进程启动,则日志丢入 `/dev/null` |
| `rdbcompression `                 | `yes`                                            | 是否启用压缩方式文件                                         |
| `slaveof <masterip> <masterport>` |                                                  | 当为进行集群启动时, 设置 `master` 的地址和端口               |
| `masterauth <master-password>`    |                                                  | 当为进行集群启动时, 连接 `master` 的密码                     |
| `maxclients`                      | 0                                                | 默认最大客户端连接数                                         |
| `maxmemory <bytes>`               |                                                  | 最大内存使用                                                 |
| `appendonly`                      | `no`                                             | 是否在 `key` 发生修改后,更新日志                             |
| `appendfilename`                  | `appendonly.aof`                                 | 更新日志名称                                                 |
| `appendfsync`                     | `everysec`                                       | 更新日志的条件:  1). `no` 操作系统更新日志时更新  2). `always` 每次更新key时更新  3). `everysec` 每秒更新 |
| `vm-enabled`                      | `no`                                             | 是否使用虚拟内存. 热点数据存入内存, 非热点数据存入磁盘       |
| `vm-swap-file`                    | `/tmp/redis.swap`                                | 虚拟内存的存放位置                                           |
| `vm-max-memory`                   | 0                                                | 虚拟内存时,将key存入内存,而value存入磁盘.                    |
| `vm-page-size `                   | 32                                               | swap文件的page大小设置. 32 或 64                             |
| `vm-pages`                        | 134217728                                        | swap文件的page数量                                           |
| `vm-max-thread`                   | 4                                                | swap文件的访问线程数,不可超过处理器内核数. 0 为串行访问      |
| `glueoutputbuf `                  | `yes`                                            | 客户端应答时,将较小的包合并为一个包发送                      |
| `hash-max-zipmap-entries`         | 64                                               | 超过最大数量时, 使用一种哈希算法                             |
| `hash-max-zipmap-value`           | 512                                              | 超过最大大小时, 使用一种哈希算法                             |
| `activerehashing`                 | `yes`                                            | 是否使用重置哈希                                             |
| `include`                         | `/path/to/local.conf`                            | 包含的其他配置文件信息                                       |

## 内存淘汰策略

- `noeviction` 当内存使用超过配置的时候会返回错误，不会驱逐任何键

- `allkeys-lru` 加入键的时候，如果过限，首先通过LRU算法驱逐最久没有使用的键

- `volatile-lru` 加入键的时候如果过限，首先从设置了过期时间的键集合中驱逐最久没有使用的键

- `allkeys-random` 加入键的时候如果过限，从所有key随机删除

- `volatile-random` 加入键的时候如果过限，从过期键的集合中随机驱逐

- `volatile-ttl` 从配置了过期时间的键中驱逐马上就要过期的键

- `volatile-lfu` 从所有配置了过期时间的键中驱逐使用频率最少的键

- `allkeys-lfu` 从所有键中驱逐使用频率最少的键

## 开发实践

### 命名规范

建议使用 `:` 进行命名分割, 区分不同的命名空间.
如 `user:session:A1B2C3E4F5E6`

### `STRING` 实践

- 用于统计数量
- 存放 `token`
- 存放热点图片 (序列化为二进制文件)



### `HASH` 实践

- 存放对象属性, 不同对象通过 `key` 的命名规范进行区分
- 在序列化和反序列化时,注意操作原子性

