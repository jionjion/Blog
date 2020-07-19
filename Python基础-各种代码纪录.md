---
title: Python基础-各种代码纪录
categories:
  - Python
tags:
  - Python
abbrlink: 989a9a57
date: 2020-02-26 21:34:11
---

## 魔术方法

### 重写描述

通过重写 `__repr__	`方法为类对象的打印输出进行重写

```python
	# 重写描述方法
    def __repr__(self):
        return "<Book(id='%s', name='%s' )>" % (self.id, self.name)
```



## 常用模块

### 日期

通过日期模块 `datetime` 进行操作

```python
import datetime

# 获得当前时间
datetime.datetime.now()

# 字符串转为日期
datetime.datetime.strptime('2001-01-01', '%Y-%m-%d')
# 字符串转为日期时间
datetime.strptime('2020-01-01 18:00:00', '%Y-%m-%d %H:%M:%S')

# 当前日期时间转字符串
datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
```