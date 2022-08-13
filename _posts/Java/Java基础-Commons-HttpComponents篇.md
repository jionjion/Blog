---
title: Java基础-Commons-HttpComponents篇
abbrlink: 4bdcafa9
date: 2020-01-28 14:01:28
categories:
  - Java
  - Commons
tags: [Java, Commons, Commons-HttpComponents]
---

> 使用 httpComponents 类库

<!-- more -->



# 简介

介绍如何发送Http请求,使用`apache httpComponents`完成.

需要Jar包 `fluent-hc-4.5.8.jar` , `httpclient-4.5.8.jar` , `httpcore-4.4.11.jar`



## 示例代码

* [*httpclient.SendGetRequestExample*][1]      发送`GET`请求,携带认证,携带Cookies,进行下载的示例
* [*httpclient.SendPostRequestExample*][2]     发送`POST`请求,携带附件,携带上下文Cookies,认证请求的示例





[1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/httpComponents/httpclient/SendGetRequestExample.java
[2]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/httpComponents/httpclient/SendPostRequestExample.java

