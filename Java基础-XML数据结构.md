---
title: Java基础-XML数据结构
abbrlink: 54f6c421
date: 2020-01-30 17:01:05
categories:
  - Java
  - XML
tags: [Java, XML]
---





# JAVA中XML文件的读写

## 简介
对Java中Xml文件或数据进行读写操作.



## 示例代码

[GitHub地址](https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/xmlReadAndWriter)

### `xmlRead`包

[`xmlRead`][1] 对XML文件的读操作

- DOM解析
- SAX解析
- JDOM解析
- DOM4J解析

### `xmlWriter`包

[`xmlWriter`][2]对XML文件的写操作

- DOM解析
- SAX解析
- JDOM解析
- DOM4J解析



## 读取XML
对Java中Xml文件或数据进行读操作.



### 背景知识

JDK为我们提供了基础的DOM,SAX方式的用来解析XML.
其中DOM方式是基于DOM文件树的方式进行解析,而SAX是基于事件驱动的方式进行解析.
- DOM解析将结果以树的形式保存在内存中,便于理解和修改,但是对内存消耗大,不适合解析过大的文件.
- SAX采用事件驱动的方式,对内存消耗少, 但是不便于对文档修改和多处访问,一般仅用于处理XML数据.

在DOM解析基础上,扩展出来JDOM和DOM4J方式,均是针对于JAVA语言的解析方式
- JDOM方式仅使用具体类和大量Collections类,扩展性较差.
- DOM4J方式扩展了XML解析, 使用接口和抽象类,易于使用

速度:
SAX>DOM4J>JDOM>DOM



### DOM方式解析XML

通过读取文件创建DOM文档对象,将DOM树加载进入内存中,可以对任意节点进行访问.
解析过程:
1. 读取文件,创建`document`对象
2. 获得感兴趣的节点们.
3. 遍历节点,获取属性和属性值
4. 获取子节点,其中空白也将会被看做一个节点 

``` java
public static void main(String[] args) throws Exception{

	//引入文件
	File file = new File("src/xmlReadAndWriter/xmlRead/Books.xml");

	//创建一个DocumentBuilderFactory对象
	DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();

	//创建一个DocumentBuilder对象
	DocumentBuilder documentBuilder = documentBuilderFactory.newDocumentBuilder();

	//解析XML文件,创建Document对象
	Document document = documentBuilder.parse(file);

	//解析Document对象
	//获得book节点.使用Dom方式解析可以获取任何我们感兴趣的节点,不需要层级关系
	NodeList bookLists = document.getElementsByTagName("book");	//返回的为一个节点list

	//遍历节点集合
	for (int i = 0; i < bookLists.getLength(); i++) {
		//解析每个book节点
		Node bookNode = bookLists.item(i);
		//节点名字
		String bookNodeName = bookNode.getNodeName();
		//获取book节点属性Map
		NamedNodeMap bookAttributes = bookNode.getAttributes();
		//遍历属性
		for (int j = 0; j < bookAttributes.getLength(); j++) {
			//莫得book节点每个属性对象
			Node attribute = bookAttributes.item(j);		//遍历每个属性
			//获得book节点每个属性
			String attributeName = attribute.getNodeName();		//获得属性名
//				String attributeValue = attribute.getNodeValue();	//互动属性值
			String attributeValue = ((Element) bookNode).getAttribute("id");	//将book节点强制换换为Element接口,其getAttribute("属性名")方法获取属性名对应的属性值
			System.out.println("节点:"+bookNodeName +"	属性名:"+attributeName+"	属性值:"+attributeValue);
		}

		//获得book节点的子节点,在解析时,空白也为一个节点.
		NodeList bookChildNodes = bookNode.getChildNodes();
		for (int k = 0; k < bookChildNodes.getLength(); k++) {
				Node bookChildNode = bookChildNodes.item(k);
			if (bookChildNode.getNodeType() == Node.ELEMENT_NODE) {	//如果为元素类型,则遍历
				String bookChildNodeName = bookChildNode.getNodeName();	//获得节点的名字
				String bookChildNodeValue = bookChildNode.getFirstChild().getNodeValue();	//默认节点的文本信息属于子节点内的值	
//					String bookChildNodeValue = bookChildNode.getTextContent();		//获得节点内的文本信息,仅限于获得纯文本的节点内容
				System.out.println("子节点:"+ bookChildNodeName+"	节点值:"+bookChildNodeValue);	//空白和换行符的为#text
			}

		}
	}
}
```



### SAX方式解析XML

SAX方式通过解析时加载文件和解析函数,完成基于事件的XML文档解析.
SaxHandler的编写需要继承`DefaultHandler.class`类,重写其中的解析方法,编写解析逻辑.

| 方法                                                         | 说明             |
| ------------------------------------------------------------ | ---------------- |
| `public void startDocument()`                                | 开始解析XML      |
| `startElement(String uri, String localName, String qName, Attributes attributes)` | 开始解析节点     |
| `characters(char[] ch, int start, int length)`               | 解析节点内字符串 |
| `endElement(String uri, String localName, String qName)`     | 结束解析节点     |
| `endDocument()`                                              | 结束解析XML      |

``` java
public class SaxHandler extends DefaultHandler{
	/**开始解析xml*/
	@Override
	public void startDocument() throws SAXException {
		super.startDocument();
		System.out.println("--------XML文件开始解析-------");
	}
	/**开始解析节点方法*/
	@Override			//qName:节点名	attributes:属性
	public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
		super.startElement(uri, localName, qName, attributes);
		System.out.println("-----节点:"+qName+"解析开始-----");
		//只有book标签有属性值.需要解析属性
		if (qName.equals("book")) {
			//遍历获取属性信息
			for (int i = 0; i < attributes.getLength(); i++) {
				String attributeName = attributes.getQName(i);		//遍历获取属性名
				String attributeValue = attributes.getValue(i);		//遍历获取属性值
//				String attributeValue = attributes.getValue("id");		//知道属性名,直接获取属性值
				System.out.println("节点:"+qName+"	属性名:"+attributeName+"	属性值:"+attributeValue);
			}
		}
		//解析只有只有文本的节点,即不是bookstore和book的节点
		if ( !qName.equals("bookstore") && !qName.equals("book") ) {
			String childNodeName = qName;
			System.out.print("节点:"+childNodeName);
		}
	}
	/**解析节点内的文本信息*/
	@Override			//ch:节点中的文本信息		start:开始的位置	length:文本的长度
	public void characters(char[] ch, int start, int length) throws SAXException {
		super.characters(ch, start, length);
		String childNodeValue = new String(ch, start, length);
		//SAX解析会将空格和换行解析为xml元素
		if (!"".equals(childNodeValue.trim())) {
			System.out.println("	节点值:"+childNodeValue);
		}
	}
	/**结束解析节点方法*/
	@Override
	public void endElement(String uri, String localName, String qName) throws SAXException {
		super.endElement(uri, localName, qName);
		//是否针对book节点结束
		System.out.println("-----节点:"+qName+"解析结束-----");	//每个节点结束后都触发
	}
	/**结束解析xml*/
	@Override
	public void endDocument() throws SAXException {
		super.endDocument();
		System.out.println("--------XML文件结束解析-------");
	}
}
```

通过传入继承`DefaultHandler`类的实现类和需要解析的文档对象,完成XML的解析.

``` java
public static void main(String[] args) throws Exception {

	//引入文件
	File xml = new File("src/xmlReadAndWriter/xmlRead/Books.xml");
	//获取SAXParserFactory实例
	SAXParserFactory saxParserFactory = SAXParserFactory.newInstance();
	//获取newSAXParser实例
	SAXParser saxParser = saxParserFactory.newSAXParser();
	//开始解析
	saxParser.parse(xml, new SaxHandler());
}
```



### JDOM方式解析XML

与DOM解析方式相似,不过不会将空字符串解析为子节点.
1. 加载文件,生成DOM对象
2. 获得根节点
3. 获得节点属性和属性值
4. 读取和遍历子节点.

``` java
public static void main(String[] args) throws Exception{
	//引入文件
	File xml = new File("src/xmlReadAndWriter/xmlRead/Books.xml");

	//创建SAXBuilder
	SAXBuilder saxBuilder = new SAXBuilder();
	//传入文件,创建Document对象
	Document document = saxBuilder.build(xml);
	//解析,生成根节点
	Element rootElement = document.getRootElement();
	//获得根节点下的子节点
	List<Element> childrenElements = rootElement.getChildren();
	//变量子节点
	for (Element children : childrenElements) {
		//节点名
		String childrenName = children.getName();
		//获得节点的属性
		List<Attribute> childrenAttributes = children.getAttributes();
		//变量属性集合
		for (Attribute childrenAttribute : childrenAttributes) {
			String childrenAttributeName  = childrenAttribute.getName(); //获得属性名
			String childrenAttributeValue  = childrenAttribute.getValue(); //获得属性值
			System.out.println("节点名:"+childrenName+"	属性:"+childrenAttributeName+"	属性值:"+childrenAttributeValue);
		}

		//获得子节点的子节
		List<Element> grandsonElements = children.getChildren();
		//遍历孙子节点
		for (Element grandsonElement : grandsonElements) {
			String grandsonElementName = grandsonElement.getName();
			String grandsonElementvalue = grandsonElement.getValue();
			System.out.println("孙子节点名: "+grandsonElementName+"	节点值: "+grandsonElementvalue);
		}
	}
}
```



### DOM4J方式解析XML

解析方式最为简单灵活
1. 读取XML文件
2. 获取根节点
3. 迭代子节点
4. 迭代获取节点属性

``` java
public static void main(String[] args) throws Exception{

	//引入文件
	File xml = new File("src/xmlReadAndWriter/xmlRead/Books.xml");
	//创建SAXReader对象
	SAXReader saxReader = new SAXReader();
	//加载XML,生成Document对象
	Document document = saxReader.read(xml);
	//获得根节点
	Element rootElement = document.getRootElement();
	//获得根节点下的子节点
	Iterator<Element> childrenIterator = rootElement.elementIterator();
	//遍历迭代器
	while (childrenIterator.hasNext()) {
		//获得book的节点
		Element childrenElement = (Element) childrenIterator.next();
		//获得book节点的节点名
		String childrenElementName = childrenElement.getName();
		//获得属性集合
		List<Attribute> childrenAttributes = childrenElement.attributes();
		//遍历属性集合
		for (Attribute childrenAttribute : childrenAttributes) {
			String childrenAttributeName = childrenAttribute.getName();
			String childrenAttributeValue = childrenAttribute.getValue();
			System.out.println("节点:"+childrenElementName+"	属性名:"+childrenAttributeName+"	属性值:"+childrenAttributeValue);
		}
		//解析book节点的子节点
		Iterator<Element> grandsonIterator = childrenElement.elementIterator();
		while (grandsonIterator.hasNext()) {
			Element grandsonElement = (Element) grandsonIterator.next();
			String grandsonName = grandsonElement.getName();
			String grandsonValue = grandsonElement.getText();
			System.out.println("节点名:"+grandsonName+"	节点值:"+grandsonValue);
		}
	}
}
```



## 写入XML
对Java中Xml文件或数据进行读操作.



### 背景知识
JDK为我们提供了基础的DOM,SAX方式的用来解析XML.
其中DOM方式是基于DOM文件树的方式进行解析,而SAX是基于事件驱动的方式进行解析.

- DOM解析将结果以树的形式保存在内存中,便于理解和修改,但是对内存消耗大,不适合解析过大的文件.
- SAX采用事件驱动的方式,对内存消耗少, 但是不便于对文档修改和多处访问,一般仅用于处理XML数据.

在DOM解析基础上,扩展出来JDOM和DOM4J方式,均是针对于JAVA语言的解析方式
- JDOM方式仅使用具体类和大量Collections类,扩展性较差.
- DOM4J方式扩展了XML解析, 使用接口和抽象类,易于使用

速度:
SAX>DOM4J>JDOM>DOM



### DOM方式生成XML

通过创建`Document`对象,为其赋值节点和属性,将其写入文件中,最终生成XML文件.
1.创建`Document`对象
2.创建节点,设置属性并装载进入根节点
3.创建子节点,并装载进入父节点中.
4.设置转换格式,输出为XML文件

``` java
public static void main(String[] args) throws Exception{
	//引入文件
	File xml = new File("src/xmlReadAndWriter/xmlWrite/Books.xml");

	//创建一个DocumentBuilderFactory对象
	DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();

	//创建一个DocumentBuilder对象
	DocumentBuilder documentBuilder = documentBuilderFactory.newDocumentBuilder();

	//生成XML文件,创建Document对象
	Document document = documentBuilder.newDocument();
	document.setXmlStandalone(true);	//设置standalone为yes,不显示
	document.setXmlVersion("1.0");		//设置版本
	//创建节点,并添加到文件中,第一个节点为根节点
	Element bookStrore = document.createElement("bookstore");
	document.appendChild(bookStrore);
	//创建子节点,设置属性,并添加到父节点中.
	Element book = document.createElement("book");
	book.setAttribute("id", "1");
	bookStrore.appendChild(book);
	//为子节点设置带文本的子节点
	Element name = document.createElement("name");
	name.setTextContent("冰与火之歌");
	book.appendChild(name);
	Element author = document.createElement("author");
	author.setTextContent("乔治马丁");
	book.appendChild(author);
	Element year = document.createElement("year");
	year.setTextContent("2014");
	book.appendChild(year);
	Element price = document.createElement("price");
	price.setTextContent("98$");
	book.appendChild(price);

	//创建转换工厂,设置格式,进行输出
	TransformerFactory transformerFactory = TransformerFactory.newInstance();
	Transformer transformer = transformerFactory.newTransformer();
	transformer.setOutputProperty(OutputKeys.ENCODING, "UTF-8");	//字符编码
	transformer.setOutputProperty(OutputKeys.INDENT, "yes");		//是否换行
	transformer.transform(new DOMSource(document), new StreamResult(xml));
}
```



### SAX方式生成XML

与解析时相似,基于事件生成XML.
1. 通过传入文件对象,生成`TransformerHandler`对象
2. 创建`Transformer`对象,设置版本信息
3. 创建节点,属性对象,开启文档对象
4. 循环开启节点,设置属性,结束节点
5. 最终结束文档对象.

``` java
public static void main(String[] args) throws Exception{

	//引入文件
	File xml = new File("src/xmlReadAndWriter/xmlWriter/Books.xml");

	//创建SAXTransformerFactory
	SAXTransformerFactory saxTransformerFactory = (SAXTransformerFactory) SAXTransformerFactory.newInstance();
	//创建TransformerHandler
	TransformerHandler transformerHandler = saxTransformerFactory.newTransformerHandler();
	//创建Transformer
	Transformer transformer = transformerHandler.getTransformer();
	//设置xml文件的属性,注意位置,在创建时设置
	transformer.setOutputProperty(OutputKeys.VERSION, "1.0");		//设置版本号
	transformer.setOutputProperty(OutputKeys.ENCODING, "UTF-8");	//设置编码
	transformer.setOutputProperty(OutputKeys.INDENT, "yes");	 	//设置换行
	//创建Result对象,使其与Handler关联
	Result result = new StreamResult(new FileOutputStream(xml));
	transformerHandler.setResult(result);
	transformerHandler.startDocument();		//打开文档,进行节点生成
	//创建属性
	AttributesImpl attributes =  new AttributesImpl();
	//创建开始节点
	transformerHandler.startElement("", "", "bookstore", attributes);
	//创建子节点
	//属性清除,子节点创建属性
	attributes.clear();
	attributes.addAttribute("", "", "id", "", "1");
	//创开始子节点
	transformerHandler.startElement("", "", "book", attributes);
	//清除属性,创建子节点文本子节点
	attributes.clear();
	//文本节点
	transformerHandler.startElement("", "", "name", attributes);
	String name = "冰与火之歌";
	transformerHandler.characters(name.toCharArray(), 0,name.toCharArray().length);
	transformerHandler.endElement("", "", "name");
	//文本节点
	transformerHandler.startElement("", "", "author", attributes);
	String author = "乔治马丁";
	transformerHandler.characters(author.toCharArray(), 0,author.toCharArray().length);
	transformerHandler.endElement("", "", "author");
	//文本节点
	transformerHandler.startElement("", "", "year", attributes);
	String year = "2014";
	transformerHandler.characters(year.toCharArray(), 0,year.toCharArray().length);
	transformerHandler.endElement("", "", "year");
	//文本节点
	transformerHandler.startElement("", "", "price", attributes);
	String price = "98$";
	transformerHandler.characters(price.toCharArray(), 0,price.toCharArray().length);
	transformerHandler.endElement("", "", "price");

	//创建结束子节点
	transformerHandler.endElement("", "", "book");
	//创建结束节点
	transformerHandler.endElement("", "", "bookstore");
	transformerHandler.endDocument(); 		//关闭文档
}
```



### JDOM方式生成RSS

JDOM方式不仅可以生成标准的XML文件,也可以生成RSS文件,进行互联网传播.

``` java
public static void main(String[] args) throws Exception{
	//创建输出文件对象
	File xml = new File("src/xmlReadAndWriter/xmlWriter/News.xml");
	//生成Document对象
	Document document = new Document();
	//生成根节点,并设置属性,放入文档中
	Element rootRss = new Element("rss");
	rootRss.setAttribute("version", "3.0");
	document.setRootElement(rootRss);
	//设置子节点,及内容
	Element channel = new Element("channel");	//子节点<channel>
	channel.setText("新闻频道");
	rootRss.addContent(channel);
	Element title = new Element("title");	//子节点<title>
	title.setText("肚子饿了!!...>o<...");
	rootRss.addContent(title);

	CDATA code = new CDATA("不需要转移的代码片段.<JavaScript:alert('123')>");	//子节点
	rootRss.addContent(code);


	//设置格式
	Format format = Format.getPrettyFormat();
	format.setEncoding("GBK");	 			//在格式输出时,设置GBK编码
	//输出
	XMLOutputter xmlOutputter = new XMLOutputter(format);
	xmlOutputter.output(document, new FileOutputStream(xml));
}
```



### DOM4J方式生成RSS

DOM4J方式生成RSS文件

``` java
public static void main(String[] args) throws Exception{
	//创建输出文件对象
	File xml = new File("src/xmlReadAndWriter/xmlWriter/News.xml");
	//创建Document对象
	Document document = DocumentHelper.createDocument();
	//创建根节点<rss>
	Element rootRss = document.addElement("rss");
	//描述根节点,增加属性
	rootRss.addAttribute("version", "3.0");
	//生成子节点,及内容
	Element channel = rootRss.addElement("channel");	//子节点<channel>
	channel.setText("新闻频道");
	Element title = rootRss.addElement("title");	//子节点<title>
	title.setText("肚子饿了!!...>o<...");
	//设置格式
	OutputFormat outputFormat = OutputFormat.createPrettyPrint();
	outputFormat.setEncoding("GBK");	 			//在格式输出时,设置GBK编码
	//生成xml文件
	XMLWriter xmlWriter = new XMLWriter(new FileOutputStream(xml),outputFormat );
	xmlWriter.setEscapeText(false);					//设置是否进行转移,将文本中的字符进行转移
	xmlWriter.write(document);
	xmlWriter.close();
}
```






[1]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/xmlReadAndWriter/xmlRead
[2]: https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/xmlReadAndWriter/xmlWriter