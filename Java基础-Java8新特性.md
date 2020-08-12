---
title: Java基础-Java8新特性
categories:
  - Java
  - JDK
tags:
  - Java
  - JDK8
abbrlink: 8bebd4ec
date: 2020-02-22 20:43:28
---

# JAVA8

## Lambda表达式

`@FunctionalInterface` 对于只有一个声明方法的接口,可以通过该注解标识为函数式接口

### 示例代码

- [*InterfaceNoParamNoReturn*][1]											无参数,无返回值的函数式接口
- [*InterfaceNoParamWithReturn*][2]					 					无参数,有返回值的函数式接口
- [*InterfaceWithParamNoReturn*][3]										 有参数,无返回值的函数式接口
- [*InterfaceWithParamWithReturn*][4]									  有参数,有返回值的函数式接口
- [*Lambda*][5]																			 调用示例

## 函数式接口
在 `java.util.function.*` 包下,基础包含四类接口,及其扩展接口

- `Consumer<T>` 消费型接口

- `Supplier<T>` 供给型接口

- `Function<T, R>` 函数型接口

- `Predicate<T>` 断言型接口

在参数个数上扩展： 比如接收双参数的，有 Bi 前缀， 比如 `BiConsumer<T,U>`, `BiFunction<T,U,R>` ;
在类型上扩展： 比如接收原子类型参数的，有 `[Int|Double|Long]` ,`[Function|Consumer|Supplier|Predicate]`
特殊常用的变形： 比如 `BinaryOperator` ， 是同类型的双参数 `BiFunction<T,T,T>` ，二元操作符 ； `UnaryOperator` 是 `Function<T,T>` 一元操作符。

### 示例代码

- [*ConsumerTest*][6] 							消费型接口
- [*FunctionTest*][7]							   函数型接口
- [*PredicateTest*][8]							  断言型接口
- [*SupplierTest*][9]							     供给型接口

## 方法引用与构造器引用

在Lambda表达式中,直接调用已经实现的方法.
要求:参数列表及返回类型必须与声明的一致

对象::实例方法名
类::静态方法名
类::示例方法名(要求,参数列表中第一个参数是方法调用实例,第二个参数是实例方法参数)

### 示例方法

- [*ArrayRefTest*][10]												数组方法引用
- [*ConstructorRefTest*][11]									 构造器引用
- [*MethodRefTest*][12]											方法引用




## Stream流
用于操作数据源的数据处理,不会改变源数据对象,经过处理后返回处理结果.

惰性求值:在多个操作组中,如果不触发终止操作,则不进行运算.而在终止操作发生时进行全部处理.

获得途径
从集合,数组,自行创建获得流

### 常用方法

| 方法        | 简述       | 说明                                                      |
| ----------- | ---------- | --------------------------------------------------------- |
| `filter`    | 过滤       | 接受lambda表达式,并删除某些                               |
| `limit(n)`  | 限制       | 限制最大数量,前n个元素                                    |
| `skip(n)`   | 跳过       | 跳过前n个元素,若流中元素不足n个,则返回一个空流            |
| `distinct`  | 去重       | 通过 `hashCode()` 和 `equals()` 去除重复元素              |
| `map`       | 映射       | 将每一个元素通过某个方法进行转换                          |
| `flatMap`   | 映射整流   | 将每个元素处理并返回的流整合为一个流                      |
| `sorted`    | 排序       | 自然排序 `(Comparable)` ,或者传入排序规则(Comparator)排序 |
| `allMatch`  | 全部匹配   | 检查是否全部匹配                                          |
| `anyMatch`  | 至少匹配   | 检查是否至少匹配一个                                      |
| `noneMatch` | 没有匹配   | 检查是否没有匹配                                          |
| `findFirst` | 查找第一个 |                                                           |
| `findAny`   | 全部       |                                                           |
| `count`     | 总个数     |                                                           |
| `max`       | 最大值     |                                                           |
| `min`       | 最小值     |                                                           |
| `reduce`    | 规约       | 将所有求和,获得一个结果                                   |
| `collect`   | 收集       | 将符合要求的成员,`Collectors`工具类,提供多种收集器        |

### 示例代码

- [*GetSteamTest*][13]											从集合,数组获得流
- [*OperateStreamITest*][15]								  过滤,查询,匹配,极值等
- [*OperateSteamIITest*][14]								  公约,收集等


## Fork/Join

串行流与并行流
Fork/Join:将任务拆解执行后并将结果进行汇总

### 测试代码

- [*ForkJoinSum*][16]																	Fork-Join模式计算求和

- [*ForkJoinSumTest*][17]															  测试

## Optional类

容器类,代表一个值存在或者不存在.避免空指针异常

### 常用方法

| 方法                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `Optional.of(T value)`         | 创建一个实例                                                 |
| `Optional.empty()`             | 创建一个空实例                                               |
| `Optional.ofNullable(T value)` | 不为空时,创建实例;为空时,创建空实例                          |
| `get()`                        | 获得返回值,如果返回值为`null`,则返回`null`                   |
| `isPresent()`                  | 判断是否包含值                                               |
| `orElse(T other)`              | 获得返回值,如果返回值不存在,则返回默认值T                    |
| `orElseGet(Supplier other)`    | 获得返回值,如果返回值不存在,则返回默认值T                    |
| `map(Function mapper)`         | 如果有值,则进行函数式计算,并返回其计算结果的`Optional`包装;如果为空,则直接返回空包装 |
| `flatMap(Function mapper)`     | 同上                                                         |

### 示例代码

- [*OptionalTest*][18]												测试用例




## 新的时间API
原有的日期类如 `Date` 或则 `Calendar` 均是线程不安全的类.日期计算,格式化复杂.
新的日期类线程安全,并每次创建时均会生成一个新的日期类对象.

- `java.time.*`           日期
- `java.time.chrono.*`    不同地区特殊的日期格式
- `java.time.format.*`    日期格式化
- `java.time.temporal.*`  日期运算操作
- `java.time.zone.*`      时区操作

常用类

- `LocalDate`         当地日期
- `LocalTime`         当地时间
- `LocalDateTime`     当地日期+时间
- `Period`            日期之间的的间隔
- `Duration`          时间之间的间隔
- `DateTimeFormatter` 日期格式化工具
- `ZoneDate`          时区日期
- `ZoneTime`          时区时间
- `ZoneDateTime`      时区日期+时间



### 示例代码

- [*DateTimeFormatterTest*][19]  									日期格式化
- [*DurationTest*][20]														时间间隔计算
- [*InstantTest*][21]														   时间戳
- [*LocalDateTimeTest*][22]									    	  日期时间
- [*PeriodTest*][23]													 	   日期间隔计算
- [*TemporalAdjusterTest*][24]										 日期矫正,偏移计算 
- [*ZoneDateTimeTest*][25]											   带时区的日期时间.




## 可重复注解与类型注解
可重复注解:可以一个注解多次注解.
类型注解:可以在类属性或者方法参数中使用





[1]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/lambda/interfaces/InterfaceNoParamNoReturn.java
[2]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/lambda/interfaces/InterfaceNoParamWithReturn.java
[3]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/lambda/interfaces/InterfaceWithParamNoReturn.java

[4]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/lambda/interfaces/InterfaceWithParamWithReturn.java
[5]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/lambda/Lambda.java



[6]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/functions/ConsumerTest.java
[7]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/functions/FunctionTest.java
[8]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/functions/PredicateTest.java
[9]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/functions/SupplierTest.java
[10]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/methodRef/ArrayRefTest.java
[11]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/methodRef/ConstructorRefTest.java
[12]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/methodRef/MethodRefTest.java
[13]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/steam/GetSteamTest.java
[14]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/steam/OperateSteamIITest.java
[15]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/steam/OperateStreamITest.java
[16]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/forkJoin/ForkJoinSum.java
[17]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/forkJoin/ForkJoinSumTest.java
[18]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8/optional/OptionalTest.java
[19]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8.time.DateTimeFormatterTest.java
[20]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8.time.DurationTest.java
[21]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8.time.InstantTest.java
[22]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8.time.LocalDateTimeTest.java
[23]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8.time.PeriodTest
[24]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8.time.TemporalAdjusterTest.java
[25]: https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/java8.time.ZoneDateTimeTest.java
