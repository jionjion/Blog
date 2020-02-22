---
title: Nginx服务搭建
abbrlink: 9ba6dd5a
date: 2017-11-21 20:20:15
categories:
  - Nginx
tags: Nginx
---
-------------------------------

# 简介
常用到的中间件,对访问控制和分发请求做出响应.
[官网](http://nginx.org)

# 搭建
## 搭建顺序

### 修改为官方配置源

1.官方配置源

``` 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
```

2. `/etc/yum.repos.d` 目录下新建`nginx.repo`文件
3. 文件内容为官方配置源,将OS替换为当前的使用系统和当前的版本号

``` 
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```
4. 查看当前配置源`yum list | grep nginx`

### 安装Nginx
1. `yum install nginx`,进行安装.
2. `nginx -v` 查看当前版本
3.  `nginx -V`查看系统参数

## 启动与停止
`systemctl restart nginx.service` 重启服务
`systemctl start nginx.service ` 启动服务
`systemctl stop nginx.service` 结束服务

### Nginx安装目录

1. `rpm -ql nginx`使用rpm包管理器查询软件的安装目录



| 路径           | 类型     | 作用                              |
| -------------- | -------- | --------------------------------- |
| /etc/logrotate | 配置文件 | Nginx日志,logrotate服务的日志切割 |
| /etc/nginx ; /etc/nginx/nginx.conf ; /etc/nginx/conf.d ; /etc/nginx/conf.d/default.conf | 目录,配置文件| Nginx主配置文件|
| /etc/nginx/fastcgi_params ; /etc/nginx/uwsgi_params ; /etc/nginx/scgi_params | 配置文件 | cgi配置相关,fastcgi配置|
| /etc/nginx/koi-utf ; /etc/nginx/koi-win ; /etc/nginx/win-utf | 配置文件 | 编码转化映射化文件|
|/etc/nginx/mime.types | 配置文件 | 设置http协议的Content-Type 与扩展名对应关系 |
| /usr/lib/systemd/system/nginx-debug.service ; /usr/lib/systemd/system/nginx.service ; /etc/sysconfig/nginx ; /etc/sysconfig/nginx-debug | 配置文件 | 用于配置系统守护进程管理器的管理方式|
| /usr/lib64/nginx/modules ; /etc/nginx/modules | 目录 | Nginx模块目录|
| /usr/sbin/nginx ; /usr/sbin/nginx-debug | 命令 | Nginx服务的启动管理的终端命令 |
| /usr/share/doc/nginx-1.12.0 ; /usr/share/doc/nginx-1.12.0/copyright ; /usr/share/man/man8/nginx.8.gz | 文件,目录 | Nginx的手册和帮助文件|
| /var/cache/nginx | 目录 | Nginx的缓存目录 |
| /var/log/nginx | 目录 | Nginx的日志目录 |

| 编译选项  | 作用 |
|---------------|--------|
|--prefix=/etc/nginx ; --sbin-path=/usr/sbin/nginx ;  --modules-path=/usr/lib64/nginx/modules ;  --conf-path=/etc/nginx/nginx.conf ;  --error-log-path=/var/log/nginx/error.log  ;   --http-log-path=/var/log/nginx/access.log  ;   --pid-path=/var/run/nginx.pid  ;  --lock-path=/var/run/nginx.lock |安装的目录或者路径 |
| --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp | 执行对应模块时,Nginx所保存临时文件 |
| --user=nginx --group=nginx | 设定Nginx进程启动的用户和组用户 |
|--with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC'  | 设置额外额度参数将被添加到CFLAGS变量中 |
| --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie' | 设置附加的参数,链接系统库 |
| --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module | 模块参数|

## Nginx配置文件
### `/etc/nginx/nginx.conf` 
- http : 配置了拦截类型

### `/etc/nginx/conf.d/default.conf`
- server 配置了监听和服务,主要配置监听的地址和端口,访问路径,默认错误页面.

## 日志记录
- `/var/log/nginx/error.log` 错误日志

- `/var/log/nginx/access_log` 对访问请求记录

### `log_format` 对日志格式化输出
对请求变量和日志变量进行控制化输出 


## 模块使用

### `--with-http_stub_status_module`  模块介绍
**配置语法**
```
Syntax: stub_status
Default: -
Context: server, location
```

**配置方法**
在`default.conf` 文件中,在server模块下配置.`location`模块

``` 
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    
    location /mystatus {
        stub_status;
    }   
    
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }   
    #error_page  404              /404.html;
    
    # redirect server error pages to the static page /50x.html
    # 
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    
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

**检查语法**
`nginx -tc /etc/nginx/nginx.conf ` 检查当前配置文件语法是否正确

**重载服务**
`nginx -s reload -c /etc/nginx/nginx.conf ` 重载配置文件

 **访问路径**
 `http://192.168.1.160/mystatus`,其中`mystatus`为配置文件中自定义的路径

### `--with-http_random_index_module` 模块介绍
目录中选择一个随机主页 

**配置语法**

``` 
Syntax: random_index on | off;
Default: random_index off;
Context: location
```
在 `/etc/nginx/nginx.conf`文件`location`部分进行配置



### ` --with-http_sub_module ` 模块介绍
Http内容替换模块

**配置语法**
进行页面内容的替换
``` 
Syntax: sub_filter string replacement;
Default: -
Context: http, server, location
```
或者
服务端对客户端的请求校验内容是否发生过更新,返回给用户最新的
``` 
Syntax: sub_filter_last_modified on | off;
Default: sub_filter_last_modified off;
Context: http,server,location
```
或者
是匹配第一个字符还是匹配所有字符,默认匹配首字符
``` 
Syntax: sub_filter_once on | off;
Default: sub_filter_once on;
Context: http,server,location
```

### `-limit_conn_module`连接频率限制

**配置语法**
对key关键字进行限制,保存在limit_conn_zone空间中
``` 
Syntax: limit_conn_zone key zone=name:size;
Default: -
Context: http
```
或者
对空间内相同时间内的访问限制
``` 
Syntax: limit_conn zone number;
Default: -
Context: http, server, location
```

### `-limit-req-module`请求频率限制

**配置语法**

``` 
Syntax: limit_req_zone key zone=name:size rate=rate;
Default: -
Context:http
```

对请求速率进行限制,参数分别表示
``` 
Syntax: limit_req_zone=name [burst=number] [nodelay];
Default: -
Context:http, server, location
```

### `-http_access_module` 基于IP的访问控制
如果客户端请求经过多次拦截代理,则仅对最后一次代理的IP地址进行限制.
**配置语法**
可以根据IP或者网段进行限制,设置允许访问的
``` 
Syntax: allow address | CIDR | unix | all;
Default: -
Context: http, server, location, limit_except
```

设置不允许访问的
``` 
Syntax: deny address | CIDR | unix | all;
Default: -
Context: http, server, location, limit_except
```

### `-http_x_forwarded_for` 基于IP的访问控制
对每次中转代理IP进行记录,实现对IP地址的限制
但是客户端可以主动修改请求头信息

### `-http_auth_basic_module` 基于用户的信任登陆
**配置语法**
提示用户信息
``` 
Syntax: auth_basic string | off;
Default: auth_basic_off;
Context: http, server, location, limit_except
```

或者
在文件中保存认证的用户密码信息
文件为`htpasswd`加密的文件
``` 
Syntax: auth_basic_user_file file;
Default: -
Context: http, server, location, limit_except
```


## 静态资源Web服务分发
静态资源:非服务器动态生成的文件,如图片,文件,浏览器端渲染
当用户访问服务时,经过Nginx代理,识别其访问类型,针对于静态资源的请求,直接进行分发.

### `sendfile`模块
配置发送文件
**配置语法**
``` 
Syntax: sendfile on | off;
Default: sendfile off;
Context: http, server, location
```

### `-tcp_nopush` 模块
将多个数据报整体推送
**配置语法**
``` 
Syntax: tcp_nopush on | off;
Default: tcp_nopush off;
Context: http, server, location
```

### `-tcp_nodelay` 模块

在`keepalive`连接下,提高网络包的传输实时性
**配置语法**

``` 
Syntax: tcp_nodelay on | off;
Default: tcp_nodelay on;
Context: http,server,location
```

### `-gzip`压缩模块
将数据包进行压缩传递

**配置语法**

``` 
Syntax: gzip on | off;
Default: gzip off;
Context: http, server, location, if in location
```

### `gzip_comp_level` 压缩比例模块
配置压缩比例

**配置语法**
``` 
Syntax: gzip_comp_level level;
Default: gzip_comp_level 1;
Context: http, server, location
```

### `gzip_http_version` 压缩传输协议
对传输压缩包的协议进行指定,一般使用1.1版本的协议
**配置语法**

``` 
Syntax: gzip_http_version 1.0 | 1.1;
Default: gzip_http_version 1.1;
Context: http, server, location
```

### `http_gzip_static_module` 预读gzip功能
首先对预压缩的文件进行读取,客户端访问时,优先使用已压缩过的文件.


### `http_gunzip_module` gunzip格式压缩
对`gunzip`方式支持的浏览器提供压缩服务.

## 浏览器缓存设置
浏览器在发出请求时,会优先调用浏览器缓存,当有缓存时,检验过期机制是否通过,如果未过期就优先调用缓存;否则向服务器发出请求,获得新的资源并根据服务器响应设置缓存.
缓存命中则请求返回304,并携带缓存设置,但浏览器可以选择性执行缓存设置.

| 目标 | 方式 |
|------| -------|
| 校验是否过期 | Expires , Cache-Control(max-age) |
| 协议中Etag头信息校验 | Etag |
| Last-Modified 头信息校验 | Last-Modified |
`Cache-Control`通过验证本地缓存的超时时间,进而判断是否过期;
`Etag` 一串字符串表示文件最后修改时间.交于服务器的比对,不一致则从新请求.
`Last-Modified`将本地文件最后一次更新时间与服务器文件最后一次更新时间作比对,不一致则请求最新的

### `-expires` 模块设置报文
对响应报文头的`Cache-Control`和`Expires`头进行设置

**配置语法**
``` 
Syntax: expires [modified] time;
             expires epoch | max | off;
Default: expires off;
Context: http, server, location, if in location
```

## 跨站点访问
在客户端请求网站A的资源时,同时请求网站B的资源,一般浏览器默认关闭,避免发生跨站点攻击.

### 跨站点
对响应头`Access-Control-Allow-Origin`进行设置,允许浏览器跨站点访问.
**配置语法**
``` 
Syntax: add_header name value [always];
Default: -
Context: http, service, location, if in location
```

## 防止盗链
通过区别用户请求,进行分发

### `http-refer`防盗链配置模块
`http-refer`记录了上一次请求的地址,通过判断上一次地址是否为本站,进而拒绝那些并非本站的访问的.


**配置语法**

``` 
 Syntax: valid_referers none | blocked | server_names | string...;
 Default: -
 Context: server, location
```

## 代理服务 
对客户端或服务端的请求进行代理分发.
**正向代理:**翻墙,通过代理实现访问服务器,代理对象为客户端.
**反向代理:**针对不同客户端进行请求分发,代理对象为服务端.

### `proxy_pass URL`代理服务

**配置语法**
``` 
Syntax: proxy_pass URL;
Default: -
Context: location, if in location, limit_except
```

### `proxy_buffering`缓冲区

**配置语法**

``` 
Syntax: proxy_buffering on | off;
Default: proxy_buffering on;
Context: http, server, location
```
**相关参数**
`proxy_buffer_size`,`proxy_buffers`,`proxy_busy_buffers_size`

### `proxy_redirect_default`重定向
对跳转重定向的地址进行设置.
**配置语法**
``` 
Syntax: proxy_redirect default;
proxy_redirect off;
proxy_redirect redirect replacement;
Default: proxy_redirect default;
Context: http, server, location
```

### `proxy_set_header` 头信息设置
对头信息进行相关设置,修改,代理分发.
**配置语法**

``` 
Syntax: proxy_set_header field value;
Default: proxy_set_header Host $proxy_host;
	proxy_set_header Connection close;
Context: http, server, location
```

**扩展**
`proxy_hide_header`(隐藏头信息),`proxy_set_body`(修改体信息).

### `proxy_connect_timeout` 连接超时
**配置语法**
``` 
Syntax: proxy_connect_timeout time;
Default: proxy_connect_timeout 60s;
Context: http, server, location
```

**扩展**
`proxy_read_timeout`(代理读取超时),`proxy_send_timeout`(代理发送超时)

## 负载均衡器SLB
**GLSB:**全局负载均衡,通过访问调度节点进而获取应用服务地址,而不会直接访问中心调度节点或者应用服务中心节点.
**SLBL:**调度节点和服务节点在同一个单元中.

Nginx为一个典型的七层负载均衡的SLB.

### `upstream`负载均衡

``` 
Syntax: upstream name {...}
Defalut: -
Context: http
```


## 动态缓存





## 在线升级

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
