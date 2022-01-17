---
title: Java基础-反射
abbrlink: '4842548'
date: 2020-01-30 10:27:14
categories:
  - Java
  - Reflex
tags: [Java, Reflex]
---

> Java 的反射操作.

<!--more-->



# 简介

通过反射,可以获得类对象的内部信息,进而可以修改或者调用类内部的相关方法.



## 类类型的获得
- 通过该类的`class`静态属性获得该类的类类型
- 通过使用该对象的`getClass()`方法,获得该对象所在类的类类型
- 通过类加载器,`Class.forName()`获得工作空间下该类的类类型

**示例**

``` java
public class GetClass {

	@SuppressWarnings({ "rawtypes", "unused" })
	public static void main(String[] args) throws Exception{
		
		User user = new User();
	
		/**
		 * 	获取类类型的方式,这里三种方式获得的结果均相同
		 * */
		
		//第一种方式
		Class clazz1 = User.class;
		
		//第二种方式
		Class clazz2 = user.getClass();
		
		//第三种方式
		Class clazz3 = Class.forName("javaReflex.User");
		
		
		/**
		 * 	通过类类型,可以创建类的对象
		 * 	前提该类必须有一个无参的构造器
		 * 	需要强制类型转换
		 * */
		
		User user2 = (User)clazz2.newInstance();
	}
}
```



## 类类型的辨认

对基本数据类型和引用数据类型进行辨认.
``` java
public class ClassType {

	public static void main(String[] args) {
		Class<Integer> intClass = int.class;
		
		Class<String> stringClass = String.class;
		
		System.out.println("基本数据类的名字"+intClass.getSimpleName());
		System.out.println("基本数据类的全名"+intClass.getName());
		System.out.println("引用数据类的名字"+stringClass.getSimpleName());
		System.out.println("引用数据类的全名"+stringClass.getName());
	}
}
```



## 反射获得类的信息

**Class**
| 返回值        | 方法                                                | 说明                                                         |
| ------------- | --------------------------------------------------- | ------------------------------------------------------------ |
| Class         | `getClass()`                                        | 获得对象的类类型                                             |
| String        | `getName()`                                         | 获得对象的类名                                               |
| Method[]      | `getMethods()`                                      | 获得对象的公共方法                                           |
| Method[]      | `getDeclaredMethods()`                              | 获得类所特有的方法                                           |
| Method        | `getMethod(String name,Class<?>... parameterTypes)` | 传入方法名,参数类型对象数组获得相应方法,如果是无参数的方法,则不需要传入参数类型数组 |
| Field[]       | `getFields()`                                       | 获得所有成员变量                                             |
| Constructor[] | `getConstructors()`                                 | 获得构造器方法                                               |

**Method**

| 返回值  | 方法                    | 说明                                                         |
| ------- | ----------------------- | ------------------------------------------------------------ |
| String  | `getName()`             | 获得方法名                                                   |
| Class[] | `getParameterTypes()`   | 获得参数对象类型                                             |
| Class   | `getReturnType()`       | 获得返回值对象类型                                           |
| Object  | `invoke(Object object)` | 传入参数执行方法,如果是无参方法,则无需参数;对于非静态方法,需要传入`newInstance()`后的类类型对象. |

**Field**

| 返回值 | 方法        | 说明         |
| ------ | ----------- | ------------ |
| String | `getName()` | 获得参数名   |
| Class  | `getType()` | 获得参数类型 |

**Constructor**

| 返回值  | 方法                  | 说明               |
| ------- | --------------------- | ------------------ |
| String  | `getName()`           | 获得构造器的名字   |
| Class[] | `getParameterTypes()` | 获得构造器参数列表 |


``` java
public class GetClassAllInfo {

	/**打印类的所有信息
	 */
	public static  void printInfo(Object object) throws Exception {
		//要获取类的信息,首先要获取类的类型
		Class<?> clazz = object.getClass();
		
		//获取类的名称
		System.out.println("传入的类名称为:"+clazz.getName());
		
		/**
		 * 	Method类,代表了对象的方法
		 * 	getMethods()方法获取的所有public的函数,包括父类继承而来的
		 * 	getDeclaredMethods()获取的是该类自己创建的方法,不问访问权限
		 * */
		Method[] methods = clazz.getMethods();
		
		//遍历所有的方法,打印其方法名,参数信息,返回值信息
		for (Method method : methods) {
			//得到其中一个方法的返回值的类类型
			Class<?> returnType = method.getReturnType();
			//获得方法名
			String methodName = method.getName();
			System.out.println("方法:"+methodName+"()\t返回值:\t"+returnType.getName());
			//获得参数列表
			Class<?>[] paramType = (Class<?>[]) method.getParameterTypes();
			System.out.print("方法:"+methodName+"()\t参数:\t");
			for (Class<?> param : paramType) {
				System.out.print(param.getName()+" ");
			}
			System.out.println();
		}
		/**	遍历所有的成员变量
		 * 	Field类封装了关于成员变量的操作
		 * 	getFields()方法获得所有的public的成员变量
		 * 	getDeclaredFields()方法获得所有的自己声明的成员变量的信息
		 * */
		
		Field[] fields = clazz.getFields();
		//遍历每一个成员变量
		for (Field field : fields) {
			Class<?> fieldType = field.getType();
			String fieldName = field.getName();
			System.out.println("成员变量的名字:"+fieldName);
			System.out.println("成员变量的类型:"+fieldType);
		}
		
		/**	遍历构造器
		 * 	Constructor封装类关于构造方法的操作
		 * 	getConstructors获得所有的public的构造函数
		 * 	getDeclaredConstructors得到所有的构造函数
		 * */
		Constructor<?>[] constructors = clazz.getConstructors();
		for (Constructor<?> constructor : constructors) {
			//获取构造函数的名字
			String constructorName = constructor.getName();
			System.out.println("构造函数的名字为:"+constructorName);
			//获得参数列表
			Class<?>[] constructorTypes = (Class<?>[]) constructor.getParameterTypes();
			System.out.print("方法:"+constructor+"()\t参数:\t");
			for (Class<?> constructorType : constructorTypes) {
				System.out.print(constructorType.getName()+" ");
			}
			System.out.println();
		}
	}
	
	
	public static void main(String[] args) throws Exception{
		User user = new User();
		printInfo(user);
	}
}
```



## 通过反射调用对象方法

1. 获得对象的类类型.
2. 调用`getMethod("play",String.class,int.class)`或`getMethod("play", new Class<?>[] {String.class,int.class})`获得该类的该方法对象.
3. 对于普通方法调用`invoke(clazz.newInstance(), "王者荣耀",10)`,实例化后传入参数,进行方法调用.如果是静态方法,则不需要进行实例化.
4. 对于无参方法,直接`getMethod("say")`根据方法方法名获得方法对象后,调用`invoke(clazz.newInstance())`实例化对象后执行方法,如果是静态方法,则不需要实例化.


**创建对象及对象方法**

``` java
public class User {

	public User(){}
	
	public User(String username){
		this.username = username;
	}
	
	public String username;
	
	public int age;
	
	public void say() {
		System.out.println("我是人,我能说话!!");
	}
	
	public void sing(String singName) {
		System.out.println("我是人,我能唱歌,来一首"+singName+"!!");
	}
	
	public void play(String game,int min) {
		System.out.println("我是人,我能跑,我能玩"+game+"不吃不喝"+min+"小时!!");
	}
}
```



**调用对象方法**

``` java
public class MethodInvoke {
	
	@SuppressWarnings("unused")
	public static void  show() throws Exception{
		//获取方法对象的类类型
		Class<?> clazz = User.class;
		
		/**
		 * 	调用方法
		 * */
		//根据名称和参数列表获取参数列表
//		Method method = clazz.getMethod("play", new Class<?>[] {String.class,int.class});
		Method method = clazz.getMethod("play",String.class,int.class);		//也可以这样传入可变参数列表
		//反射调用方法,有返回值.
		//必须实例化:.newInstance()	或者,将该方法定义为static修饰的,或者传入对象 
//		Object object = method.invoke(clazz.newInstance(), new Object[]{ "王者荣耀",10});
		Object object = method.invoke(clazz.newInstance(), "王者荣耀",10);
		
		//调用无参的方法
		Method method2 = clazz.getMethod("say");	
		method2.invoke(clazz.newInstance());
	}
	
	public static void main(String[] args)throws Exception{
		
		show();
	}
}

```



## 参考代码

- [*javaReflex*][1]







[1]:https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaReflex