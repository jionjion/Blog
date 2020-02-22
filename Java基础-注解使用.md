---
title: Java基础-注解使用
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




## JDK自带的注解
在JDK中有很多常见注解,举例:

| 注解              | 解释                 |
| ----------------- | -------------------- |
| @Override         | 重写父类的方法       |
| @Deprecated       | 表示该方法已经过时了 |
| @Suppvisewarnings | 忽略特定警告         |



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
标识该注解使用范围,是在类或者方法或者属性上等

| 元注解            | 解释           |
| ----------------- | -------------- |
| `TYPE`            | 类或者接口使用 |
| `FIELD`           | 字段声明       |
| `METHOD`          | 方法声明       |
| `PARAMETER`       | 参数声明       |
| `CONSTRUCTOR`     | 构造方法声明   |
| `LOCAL_VARIABLE`  | 局部变量声明   |
| `ANNOTATION_TYPE` | 注解类         |
| `PACKAGE`         | 包声明         |
| `TYPE_PARAMETER`  | 参数类         |
| `TYPE_USE`        | 自定义使用     |

### `@Retention` 运行机制注解
表示注解生效的时期

| 注解    | 说明     |
| ------- | -------- |
| SOURCE  | 源码阶段 |
| CLASS   | 编译阶段 |
| RUNTIME | 运行阶段 |

### `@Inherited` 子类继承注解
表示该类的子类可以复用该类的注解方法

### `@Documented` doc帮助注解
生成 `JAVA` 帮助文档时包含信息



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

	@Description(desc="我是一个描述",author="Jion",age=23, value = "我是方法上的注解")
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






[1]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaAnnotation/demo