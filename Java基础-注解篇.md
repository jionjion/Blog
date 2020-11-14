---
title: Java基础-注解篇
abbrlink: e7572b2
date: 2020-01-29 11:57:10
categories:
  - Java
  - Annotation
tags: [Java, Annotation]
---

# 简介

java注解的使用和自定义注解进行介绍



## 解释

注释是标注在类或者方法,属性上的一组说明,通过注解的引用,可以简化各种配置.

所有注解均为 `java.lang.annotation.Annotation` 接口的子类.


## JDK 注解
在JDK中有很多常见注解,举例:

| 注解                   | 类                              | 解释                 |
| ---------------------- | ------------------------------- | -------------------- |
| `@Override`            | `java.lang.Override`            | 重写父类或接口的方法 |
| `@Deprecated`          | `java.lang.Deprecated`          | 标识该方法已经过时了 |
| `@Suppvisewarnings`    | `java.lang.SuppressWarnings`    | 忽略特定警告         |
| `@SafeVarargs`         | `java.lang.SafeVarargs`         | 忽略泛型检查         |
| `@FunctionalInterface` | `java.lang.FunctionalInterface` | 标识为函数式接口     |



## 注解的分类

根据注解运行机制分类,可以分为三类:

| 生效时期   | 说明                                   |
| ---------- | -------------------------------------- |
| 源码注解   | 仅在源码中有                           |
| 编译时注解 | 编译为class文件时,在源码中有.JDK自带的 |
| 运行时注解 | 运行阶段依然有.如Spring的注解          |



## 元注解

用来注解其他注解的类为**元注解**,通过元注解的标注,可以实现自定义注解.
### `@Target` 作用域注解
标识该注解使用范围. 使用枚举类 `java.lang.annotation.ElementType`  标识其作用范围

| 元注解                        | 解释                     |
| ----------------------------- | ------------------------ |
| `ElementType.TYPE`            | 类, 接口, 注解类, 枚举类 |
| `ElementType.FIELD`           | 字段, 枚举中成员         |
| `ElementType.METHOD`          | 方法                     |
| `ElementType.PARAMETER`       | 参数                     |
| `ElementType.CONSTRUCTOR`     | 构造方法                 |
| `ElementType.LOCAL_VARIABLE`  | 方法中局部变量           |
| `ElementType.ANNOTATION_TYPE` | 注解类                   |
| `ElementType.PACKAGE`         | 包                       |
| `ElementType.TYPE_PARAMETER`  | 参数类型                 |
| `ElementType.TYPE_USE`        | 自定义使用               |

### `@Retention` 保留策略注解
表示注解生效的时期. 使用枚举类 `java.lang.annotation.RetentionPolicy` 表示其保留策略

| 注解                      | 说明     | 适用场景               |
| ------------------------- | -------- | ---------------------- |
| `RetentionPolicy.SOURCE`  | 源码阶段 | 编码阶段, 开发行为约定 |
| `RetentionPolicy.CLASS`   | 编译阶段 | 编译规则检查, 代码格式 |
| `RetentionPolicy.RUNTIME` | 运行阶段 | 自定义注解, 框架扩展   |

注意: 对于运行时注解, 可以通过 `java.lang.reflect.AnnotatedElement` 接口获得对象中的注解信息.

### `@Inherited` 子类继承注解

表示该类的子类可以复用该类的注解方法

### `@Native` 本地变量注解

表示变量为本地变量, 直接声明在内存中

### `@Repeatable` 可重复注解

表示注解可以被重复标注.

### `@Documented` doc帮助注解
生成 `JAVA` 帮助文档时包含信息

## 相关异常

| 异常                                                   | 级别               | 说明                                                 |
| ------------------------------------------------------ | ------------------ | ---------------------------------------------------- |
| `java.lang.annotation.AnnotationFormatError`           | `Error`            | 虚拟机在读取一个错误的注解时, 抛出异常               |
| `java.lang.annotation.AnnotationTypeMismatchException` | `RuntimeException` | 程序在读取编译后或者反序列化后的对象注解时, 抛出异常 |
| `java.lang.annotation.IncompleteAnnotationException`   | `RuntimeException` | 由于注解的属性信息不完整, 程序读取失败后抛出         |



## 自定义注解

 1. 在类名前使用 `@interface `标注该类为注解类
 2. 在类上使用**元注解**标注适用范围,运行机制,是否允许子类继承等.
 3. 在类中创建无修饰符的属性,可以通过 `default` 指定默认参数
 4. 当成员属性只有一个的时候,取名必须为 `value` .

``` java
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited //可以复用注解
@Documented
public @interface Description {		//使用@interface定义注解
							
	String desc() default "";			//当成员只有一个的时候,成员必须取名为value(),使用时可以忽略成员名和等号
							//可以没有成员,此时为标注注解
	String author() default "JION";		//成员以无参的形式进行声明
	
	int age() default 18;	//可以给定默认值
	
	String value();
}
```


## 使用自定义注解

 1. 根据自定义注解的适用范围,在类和方法上使用该注解
 2. 对于 `value()` 属性,可以直接赋值使用

``` java
@Description("这是一个类上的注解")
public class UseDescription {

	@Description(desc="我是一个描述", author="Jion", age=23, value = "我是方法上的注解")
	public void model() {
		System.out.println("使用自定义的注解");
	}
}
```



## 解析自定义注解

### 相关方法

| 方法                                                         | 说明                                 |
| ------------------------------------------------------------ | ------------------------------------ |
| `boolean java.lang.Class.isAnnotationPresent(Class<? extends Annotation> annotationClass)` | 判断class中注解是否存在              |
| `<Description> Description java.lang.Class.getAnnotation(Class<Description> annotationClass)` | 从class或 method中获得传入的注解对象 |
| `String javaAnnotation.introduce.Description.value()`        | 注解对象的value属性值                |



### 解析注解


``` java
/**解析注解,只能解析运行时注解*/
public class ParseDescription {

	public static void main(String[] args) throws Exception {
		//使用类加载器加载类
		Class<?> clazz  = Class.forName("javaAnnotation.introduce.UseDescription");
		
		//找到类上的注解,判断是否存在
		boolean isExist = clazz.isAnnotationPresent(Description.class);
		//拿到注解实例
		if (isExist) {
			//获得注解对象
			Description description = (Description) clazz.getAnnotation(Description.class);
			System.out.println("类上注解的默认Value为:"+description.value());
		}
		
		//找到方法上的注解,默认对每个方法进行遍历,不仅限于自定义的方法
		Method[] method = clazz.getMethods();
		for (Method m : method) {
			boolean isExis = m.isAnnotationPresent(Description.class);
			if (isExis) {
				Description description = (Description) m.getAnnotation(Description.class);
				System.out.println("获取方法上的注解:"+description.value());
			}
		}
	}
}
```



## 示例代码

* [*demo*][1]  基于注解的SQL生成






[1]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaAnnotation/demo.java