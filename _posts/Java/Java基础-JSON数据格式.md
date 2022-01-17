---
title: Java基础-JSON数据格式
abbrlink: cb8b523e
date: 2020-01-30 12:11:43
categories:
  - Java
  - JSON
tags: [Java, JSON]
---

> JSON 的简单使用

<!--more-->



## 简介

一种与开发语言无关的、轻量级的数据格式。全称 `JavaScript Object Notation`
该练习是对 `JSON` 和 `GSON` 两种数据格式的创建和解析.



## 数据结构

### 基本类型

| 类型    | 名称                         |
| ------- | ---------------------------- |
| String  | 字符串类型, 必须加双引号表示 |
| number  | 数字类型, 整型或者浮点型     |
| boolean | 布尔类型,true/false          |
| null    | 空值或者空串                 |



### 对象结构

使用一组大括号`{}`表示的键值对,在标准模式下,键名也需要用`"  "`双引号包围.键必须为String类型,而值可以为基本类型或者其他数据结构.
![对象数据结构][1]



### 数组结构

使用中括号`[]`来起始，并用逗号`,`来分隔元素。
![数组数据结构][2]



## 包结构

* `bean`  封装对象模型
* `gson`  创建和解析`gson`数据
* `json`   创建和解析`json`数据



## 子包描述

### `bean`包
封装了一个用来作为数据对象的模型`Student`类,注意其中属性修饰

 - `@SerializedName` 修饰后属性名强制指定,不会被大小写改变,`gson`所特有的注解
 - `transient`关键字修饰的属性不会被映射为数据结构

### `gson`类
- `MakeGSONObject` 创建gson对象
- `ParseGSONObject` 解析gson对象

#### 创建gson对象

 1. 使用javaBean对象完成JSON数据结构的创建
其中`setFieldNamingStrategy()`方法重写了对gson对象的键的名称重写,当对象属性名为`age`时,对其进行别名重写.

``` java
/**使用javaBean创建GSON*/
public static void createGSONObjectByJavaBean() throws Exception {
	Student student = new Student();
	student.setName("张谦");
	student.setAge(24.2);
	student.setBirthday(new SimpleDateFormat("YYYY-MM-DD").parse("1994-04-12"));	//日期转换
	student.setSchool("河南科技学院");
	student.setSingle(false);
	student.setCar(null);
	//创建GSON对象
//		Gson gson = new Gson();	//使用直接new的方式创建
	GsonBuilder gsonBuilder = new GsonBuilder();
	gsonBuilder.setPrettyPrinting();	//漂亮的格式打印
	gsonBuilder.setFieldNamingStrategy(new FieldNamingStrategy() {	//
		//
		@Override
		public String translateName(Field field) {		//field,表示属性名
			if (field.getName().equals("age")) {		//表示属性名是否相等
				return "age是:";
			}
			return field.getName();
		}
	});
	Gson gson = gsonBuilder.create();	//创建JSON对象
	System.out.println("GSON开始:");
	System.out.println(gson.toJson(student));
	System.out.println("-------------------");
}
```

#### 解析gson对象

 1. 读取文件解析gson

``` java
/**通过文件解析*/
public static void parseGSONObjectByFile() throws Exception{
	File file = new File("F:\\JAVA_WorkSpace\\JSON\\src\\json\\GSON对象.gson");	//文件的地址
	//将文件转为String
	String content = FileUtils.readFileToString(file,"utf8");
	//设置日期格式
	Gson gson = new GsonBuilder().setDateFormat("yyyy-MM-dd").create();	//日期格式
	//设置传入的文件,并转为的JavaBean
	Student student = gson.fromJson(content, Student.class);
	System.out.println("姓名:"+student.getName());
	System.out.println("年龄:"+student.getAge());					//需要大小写一致,并在JavaBean中声明的
	System.out.println("生日:"+student.getBirthday());
	System.out.println("学校:"+student.getSchool());
	System.out.println("专业:"+student.getMajor());
	System.out.println("单身:"+student.isSingle());
}
```


### `json`类
- `MakeJSONObject` 创建json对象
- `ParseJSONObject` 解析json对象

#### 创建json对象

 1. 直接创建json对象

``` java
/**直接创建JSON对象*/
public static void createJSONObjectByJSONObject() throws JSONException{
	JSONObject jsonObject = new JSONObject();
	Object nullObject = null;	//创建空对象,表示null本身
	jsonObject.put("name", "张谦");
	jsonObject.put("age", 24.2);
	jsonObject.put("birthday", "1994-4-12");
	jsonObject.put("school", "河南科技学院");
	jsonObject.put("marjor", new String[]{"机械设计","电子工程","软件工程"});
	jsonObject.put("single", false);
	jsonObject.put("car", nullObject);	//传入空对象
	//输出
	System.out.println("JSON解析:");
	System.out.println(jsonObject.toString());
	System.out.println("-------------------");

}
```

 2. 通过map创建json对象

``` java
/**通过Map创建JSON对象*/
public static void createJSONObjectByMap() throws JSONException{
	Map<String, Object> map = new HashMap<String, Object>();
	Object nullObject = null;	//创建空对象
	map.put("name", "张谦");
	map.put("age", 24.2);
	map.put("birthday", "1994-4-12");
	map.put("school", "河南科技学院");
	map.put("major", new String[]{"机械设计","电子工程","软件工程"});
	map.put("single", false);
	map.put("car", nullObject);	//传入空对象
	//将Map解析为JSON对象
	JSONObject jsonObject = new JSONObject();
	jsonObject.accumulateAll(map);
	//输出
	System.out.println("JSON解析:");
	System.out.println(jsonObject.toString());
	System.out.println("-------------------");
}	
```

 3. 通过javaBean创建json对象

``` java
/**将javaBean解析JSON*/
public static void createJSONObjectByJavaBean() throws Exception {
	Student student = new Student();
	student.setName("张谦");
	student.setAge(24.2);
	student.setBirthday(new SimpleDateFormat("YYYY-MM-DD").parse("1994-04-12"));	
	student.setSchool("河南科技学院");
	student.setSingle(false);
	student.setCar(null);
	//将JavaBean解析为JSON对象
	JSONObject jsonObject = JSONObject.fromObject(student);		//对日期支持不好	
	System.out.println("JSON解析:");
	System.out.println(jsonObject.toString());
	System.out.println("-------------------");
}
```

#### 解析json对象

 1. 通过文件解析json对象

``` java
/**ʹ解析JSON文件*/
public static void parseJSONObjectByFile() throws Exception{
	File file = new File("F:\\JAVA_WorkSpace\\JSON\\src\\json\\JSON对象.json");	//传入文件
	//将文件转为String
	String content = FileUtils.readFileToString(file,"utf8");
	//解析为JSON对象
	JSONObject jsonObject = JSONObject.fromObject(content);
	//获得JSON对象的属性
	String name = jsonObject.getString("name");
	System.out.println("姓名:"+name);
	float age = (float) jsonObject.getDouble("age");
	System.out.println("年龄:"+age);
	Date birthday = new SimpleDateFormat("YYYY-MM-DD").parse(jsonObject.getString("birthday"));
	System.out.println("生日:"+birthday);
	String school = jsonObject.getString("school");
	System.out.println("学校:"+school);
	JSONArray jsonArray = jsonObject.getJSONArray("major");
	for (Object object : jsonArray) {
		System.out.println("专业:"+object.toString());
	}
	boolean single = jsonObject.getBoolean("single");
	System.out.println("单身:"+single);
}
```


[1]: https://raw.githubusercontent.com/jionjion/Picture_Space/master/WorkSpace/Java/javaBase/json-01.png
[2]: https://raw.githubusercontent.com/jionjion/Picture_Space/master/WorkSpace/Java/javaBase/json-02.png