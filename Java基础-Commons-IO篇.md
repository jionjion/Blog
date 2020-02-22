---
title: Java基础-Commons-IO篇
abbrlink: 8008c0a9
date: 2020-01-28 14:21:07
categories:
  - Java
  - Commons
tags: [Java, Commons, Commons-IO]
---

# 简介

`Commons-IO` 提供了对文件,目录的IO操作,其中常用的工具类有

- `XXXComparator` 文件类,对当前文件列表进行各种排序
- `XXXFilter` 文件过滤,查找符合要求的文件对象
- 目录文件内文件变化监听 
- `FilenameUtils` 文件名工具类
- `FileSystemUtils` 当前文件系统工具类
- `FileUtils` 文件对象类
- `IOCase` 通过IO流比较对象类



## 示例代码

- [*ComparatorExample*][1] 							轻松地比较和排序文件和目录,根据文件名,大小,最后修改日比较
- [*FiltersExample*][2]                                        文件过滤器,通过不同的筛选条件,进行文件查询
- [*InputExample*][3]                                         自动将读取的字节从输入复制到输出
- [*FileMonitorExample*][4]                               监视文档的状态变化
- [*OutputExample*][5]                                      可以将输入流发送到2个不同的输出
- [*FilenameUtilsExample*][7]                           与文件名相关的操作
- [*FileSystemUtilsExample*][6]                         返回指定储存介质的可用空间
- [*FileUntilsExample*][8]                                   文件操作（移动，打开和读取文件，检查文件是否存在等）的方法
- [*IOCaseExample*][9]                                       通过 `IOCase` 枚举类, 进行IO之间的比较操作







[1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/comparatorDemo/ComparatorExample.java
[2]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/filefilterDemo/FiltersExample.java
[3]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/inputDemo/InputExample.java
[4]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/monitorDemo/FileMonitorExample.java
[5]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/outputDemo/OutputExample.java
[6]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/utilityTool/FileSystemUtilsExample.java
[7]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/utilityTool/FilenameUtilsExample.java
[8]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/utilityTool/FileUntilsExample.java
[9]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/commons/io/utilityTool/IOCaseExample.java

