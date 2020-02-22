---
title: Java基础-IO篇
abbrlink: 6498fdc3
date: 2020-01-28 17:21:58
categories:
  - Java
  - IO
tags: [Java, IO]
---

# JAVA中IO文件操作



## 简介

java中对文件读取进行了抽象,实现方式有字节流,字符流.




## 包结构

* *[BufferedInputOrOutputType][1]*                 						带缓冲的字节流读写
* *[BufferedReaderOrWriteOrPrintType][2]*    						 带缓冲的字符流过滤器读写
* *[DataInputOrOutputSteamType][3]*             						扩展数据类型的字节流读写
* *[demo][4]*                                                    						   一些功能尝试
* *[encoedType][5]*                                          						  文本编码格式
* *[FileReadOrFileWrite][6]*                                					   文件字符流读写
* *[fileType][7]*                                                						   文件类型读写
* *[InputOrOutputStreamType][8]*                   					    字符流读写
* *[InputSteamReaderOrOutputSteamWriteType][9]*             带缓冲的字节流读写操作
* *[ObjectReadeOrWriteType][10]*                    						  对象读取操作
*  *[RandomAccessFileType][11]*                      							随机文件读取



## 子包描述
### `BufferedInputOrOutputType` 包

该类对 `InputStream` 和 `OutputStream` 进行包装,通过追加套接字的形式,为其提供缓存,提高了文件读写的效率.



### `BufferedReaderOrWriteOrPrintType` 包

该类对 `FileReader` 和 `FileWriter` 类进行了包装,为其提供了缓冲,提高了文件的读写效率



### `DataInputOrOutputSteamType` 包

该类对 `InputStream` 和 `OutputStream` 类进行了包装,增强了其对数据,格式数据的支持



### `demo` 包

使用 `commons-io` 对文件的读取



### `encoedType` 包

介绍了常用的编码格式



### `FileReadOrFileWrite` 包

实现了对文件以字节流方式的直接访问



### `fileType` 包

`File` 类抽象了文件对象,其指向可以是一个目录,也可以是一个具体的文件



### `InputOrOutputStreamType` 包

`FileInputSteam` 类和 `FileOutputSteam` 继承了其抽象父类 `InputSteam` 和 `OutputSteam` , 实现了对文件以字节流方法进行访问



### `InputSteamReaderOrOutputSteamWriteType` 包

`InputStreamReader` 类和 `OutputStreamWriter` 继承了其抽象父类 `Reader` 和 `Writer` ,包装或解析了byte流和char流之间的转换.



### `ObjectReadeOrWriteType` 包

`ObjectInputStream` 和 `ObjectOutputStream` 实现了将对象转字节数组,实现了对象的序列化操作.



### `RandomAccessFileType` 包
`randomAccessFile` 实现了对文件任意位置的访问写入.可以用来执行多线程文件下载写入.





[1]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/BufferedInputOrOutputType
[2]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/BufferedReaderOrWriteOrPrintType
[3]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/DataInputOrOutputSteamType
[4]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/demo
[5]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/encoedType
[6]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/FileReadOrFileWrite
[7]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/fileType
[8]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/InputOrOutputStreamType
[9]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/InputSteamReaderOrOutputSteamWriteType
[10]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/ObjectReadeOrWriteType
[11]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/fileReadOrWrite/RandomAccessFileType