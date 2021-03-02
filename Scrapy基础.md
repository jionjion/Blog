---
title: Scrapy基础
abbrlink: 41b2623
date: 2020-01-03 21:16:43
categories:
  - Python
  - Scrapy
tags: [Python, Scrapy]
---

> Scrapy 基础操作

<!--more-->



## 官网

官网地址: 

使用ScrapydWeb管理爬虫

```bash
pip install scrapydweb

$ scrapydweb -h    # 初始化
$ scrapydweb  	# 启动
```



## 安装

使用命令

`pip install Scrapy`



## 创建项目

使用命令

`scrapy startproject 项目名 [目录名]`

以下操作在项目文件夹下操作



## 文件目录结构

- `scrapy.cfg`         部署配置文件

- `模块目录

  - `spiders`        我们的爬虫/蜘蛛 目录          
  - `items.py`      项目项定义
  - `middlewares.py` 中间件

  - `pipelines.py`     项目管道

  * `settings.py`       项目设置



## 创建爬虫

在当前目录下,执行

`scrapy genspider example example.com`  

创建一个爬取 `example.com` 的爬虫文件. 在 `spiders` 文件夹下



## shell解析

爬取一个页面,并解析 `scrapy shell 解析网页URL` 

示例

`scrapy shell -s USER_AGENT="Mozilla/5.0 (Windows NT 10.0; …) Gecko/20100101 Firefox/60.0" http://top.baidu.com/buzz?b=1&fr=topindex`



### 解析语法

获取标签

`response.xpath('//table[@class="list-table"]//td[@class="keyword"]/a[1]')`

获得标签内容.被 `<>` 包裹

`response.xpath('//table[@class="list-table"]//td[@class="keyword"]/a[1]/text()')`

获取标签本文. 属性或者标签文本

`response.xpath('//table[@class="list-table"]//td[@class="keyword"]/a[1]/text()').extract()`

获取内容,并追加正则

`response.xpath('//table[@class="list-table"]//td[@class="keyword"]/a[1]/text()').re('[.0-9]+')`

### 对象

`scrapy`   爬虫模块, `python` 类库示例

`spider` 当前爬虫对象

`item`  数据对象

`request` 请求对象

`response`  响应对象

`settings` 爬取设置.默认配置为当前项目 `settings.py` 配置



### 命令

`shelp()`  打印当前帮助信息

` view(response)` 使用当前电脑的默认浏览器打开爬取后的Html页面.

`fetch(url[, redirect=True])`  重新请求某URL,并替换当前 `response` 响应内容. `redirect` 是否允许远端服务器转发

`fetch(req)` 发送一个新的 `request` 请求对象.并获得响应.





## ORM实体类

`items.py`中通过继承编写实体,实体类以`Item`结尾```pythonclass 对象Item(scrapy.Item):    属性 = scrapy.Field()```在解析方法中,封装ORM类,并将封装后的类,通过`yield 对象`将其传递给pipeline入库



## 存入数据库

在`pipelines.py`中编写不同的处理方法,可以将处理后的结果存入数据库或者文件可以定义不同的类,在类名以`Pipeline`结尾,在方法中重写`process_item`对传递的ORM类进行操作注意,编写完成的类需要在`setting.py`中注册才能使用该方法进行后续操作  



## Xpath语法    

在浏览器控制台中,可以用过 `$x('Xpath')` 方式进行解析

| 语法                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| `article`                             | 选取所有的<article>元素的所有子节点                          |
| `/article`                            | 选取根元素<article>                                          |
| `article/a`                           | 选取所有属于<article>的直接子元素的<a>元素                   |
| `//div`                               | 选取所有<div>子元素,无论出现在文档的任何位置                 |
| `article//div`                        | 选取所有属于<article>元素的后代的div元素,不管它出现在<article>下的任何位置 |
| `//@class`                            | 选取所有名为class的属性,@表示选择属性                        |
| `/article/div[1]`                     | 选取<article>子元素的第一个<div>元素,**下标从1开始**         |
| `/article/div[last()-1]`              | 选取<article>子元素的倒数第一个<div>元素                     |
| `//div[@lang]`                        | 选择所有拥有`lang`属性的<div>元素                            |
| `//div[@lang='en']`                   | 选取所有`lang`属性值为en的<div>元素                          |
| `//div[contains(@class,"main")]`      | 选取所有class含有main属性值的<div>元素                       |
| `//div[not(contains(@class,"main"))]` | 选取所有class不含有main属性值的<div>元素                     |
| `//div[starts-with(@class,"first")]`  | 选取所有class以first开头的属性的<div>元素                    |
| `/div/*`                              | 选取属于<div>元素的所有子节点                                |
| `//*`                                 | 选取所有元素                                                 |
| `//div[@*]`                           | 选取带有属性的<div>元素                                      |
| `//div/a | //div/p`                   | 选取<div>元素下的<a>和<p>元素                                |
| `//span | //ul`                       | 选取所有的<span>和<ul>元素                                   |
| `article/div/p | //span`              | 选取所有的<article>元素的<div元素的<p>元素,以及文档中的<span>元素 |
| `//html`                              | 选取整个HTML页面                                             |
| `/a/@href`                            | 选取<a>链接的**href属性值**                                  |
| `/a/text()`                           | 选取<a>链接的**标签内文本**                                  |
| `//*[text()="Hello"]`                 | 选取所有标签内含有Hello内容的标签                            |



## 爬虫配置

### 伪装用户浏览器
携带用户浏览器信息的请求
`scrapy shell -s USER_AGENT="Mozilla/5.0 (Windows NT 10.0; …) Gecko/20100101 Firefox/60.0" 解析网页URL`

修改配置文件 `settings.py`

`USER_AGENT = 'Mozilla/5.0 (Windows NT 5.1; rv:5.0) Gecko/20100101 Firefox/5.0'`



### 忽略robots协议
在`setting.py`里把robots协议项设置为`Flase`,这样爬虫就不遵守网站规定的哪些不能爬取的协议

`OBOTSTXT_OBEY = False`

或者在爬取请求时添加请求

`scrapy shell -s USER_AGENT="Mozilla/5.0 (Windows NT 10.0; …) Gecko/20100101 Firefox/60.0" 解析网页UR`



### 日志使用

```python
import logging

# 日志
log = logging.getLogger(__name__)

log.info('日志...')
```



## 部署

### 软件安装

其中, `scrapyd-client` 仅支持Linux环境

```python
pip install scrapyd
pip install scrapyd-client
```

### 部署

通过命令生成部署程序

` scrapyd-deploy --build-egg 部署项目.egg`