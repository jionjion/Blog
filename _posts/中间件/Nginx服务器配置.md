---
title: Nginx服务器配置
abbrlink: '29345992'
date: 2018-12-03 21:38:25
categories:
  - Nginx
tags: [Nginx]
---

> Nginx 的使用与记录

<!--more-->



# 基础篇

## Nginx中间件
中间件:处理Web请求与应用,硬件之间的桥梁
Nginx是一个开源的高性能,可靠的HTTP中间件,代理服务

### 选择Nginx的原因
#### 实现IO多路复用epoll
1. 让一个Socket实现请求多个IO请求
多个描述符的I/O操作都能够在一个线程内并发交替地顺序完成,这就叫做I/O多路复用,这里的"复用"是指复用同一个线程.

2. 什么是epoll
IO多路复用的实现方式,另有select ,

#### 轻量级
1.功能模块
2.代码模块

#### CPU亲和
 将CPU核心和Nginx工作进程相绑定,把每个worker进程固定在一个CPU中执行,减少切换CPU的cache miss,获得更好的性能

 #### sendfile
 减少文件之间的传递方式,不通过用户空间而是将静态资源直接通过内核空间进行拷贝

## 快速安装
### 软件安装

编译环境
` yum -y install gcc gcc-c autoconf pcre pcre-devel make automake`
其他工具
`yum -y install wget httpd-tools vim`
目录环境
`cd /opt; mkdir app download logs work backup`

| 目录 | 作用 |
| ---- | ---- |
|  app    |     软件目录 |
|  download   | 下载目录      |
| logs | 日志信息 |
| work | 工作目录 |
| backup | 备份文件 |

访问网站`http://nginx.org/en/download.html`获得想要安装版本
拷贝CentOS的yum源配置信息

``` bash
[nginx] 
name = nginx repo 
baseurl = 
http://nginx.org/packages/mainline/OS/OSRELEASE/$basearch/gpgcheck = 0 
enabled = 1
```
新增yum源文件,在目录`/etc/yum.repos.d/`下,新增文件`nginx.repo`,拷贝以下信息
``` bash
[nginx] 
name = nginx repo 
baseurl = http://nginx.org/packages/mainline/centos/7/$basearch/
gpgcheck = 0 
enabled = 1
```
检查 `yum list | grep nginx`,查看是否有Nginx的对应版本
安装 `yum install nginx`,完成安装
使用`nginx -v`查看当前安装版本,`nginx -V`查看当前nginx的安装模块

### 版本信息

`nginx -v` 查看当前版本
`nginx -V`查看系统参数


## 启动与停止
`systemctl restart nginx.service` 重启服务
`systemctl start nginx.service ` 启动服务
`systemctl stop nginx.service` 结束服务

## Nginx目录信息
| 目录 | 类型 | 作用 |
| --- | --- | --- |
| /etc/logrotate.d/nginx | 配置文件 | Ngixn的日志配置,滚动策略 |
| /etc/nginx <br> /etc/nginx/nginx.conf <br> /etc/nginx/conf.d <br>  /etc/nginx/conf.d/default.conf | 配置文件 | Nginx主要配置文件 |
| /etc/nginx/fastcgi_params <br> /etc/nginx/uwsgi_params  <br>  /etc/nginx/scgi_params | 配置文件 | cgi配置,fastcgi配置 |
| /etc/nginx/koi-utf <br> /etc/nginx/win-utf | 配置文件 | 编码转化映射文件 |
| /etc/nginx/mime.types | 配置文件 | http协议 Content-Type 与扩展名对应关系 |
| /etc/sysconfig/nginx <br> /etc/sysconfig/nginx-debug <br> /usr/lib/systemd/system/nginx-debug.service <br> /usr/lib/systemd/system/nginx.service | 配置文件 | 配置系统守护进程与守护进程管理方式 |
| /etc/nginx/modules <br> /usr/lib64/nginx/modules | 目录 | 模块目录 |
| /usr/sbin/nginx <br>  /usr/sbin/nginx-debug  <br> /usr/lib64/nginx | 命令 | 服务的启用管理的终端命令 |
| /usr/share/doc/nginx-1.15.6 <br> /usr/share/doc/nginx-1.15.6/COPYRIGHT  <br> /usr/share/man/man8/nginx.8.gz | 目录,文件 | 帮助手册 |
| /var/cache/nginx | 目录 | 缓存目录 |
| /var/log/nginx | 目录 | 日志目录 |
|  /usr/libexec/initscripts/legacy-actions/nginx <br> /usr/libexec/initscripts/legacy-actions/nginx/check-reload <br>  /usr/libexec/initscripts/legacy-actions/nginx/upgrade | 文件,目录 | 版本兼容性 |
|  /usr/share/nginx <br>  /usr/share/nginx/html <br> /usr/share/nginx/html/50x.html <br> /usr/share/nginx/html/index.html | 文件,目录 | 默认页面,错误页面 |

## 安装编译参数
|  编译选项 | 作用 |
| --- | --- |
| --prefix=/etc/nginx <br> --sbin-path=/usr/sbin/nginx <br> --modules-path=/usr/lib64/nginx/modules <br> --conf-path=/etc/nginx/nginx.conf  <br> --error-log-path=/var/log/nginx/error.log  <br> --http-log-path=/var/log/nginx/access.log  <br> --pid-path=/var/run/nginx.pid  <br> --lock-path=/var/run/nginx.lock | 安装的路径,pid文件,lock锁文件 |
|  --http-client-body-temp-path=/var/cache/nginx/client_temp <br>  --http-proxy-temp-path=/var/cache/nginx/proxy_temp <br>  --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp  <br> --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp  <br> --http-scgi-temp-path=/var/cache/nginx/scgi_temp | 执行对应模块时,所生成的临时文件 |
| --user=nginx  <br> --group=nginx  | 设定进程的启动用户和用户组 |
| --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC'  | C语言编译时.设置额外的参数到CFLAGS变量 |
| --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' | C语言编译时,需要的链接系统库 |
| --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module | 安装的扩展模块 |

## 配置语法
nginx配置文件`/etc/nginx/nginx.conf`
```bash
# 启动用户
user  nginx;
worker_processes  1;

# 错误日志
error_log  /var/log/nginx/error.log warn;
# pid文件位置
pid        /var/run/nginx.pid;

#
events {
    worker_connections  1024;
}

# 服务器配置
http {
	# 请求Content-Type
    include       /etc/nginx/mime.types;
	# 
    default_type  application/octet-stream;
	
	# 日志类型
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

	# 访问日期
    access_log  /var/log/nginx/access.log  main;

	# sendfile访问,避免从用户空间访问,而从内核空间访问
    sendfile        on;
	
	# 
    #tcp_nopush     on;

	# 持续连接超时,单位秒
    keepalive_timeout  65;

	# gzip压缩
    #gzip  on;

	# 其他配置文件
    include /etc/nginx/conf.d/*.conf;
}
```

默认配置文件,`/etc/nginx/conf.d/default.conf`.

```bash
# 监听服务,可以有多个
server {
	# 监听端口
    listen       80;
	# 服务名,域名,
    server_name  localhost;

	# 字符集合
    #charset koi8-r;
	# 当前监听服务的日志
    #access_log  /var/log/nginx/host.access.log  main;

	# 监听路径,默认路径
    location / {
		# 页面的根路径
        root   /usr/share/nginx/html;
		# 默认首页
        index  index.html index.htm;
    }

	# 错误页面 		错误代码  	返回页面
    #error_page  404              /404.html;
	# 错误页面		错误代码	返回路径页面
    error_page   500 502 503 504  /50x.html;
	# 错误的返回页面
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```


## http请求
http请求包括:
request
请求行,请求头部,请求数据
response
状态行,消息包头,响应正文


## Nginx的log
error.log 记录错误信息
access_log 记录访问状态

### 配置语法 log_format
将请求信息中的参数放入到access_log日志中

语法 `log_format name[escape=default | json] string ... ;`
默认 `log_format combined "..."; `
限制 `http`


## 默认变量
### http请求变量
- `arg_parameter` :  获得请求http请求参数parameter
- `http_HEADER`: 将请求的header的信息进行输出,如`http_user_agent`,横杠转为下划线,并小写
- `sent_http_HEADER`: 将响应的header的信息进行输出.

### nginx内置变量
参阅官网文档

### 自定义变量

## 默认模块
分为Nginx官方提供的和第三方提供

### with-http_stub_status_module
用于监控Nginx的客户端状态,当用户访问配置的监听端口时,返回当前信息
`如:http://192.168.1.16/mystatus`

返回信息
Active connections(当前活跃连接): 2 
server accepts(请求)  handled(连接) requests(总处理)
 6 6 7 
Reading(读): 0 Writing(写): 1 Waiting(等待): 1 

#### 配置语法

语法 `stub_status;`
默认 `-`
限制 `server, location` 

### with-http_random_index_module
在当前目录中随机选择一个作为随机主页,不会选择以.开头的文件

#### 配置语法

语法 `random_index on | off;`
默认 ` off;`
限制 `location`

### with-http_sub_module
对响应http内容的替换,将服务器的返回给客户端的内容进行修改.

#### 配置语法
将string的指定的内容替换为replacement的内容

语法 `sub_filter string replacement;`
默认 `-`
限制 `http, server, location`

或者
匹配最后一次修改的 
语法 `sub_filter_last_modified on | off;`
默认 `-`
限制 `http, server, location`

或者
是否匹配第一次出现的,`on`表示只匹配第一次的,`off`表示都进行匹配
语法 `sub_filter_once on | off;`
默认 `sub_filter_once on`
限制 `http, server, location`

###  limit_conn_module
连接频率限制,根据http版本的可以一次连接中发起多次请求
http 1.0 一次连接一次请求
http 1.1 一次连接多次顺序请求
http 2.0 一次连接多路并行请求

#### 配置语法
连接池限制,`key`作为限制的区分依据,如请求IP(conn),`name`命名连接池,`size`连接池大小,`k`千字节或`m`兆表示
语法 `limit_conn_zone key zone=name:size;`
默认 `-`
限制 `http`

配置连接池信息,`zone`连接池的名字,`number`限制并发请求限制
语法 `limit_conn zone number;`
默认 `-`
限制 `http, server, location`


###  limit_req_module
请求频率限制

#### 配置语法

请求池限制,`name`命名请求池,`size`请求池大小,`tate`请求速率限制,个/秒
语法 `limit_req_zone key zone=name:size rate=tate;`
默认 `-`
限制 `http`

配置请求池,`zone`请求池的名字,`burst`当并发访问超过限制时,等待的请求数 `nodelay`超过请求后,是否等待连接,否则返回错误
语法 `limit_req zone=name [burst=number] [nodelay];`
默认 `-`
限制 `http, server, location`


## 访问限制

- 基于IP的访问控制
- 用户认证登录访问控制 


### http_access_module

#### 配置语法
配置那些可以访问
`address`IP地址,`CIDR`网段,	`unix`linux之间Soket访问,`all`允许所有访问

语法 `allow address | CIDR | unix: | all;`
默认 `-`
限制 `http, server, location, limit_except`

配置那些不能访问
语法 `deny address | CIDR | unix | all;`
默认 `-`
限制 `http, server, location, limit_except`

限制
仅能限制最后一个请求的IP,网段,对于多次代理转发的请求,并不能很好的限制请求
方法一:采用别的Http请求进行控制,如`HTTP_X_FORWARD_FOR`
方法二:结合geo模块使用
方法三:通过HTTP自定义变量传递

### http_auth_basic_module
对请求进行用户密码认证

#### 配置语法
`string`表示开启,并作为前端提示返回用户

语法 `auth_basic string | off;`
默认 `auth_basic off;`
限制 `http, server, location, limit_except`

配置认证用户密码文件
`file`为认证的用户名密码存放文件,便于认证提示
具体扩展验证方式,加解密方式,参考官网

语法 `auth_basic_user_file file;`
默认 `-`
限制 `http, server, location, limit_except`

示例
安装 htpasswd
` yum install httpd-tools`
生成密码,为`admin`用户生成密码,接下来输入两次密码完成预设
` htpasswd -c ./auth_conf admin`

扩展:可以使用Nginx结合LADP利用nginx-auth-ldap模块

## 更多补充

- Http的请求和连接
- 请求限制与连接限制
-  access模块配置语法
-  请求限制局限性
- 基本安全认证
- auth模块配置语法
- 安全认证局限性



# 实践


## 静态资源WEB服务
客户通过浏览器访问Nginx,Nginx判断如果是图片,视频,文本,直接返回对应静态资源文件.

### 静态资源服务场景
比如,CDN分发网络,将文件存在不同的地区Nginx服务器中;用户在不同地区访问资源时,交由Nginx返回最近节点的静态资源

### 静态资源服务配置

#### 配置语法 sendfile
使用`sendfile`方式,通过内核资源进行文件传输,加快静态资源传输速度

语法 `sendfile on | off;`
默认 `sendfile off;`
限制 `http, server, location, if in location`

#### 配置语法 tcp_nopush
在开启`sendfile`下,将多个数据包整合统一发送,提高网络包的传输效率,推荐大本文下开启

语法 `tcp_nopush on | off;`
默认 `tcp_nopush off;`
限制 `http, server, location`

#### 配置语法 tcp_nodelay
在开启`keepalive`下,生成数据包时就进行发送,提高网络数据包的传输实时性;对于数据及时性要求高的传输,要求开启

语法 `tcp_nodelay on | off`;
默认 `tcp_nodelay on`;
限制 `http, server, location`

### 静态资源压缩

#### 配置语法 gzip 
对文件进行压缩,提高传输效率.

语法 `gzip on | off;`
默认 `gzip off;`
限制 `http, service, location, if in location`

####  配置语法 gzip_comp_level
文件压缩等级

语法 `gzip_comp_level leve;`
默认 `gzip_comp_level 1;`
限制 `http, server, location`

#### 配置语法 gzip_http_version
语法 `gzip_http_version 1.0 | 1.1;`
默认 `gzip_http_version 1.1;`
限制 `http, server, location`

#### 扩展模块 http_gzip_static_module
预读gzip功能
在进行压缩前先试图从磁盘中加载压缩文件.如果存在已经压缩好的文件,则直接返回用户,否则进行资源的正常加载返回.
多用于下载压缩包文件时,对于一般文件的在线预览意义不大.

#### 扩展模块 http_gunzip_module 
支持gunzip压缩方式
当一些浏览器不支持gzip压缩方式时,使用gunzip压缩进行返回


### 客户端缓存
浏览器在初次请求网站时,没有缓存需要将请求的静态资源存放在本地缓存中;第二次访问网站时,会首先校验文件过期与否,如果没有过期,则直接使用缓存文件,否则重新请求.

| 目标 | 方式 |
|------| -------|
| 校验是否过期 | Expires , Cache-Control(max-age) |
| 协议中Etag头信息校验 | Etag |
| Last-Modified 头信息校验 | Last-Modified |

通过请求头中的`Expires`,`Cache-Control(max-age)`的信息,进行校验当前缓存文件是否过期;如果超期了,则先后进行Etage和Last-Modified验证
Etag头信息通过一串字符与服务器的文件的描述字符相比较判断是否过期,
Last-Modified头信息校验文件时间戳与服务器的相比是否过期.
如果是缓存文件,则使用304返回;否则则为200返回


#### 配置语法 expires
为返回文件添加 Cache-Control, Expires 头信息

语法 `expires [modified] time;`或者`expires epoch | max | off;`
默认 `expires off;`
限制 `http, server, location, if in location`

### 防盗链
防止网络资源被恶意占用

#### 配置语法 http_refer 
为请求的`http_refer`信息进行验证,限制
http_refer信息记录的浏览器的上一次浏览页面信息
通过这个属性,来判断一些静态资源,如图片等,是否为自己页面的请求加载,进而判断拦截

`none`允许不带http_refer信息的请求;
`blocked`允许不标准的请求信息
`string` 填写允许请求访问的地址,支持正则表达式
语法 `valid_referes none | blocked | server_names | string ...;`
默认 `-`
限制 `server, location`

### 跨域访问
跨站访问.当在一个页面进行多个主机请求(不同域名).
容易出现CSRF攻击,造成浏览器不安全.

#### 配置语法 add_header
添加`Access-Control-Allow-Origin`头信息,指明允许跨站访问的请求网站
添加`Access-Control-Allow-Methods`头信息,指明允许访问的方式.如GET,POST,PUT,DELETE,OPTIONS

语法 `add_header name value [always];`
默认 
限制 `http, server, location, if in location`

## 代理服务
客户端通过各种协议如HTTP,HTTPS,ICMP\POP\IMAP,RTMP访问nginx,nginx作为代理服务器,将用户请求分发到不同的服务器处理

### 正向和方向代理的区别
正向代理是为客户端,如各种翻墙,客户端限制访问服务器
方向代理是为服务端,如分发客户请求到服务器,服务器80端口反向代理到8080端口

### 代理配置
#### 配置语法 proxy_pass
将Nginx收到的请求转到代理的URL

语法 `proxy_pass URL;`
默认 `-`
限制 `location, if in location, limit_except`

#### 配置语法 proxy_buffering
配置代理的缓存开启

语法 `proxy_buffering on | off;`
默认 `proxy_buffering on;`
限制 `http,server, location`

#### 配置语法 proxy_buffer_size
配置代理的缓存大小,单位k千字节,小于页面即可

语法 `proxy_buffer_size  size;`

#### 配置语法 proxy_buffers
配置缓存数量,大小,单位k千字节

语法 `proxy_buffers num size;`

#### 配置语法 proxy_busy_buffers_size
busy缓存大小,单位k千字节

语法 `proxy_busy_buffers_size size;`

#### 配置语法 proxy_max_temp_file_size
当内存空间耗尽,放到临时文件的大小,单位k千字节
文件放置位置在nginx安装时指定的缓存目录

语法 `proxy_max_temp_file_size 256K;`

#### 配置语法 proxy_redirect
配置代理跳转重定向,默认返回301重定向地址

语法 `proxy_redirect default | off | redirect replacement;`
默认 `proxy_redirect default;`
限制 `http, server, location`

#### 配置语法 proxy_set_header
为代理跳转的请求添加http请求头信息

语法 `proxy_set_header field value;`
默认 `proxy_set_header Host $proxy_host; proxy_set_header Connext close;`
限制 `http, server, location`

#### 配置语法 proxy_hide_header

#### 配置语法 proxy_set_body

#### 配置语法 proxy_connect_timeout
配置代理转发连接超时时间,限制tcp连接超时时间,单位s秒

语法 `proxy_connect_timeout time;`
默认 `proxy_connect_timeout 60s;`
限制 `http, server, location`

#### 配置语法 proxy_read_timeout
已经建立连接情况下,等待处理的时间,超时自动断开,单位s秒

#### 配置语法 proxy_send_timeout
请求完成后,发送给客户端的发送超时时间,单位s秒

#### 配置语法 proxy_next_upstream
当负责代理的服务器挂掉后,自动跳到下一台服务器,在多个服务对外提供时,可以实现热切换

## 负载均衡
当用户请求Nginx时,通过某些业务规则合理地分发请求方式,均衡服务压力.
四层负载均衡:根据传输层,进行不同服务的分发.
七层负载均衡:根据应用层,进行基于内容的分发.Ngixn属于七层负载均衡

### Nginx负载均衡原理
通过将用户的请求进行代理转发给其他服务,如一组`upstream server`,结合不同的轮训规则,降低负载

#### 配置语法 upstream
定义一组服务池,将多个服务进行打包,通过不同的轮询规则,进行代理分发,
`upstream`关键字,定义服务池,`name`命名服务池的名字,
可以根据IP,域名,网段,unix的Socket等方式定义服务节点.
更多参数如下:
`weight` 定义权重,后跟数字,越大几率越高;用于加权轮询
`down` 该节点暂不提供服务
`backup` 备份节点,当主节点荡掉后启用
`max_fails` 允许请求失败的次数,超过后停止使用
`fail_timeout` 经过max_fails失败后,服务暂停的时间,然后再去尝试服务是否恢复
`max_conns` 最大连接数量

语法 `upstream name {  service address | domain | CIDR | unix  [weight=size] [backup ...]}`
默认 `-`
限制 `http` 

轮训方式
加权轮训(默认) :   weight值越大,分配到的几率更高
ip_hash: 每个请求按照IP的hash结果分配,使同一个IP访问固定的后端服务器,
least_conn: 最小链接数,哪个机器当前链接少,就分发
url_hash:根据访问的url进行分类,相同的url定向到同一个服务器
hash关键数值:使用hash去解算自定义的key,相同的key访问同一个服务器,如定义cookie

配置自定义hash,轮训规则
语法 `hash key [consistent]`
默认 `-`
限制 `upstream`

## 缓存服务
减少对后端的请求
缓存如果放在服务端,则成为服务端缓存,如reduce
缓存如果放在代理或者中间件,则被称为代理缓存
缓存如果放在浏览器,则成为客户端缓存

### 配置语法 proxy_cache_path
配置缓存文件的存放信息
`path`定义路径
`keys_zone`关键字,定义`name`命名代理路径,并配置`size`大小
`levels`目录分集,一般写作`levels=1:2`,以两层目录分集
`max_size` 最大缓存大小,溢出后删除低命中缓存
`inactive` 缓存文件未命中,最大生命周期
`use_temp_path` 临时缓存,一般关闭

语法 `proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loadere_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];`
默认 `-`
限制 `http`


### 配置语法 proxy_cache
配置具体使用的代理路径

语法 `proxy_cache zone | off;`
默认 `proxy_cache off;`
限制 `http, server, location`

### 配置语法 proxy_cache_valid
配置缓存过期周期

语法 `proxy_cache_valid [code ...] time;`
默认 `-`
限制 `http, server, location`

### 配置语法 proxy_cache_key 
配置缓存的维度

语法 `proxy_cache_key string;`
默认 `proxy_cache_key $scheme$proxy_host$request_uri;`
限制 `http, server, location`

### 配置语法 proxy_no_cache
配置哪些页面不能够进行缓存

语法 `proxy_no_cache string....;`
默认 `-`
限制 `http, server, location`


### 配置语法 slic
如果请求文件过大,则将文件分片返回.
优势:每个请求作为一个独立的文件,单个请求失败其他不受影响
劣势:当文件很大,分片很小,分片过多时,可能会导致文件描述符耗尽

`size` 指定分片文件的大小
语法 `slice size`
默认 `slice 0`
限制 `http, server, location`



# 深度学习

## 动态分离
当页面既有静态资源和动态页面请求渲染时,进行资源相分离
通过将静态资源放置在同一个连接路径下,匹配某个请求,而动态资源分配另外的请求的规则匹配.当动态资源出现问题时,不会被影响静态资源刷新.


## rewrite规则
通过重写请求的URL可以实现URL的跳转,兼容页面版本请求,将旧版本的请求重新分发;缩短请求URL的地址栏显示;优化SEO请求.

### Nginx正则
`^` 匹配字符串的开始
`$` 匹配字符串的结束
`{n}` 重复n次
`n,` 重复n次或者更多次
`[c]` 匹配单个字符c
`[a-z]` 匹配a-z小写字母的任意一个
`\` 转移字符
`( )` 用于匹配括号之间的内容,通过$1,$2进行调用

### 配置语法 rewrite
重写请求URL
`regex` 匹配需要改写的URL的正则
`replacement` 替换为的URL
`flag` 各种标签,有如下
- `last`停止rewrite检测,并跳转到指定的请求,相当于两次请求
- `break` 停止rewrite检测,尝试访问当前url,当前url不存在页面则返回404
- `redirect` 返回302临时重定向,地址栏显示跳转后的地址,,以后客户端依然会请求该url
- `permanent` 返回301永久重定向,地址栏会显示跳转后的地址.以后客户端不会再请求该url

语法 `rewrite regex replacement [flag];` 
默认 `-`
限制 `serever, location, if`


#### 优先级
1.执行server模块下的rewrite指令
2.执行location匹配的rewrite指令
3.执行选定的location中的rewrite指令

## 高级模块

### 配置语法 secure_link
1.制定允许及检查请求连接的真实性,以及保护资源免遭未经授权的访问
2.限制连接生效周期,超过一定时间之后请求连接失效

验证过程
1.客户端请求下载
2.服务器生成下载地址,并携带校验参数`md5`(url,cookie,key)摘要和`expires`过期时间
3.客户端携带校验参数,在规定时间请求服务器
4.服务器校验,并返回下载资源


`expression` 配置表达式
语法 `secure_link expression;`
默认 `-`
限制 `http, server, location`

`expression` 配置表达式
语法 `ssecure_link_md5 expression;`
默认 `-` 
限制 `http, server, location`

### 配置语法 geoip
基于IP的地址匹配MaxMind GeoIP二进制文件,读取IP所在的地域信息,随后做出规则
安装`yum install nginx-module-geoip`

## Https模块
https加密原理
1.客户端发起ssl连接
2.服务端将公钥发送客户端
3.客户端使用公钥加密对称密码给服务端
4.服务端使用私钥解密客户端提供的对称密码
5.建立基于对称密码的传输数据请求响应

步骤
1.生成key秘钥
2.生成证书签名请求文件csr文件
3.生成证书签名文件ca文件

优化
1.开启keepalive长连接
`ssl_session_timeout 10m;`
2.设置ssl session缓存
`ssl_session_cache shared:SSL:10m;`



# 在线升级

首先查看当前的Nginx配置,确定各模块依赖版本

```bash
[root@localhost software]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.15.6
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-3) (GCC)
built with OpenSSL 1.1.0j  20 Nov 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --add-module=/software/nginx_upstream_check_module-master --with-openssl=/software/openssl-1.1.0j --with-http_ssl_module
```

备份原有,(提示覆盖原有的备份时,输入y)

```bash
mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
```

当前工作空间,以下操作在当前文件夹下执行

```bash
/software
```

在当前工作空间下,下载最新的版本的Nginx,并解压

```bash
wget https://nginx.org/download/nginx-1.16.1.tar.gz
tar -zxvf nginx-1.16.1.tar.gz
```

进入解压后文件目录

```bash
cd /software/nginx-1.16.1
```

注入上面看到的配置信息,并编译.(编译用时三分钟)

```bash
[root@localhost nginx-1.16.1]# ./configure --prefix=/usr/local/nginx --add-module=/software/nginx_upstream_check_module-master --with-openssl=/software/openssl-1.1.0j --with-http_ssl_module

[root@localhost nginx-1.16.1]# make
```

检查是否编译成功. 在当前目录下出现 objs 目录,并在objs目录下存在 nginx 命令时.表示编译成功

```bash
[root@localhost nginx-1.16.1]# ls
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  objs  README  src
[root@localhost nginx-1.16.1]# cd objs/
[root@localhost objs]# ls
addon  autoconf.err  Makefile  nginx  nginx.8  ngx_auto_config.h  ngx_auto_headers.h  ngx_modules.c  ngx_modules.o  src
```

查看编译后版本信息

```bash
[root@localhost objs]# ./nginx -V
nginx version: nginx/1.16.1
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-3) (GCC)
built with OpenSSL 1.1.0j  20 Nov 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --add-module=/software/nginx_upstream_check_module-master --with-openssl=/software/openssl-1.1.0j --with-http_ssl_module
```

没有问题后,将当前程序拷贝

```bash
[root@localhost objs]# cp nginx /usr/local/nginx/sbin/
```

更新现有程序 

```bash
[root@localhost nginx-1.16.1]# make upgrade
/usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
sleep 1
test -f /usr/local/nginx/logs/nginx.pid.oldbin
kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin`
```

检查是否更新成功

```bash
[root@localhost nginx-1.16.1]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.16.1
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-3) (GCC)
built with OpenSSL 1.1.0j  20 Nov 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --add-module=/software/nginx_upstream_check_module-master --with-openssl=/software/openssl-1.1.0j --with-http_ssl_module
```

  重启服务

```bash
service nginx restart
```

# 架构
## 常见问题

### Spring 重定向时从 https 转为 http

修改 `location` 下, 增加 `proxy_redirect http:// https://;` 默认从 `http` 转为 `https`

### 相同Server_name的多个虚拟主机的优先级

`server_name`,如相同域名,在多个匹配文件中重复引用,则优先级根据配置文件的排列顺序进行读取

### location的匹配优先级
在匹配符中,根据使用的不同,具有不同的匹配顺序

`=` 进行普通字符精确匹配,优先级最高
`^~`  普通字符匹配,前缀匹配
`~ \~*`  执行一个正则匹配,优先级最低

### try_files使用
`try_files` 查看文件是否存在

### Nginx的alias和root区别
`root`结合请求的目录,将url中的请求路径拼拼接在root中定义的根目录进行访问
`alias`替换请求的目录,直接在alias定义的目录中rul中的文件

### 上传文件限制
设置 `client_max_body_size`进行限制

### 网关错误
返回`502 bad gateway`,表示后端服务无响应 
返回`504 gateway time-out`,表示服务器运行超时

## 性能优化
可以从以下切入
- 网络
- 系统 
- 服务
- 程序
- 数据库 

### 文件句柄
文件句柄
在每次打开一个文件中,会分配一个文件句柄,程序默认的最大文件句柄为1024个
设置方式
系统全局性修改,用户局部性修改,进程局部性修改
在`/etc/security/limits.conf` 修改全局和用户的修改
在`/etc/nginx/nginx.conf`配置`worker_rlimit_nofile`属性限制最大句柄

### CPU亲和
对多核CPU进行限制
在`/etc/nginx/nginx.conf`配置`worker_processes`限制使用核数,并用`worker_cpu_affinity`绑定,`worker_cpu_affinity`限制CPU使用

最佳使用
`worker_cpu_affinity auto`表示自动配置CPU


## 恶意行为
### 爬虫
- 使用`referer`限制
- 使用`secure-link_module`验证数据时效性和有效性
- 使用`access_module`对后台登录,部分用户服务进行数据IP控制


## 防火墙
ngx_lua_waf

## 禁止非IP访问

在配置文件中增加以下配置. 监听 `80` 和 `443` 端口,使其在访问时,如果为IP自动拒绝.返回 `500` 错误
另外,SSH证书可以随意指定,仅做配置使用

```bash
server {
    listen       80 default;
    listen       443 default_server;
    server_name  _;
    return       500;
    # 证书配置
    ssl_certificate                    /etc/nginx/cert/jionjion.top.pem;
    ssl_certificate_key                /etc/nginx/cert/jionjion.top.key;
    ssl_session_timeout                5m;
    ssl_protocols                      TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers                        ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers          on;
}
```

