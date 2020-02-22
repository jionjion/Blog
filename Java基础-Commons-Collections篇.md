---
title: Java基础-Commons-Collections篇
abbrlink: 81a9d040
date: 2020-01-28 11:50:51
categories:
  - Java
  - Commons
tags: [Java, Commons, Commons-Collections]
---



##简介

简化Java中集合框架的操作. 参考[官网](http://commons.apache.org/proper/commons-collections/)

其中

- `ListUtils` 提供了对 `List` 的扩展支持

- `SetUtils` 提供了对 `Set` 集合的扩展工具类
- `SetView` 内部类为 `SetUtils` 下的不可变的 `Set` 集合
- `Bag` 提供了对 `Set` 集合的统计扩展
- `MapUtils` 提供了对 `Map` 字典的扩展工具
- `BidiMap` 扩展了 `Map` 字典,可以通过 `value` 找到 `key`
- `IterableMap`  继承 `Map` 字典,可以进行迭代操作
- `OrderedMap` 扩展了 `Map` 字典可以



## 示例代码

- [*BagsExample*][1] 				额外提供了对 `set` 的统计,计算类
- [*BidiMapExample*][2]           可以通过 `key` 寻找 `value` ,也可以通过 `value` 寻找 `key` 要求 `Key-Value` 一一对应,否则如果 `key` 或者 `value` 重复,只会保存最后一个结果
- [*IterableMapExample*][3]     `IterableMap` 类的例子
- [*ListUtilsExample*][4]            测试 `ListUtils` 类的常用方法
- [*MapUtilsExample*][5]          `MapUtils` 类的例子
- [*OrderedMapExample*][6]   `OrderedMap` 类的例子
- [*SetUtilsExample*][7]            `SetUtils` 类常用的方法
- [*SetViewExample*][8]            `SetView` 类,只读的 `set` 集合



[1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/BagsExample.java
[2]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/BidiMapExample.java
[3]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/IterableMapExample.java
[4]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/ListUtilsExample.java
[5]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/MapUtilsExample.java
[6]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/OrderedMapExample.java
[7]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/SetUtilsExample.java
[8]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/collections/SetViewExample.java



