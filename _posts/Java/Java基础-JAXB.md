---
title: Java基础-JAXB
abbrlink: 59f2ddee
date: 2020-01-30 11:56:49
categories:
  - Java
  - JAXB
tags: [Java, JAXB]
---

> JAXB 的使用, 将对象与XML进行转换

<!-- more -->



# 简介

介绍Java8自带工具中,对JAXB的使用
JAXB工具类实现了将java对象转为xml对象,避免了直接使用XML解析的繁琐.



## 常用注解说明

以下类多用在java对象中,标识生成的xml的数据结构
`@XmlType` 				    类注解,标识类对应生成的xml类型
`@XmlRootElement` 			类注解,标识生成的xml的根节点,及节点约束空间
`@XmlAttribute`			    共有方法或者共有属性注解,标识该属性作为根节点的xml属性
`@XmlJavaTypeAdapter`		共有方法或者共有属性注解,指定特定的类作为转换解析该属性的方法
`@XmlElementWrapper`		共有方法或者共有属性注解,为该节点包裹特定的标签




## 将Java对象转为XML对象 


### 构建JavaBean

注意,在方法中,如果属性为私有属性,则不能在对应的私有属性中添加`@XmlElement`注解,而是该属性对应得get/set方法中.


``` java
// XmlType标识该XML的属性.namespace表示命名空间;propOrder指定生成的xml对象的标签顺序
@XmlType(namespace = "https://www.w3.org/2001/XMLSchema", propOrder = { "name", "age", "address", "postcode", "birthday" })
// XmlRootElement标识这是一个xml的根标签,name指明标签的名字,缺省为类名.namespace:指明该标签的命名空间
@XmlRootElement(name = "student", namespace = "www.jionjion.top/java/example/bean")
public class Student {
	
	private Integer id;
	// 标识这是一个属性,并为必输,属性标注在根节点中
	@XmlAttribute(name="id", required=true)
	public void setId(Integer id) {
		this.id = id;
	}

	private String name;
	// 在私有属性中,@XmlElement注解仅在get()/set()方法使用,如果为共有属性,则不做限制
	//标识这是一个标签元素,可以指定标签级别的,且为必输
	@XmlElement(name = "name", namespace = "www.jionjion.top/java/example/bean/name", required=true)
	public void setName(String name) {
		this.name = name;
	}

	private Double age;
	@XmlElement(name = "age")
	public void setAge(Double age) {
		this.age = age;
	}
	
	private String address;
	@XmlElement(name = "address")
	public void setAddress(String address) {
		this.address = address;
	}

	private Long postcode;
	// 标识这是一个属性,并为必输
	@XmlElement(name = "postcode")
	public void setPostcode(Long postcode) {
		this.postcode = postcode;
	}

	private Calendar birthday;
	
	@XmlElement(name = "birthday")
	// 适配器,限定转换格式
	@XmlJavaTypeAdapter( DateAdapter.class )
	public void setBirthday(Calendar birthday) {
		this.birthday = birthday;
	}

	public String getName() {
		return name;
	}
	
	public Integer getId() {
		return id;
	}

	public Double getAge() {
		return age;
	}

	public String getAddress() {
		return address;
	}

	public Long getPostcode() {
		return postcode;
	}

	public Calendar getBirthday() {
		return birthday;
	}

	@Override
	public String toString() {
		return "Student [id=" + id + ", name=" + name + ", age=" + age + ", address=" + address + ", postcode="
				+ postcode + ", birthday=" + birthday + "]";
	}
}
```



### 测试方法

``` java
	@Test
	public void testMarshalObject() throws JAXBException{
		// 创建bean
		Student student = new Student();
		student.setName("Jion");
		student.setId(1);
		student.setAge(23.5D);
		student.setAddress("上海");
		student.setPostcode(1000L);
		// 日期
		Calendar birthday = Calendar.getInstance();
		birthday.set(2019, 0, 1);
		student.setBirthday(birthday);
		
		// 容器
		JAXBContext jaxbContext = JAXBContext.newInstance( Student.class );
		// 转换类
		Marshaller jaxbMarshaller = jaxbContext.createMarshaller();
		// XML格式
		jaxbMarshaller.setProperty( Marshaller.JAXB_FORMATTED_OUTPUT, true );
		// 控制台输出
		jaxbMarshaller.marshal( student, System.out );
		// 输出文件
		jaxbMarshaller.marshal( student, new File( "student.xml" ) );
		
		/*
		  	<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
			<ns3:student xmlns:ns2="www.jionjion.top/java/example/bean/name" xmlns:ns3="www.jionjion.top/java/example/bean" id="1">
			    <ns2:name>Jion</ns2:name>
			    <age>23.5</age>
			    <address>上海</address>
			    <postcode>1000</postcode>
			    <birthday>2019-01-01</birthday>
			</ns3:student>
		 */
	}
```



## 将Java列表属性转为多个XML节点

### 构建JavaBean

这里通过静态内部类,完成

``` java
// 根节点
@XmlRootElement(name = "books")
public class BookListWrapper {

	// 私有属性,不支持@XmlElement注解
	private String bookcase;
	
	// 可以注解在get()
	@XmlElement(name="bookcase")
	public String getBookcase() {
		return bookcase;
	}

	public void setBookcase(String bookcase) {
		this.bookcase = bookcase;
	}



	// 子节点们
	private List<book> books;
	
	// 
	public List<book> getBooks() {
		return books;
	}

	// 也可以注解在set()
	@XmlElement(name="book")
	// 被<list>标签,包裹列表
	@XmlElementWrapper(name = "list")
	public void setBooks(List<book> books) {
		this.books = books;
	}




	/** 内部类 */
	public static class book {
		private String name;

		private String author;

		private Double price;

		public book() {
			super();
		}
		
		public book(String name, String author, Double price) {
			super();
			this.name = name;
			this.author = author;
			this.price = price;
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public String getAuthor() {
			return author;
		}

		public void setAuthor(String author) {
			this.author = author;
		}

		public Double getPrice() {
			return price;
		}

		public void setPrice(Double price) {
			this.price = price;
		}
	}
}
```



### 测试方法 

``` java

	@Test
	public void testMarshalObjectList() throws JAXBException {
		// 多个图书
		BookListWrapper books = new BookListWrapper();
		// 图书A
		BookListWrapper.book bookA = new BookListWrapper.book("Oralce指南","Jion",12.5);
		BookListWrapper.book bookB = new BookListWrapper.book("Vue入门","Arise",15.5);

		// 赋值list
		List<BookListWrapper.book> list = new ArrayList<BookListWrapper.book>();
		list.add(bookA);
		list.add(bookB);
		// 子节点
		books.setBookcase("计算机书架");
		books.setBooks(list);
		
		
		// 将ListObject 转为 XML
		// 容器
		JAXBContext jaxbContext = JAXBContext.newInstance( BookListWrapper.class );
		// 转换类
		Marshaller jaxbMarshaller = jaxbContext.createMarshaller();
		// XML格式
		jaxbMarshaller.setProperty( Marshaller.JAXB_FORMATTED_OUTPUT, true );
		// 控制台输出
		jaxbMarshaller.marshal( books, System.out );	
		/*
			<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
			<books>
			    <bookcase>计算机书架</bookcase>
			    <list>
			        <book>
			            <author>Jion</author>
			            <name>Oralce指南</name>
			            <price>12.5</price>
			        </book>
			        <book>
			            <author>Arise</author>
			            <name>Vue入门</name>
			            <price>15.5</price>
			        </book>
			    </list>
			</books>
		 */
	}

```



## 使用适配器指定解析格式

### 适配器类
通过继承`XmlAdapter<T1, T2>`类,实现其中的方法,并指定泛型.`T1`为xml中的数据格式,`T2`为java对象时数据格式
`marshal`将java对象解析为xml时调用;
`unmarshal`将xml解析为java对象时调用.

``` java
public class DateAdapter extends XmlAdapter<String, Calendar> {

	@Override
	public Calendar unmarshal(String str) throws Exception {
		SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
		Date date = format.parse(str);
		Calendar calendar = Calendar.getInstance();
		calendar.setTime(date);
		return calendar;
	}

	@Override
	public String marshal(Calendar calendar) throws Exception {
		SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");		
		Date date = calendar.getTime();
		return format.format(date);
	}
}
```

### 调用方式
在类的属性中,通过`@XmlJavaTypeAdapter`注解使用.如

``` java
private Calendar birthday;

@XmlElement(name = "birthday")

// 适配器,限定转换格式
@XmlJavaTypeAdapter( DateAdapter.class )
public void setBirthday(Calendar birthday) {
	this.birthday = birthday;
}
```



## 将XML转为Java对象

### 测试方法

``` java
	@Test
	public void testUnMarshalObject() throws JAXBException{
	/*
	  	<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
		<ns3:student xmlns:ns2="www.jionjion.top/java/example/bean/name" xmlns:ns3="www.jionjion.top/java/example/bean" id="1">
		    <ns2:name>Jion</ns2:name>
		    <age>23.5</age>
		    <address>上海</address>
		    <postcode>1000</postcode>
		    <birthday>2019-01-01</birthday>
		</ns3:student>
	 */		
		// 文件
		File file = new File( "student.xml" );
		JAXBContext jaxbContext = JAXBContext.newInstance( Student.class );
		// 转换
		Unmarshaller jaxbUnmarshaller = jaxbContext.createUnmarshaller();
		Student student = (Student)jaxbUnmarshaller.unmarshal( file );
		System.out.println( student );
	}
```



## 示例代码

- [*BookListWrapper*][1]   					具有嵌套结构的Java对象类
- [*DateAdapter*][2]						      日期格式化适配器
- [*Student*][3]							           Java对象类
- [*StudentListWrapper*][4]		           Java对象容器列表
- [*Marshal*][5]				                      测试,将java对象转为xml
- [*UnMarshal*][6]			                     测试,将xml转为java对象



[1]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/jaxb/bean/BookListWrapper.java
[2]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/jaxb/bean/DateAdapter.java
[3]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/jaxb/bean/Student
[4]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/jaxb/bean/StudentListWrapper.java
[5]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/jaxb/test/Marshal.java
[6]:https://github.com/jionjion/JAVA_WorkSpace/blob/master/JavaBase/src/jaxb/test/UnMarshal.java

