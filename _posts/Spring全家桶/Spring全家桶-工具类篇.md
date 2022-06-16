---
title: Spring全家桶-工具类篇
typora-root-url: ../../
date: 2022-02-21 09:39:24
categories:
  - Java
  - Spring
tags: [Java, Spring]
---



> 介绍Spring中各种工具类



# 简介

介绍 Spring 中自带的一些工具类



## String 字符操作

StringUtils 使用

```java

/**
 * StringUtils 测试类
 *
 * @author Jion
 */
@Slf4j
public class StringUtilsTest {

    @Test
    public void test() {
        // 判断字符串是否已指定内容开头。忽略大小写
        log.info("是否以指定开头: {}", StringUtils.startsWithIgnoreCase("ABC", "A"));
        log.info("是否以指定开头: {}", StringUtils.startsWithIgnoreCase("ABC", "a"));

        // 判断字符串是否是以指定内容结束,忽略大小写
        log.info("是否以指定结尾: {}", StringUtils.endsWithIgnoreCase("ABC", "C"));
        log.info("是否以指定结尾: {}", StringUtils.endsWithIgnoreCase("ABC", "c"));

        // 是否包含空白符
        log.info("是否包含空白符: {}", StringUtils.containsWhitespace("AB "));

        // 判断字符串非空且长度不为 0,即,Not Empty
        log.info("判断字符串非空且长度不为0: {}", StringUtils.hasLength(" "));

        // 判断字符串是否包含实际内容,即非仅包含空白符,也就是 Not Blank
        log.info("判断字符串是否包含实际内容: {}", StringUtils.hasText(" "));

        // 判断字符串指定索引处是否包含一个子串。
        log.info("判断字符串指定索引处是否包含一个子串: {}", StringUtils.substringMatch("ABC", 1, "BC"));

        // 计算一个字符串中指定子串的出现次数
        log.info("计算一个字符串中指定子串的出现次数: {}", StringUtils.countOccurrencesOf("ABB", "B"));
    }

    @Test
    public void testB() {
        // 查找并替换指定子串
        log.info("查找并替换指定子串: {}", StringUtils.replace("ABC", "ABC", "abc"));

        // 去除头部的特定字符
        log.info("去除头部的特定字符: {}", StringUtils.trimLeadingCharacter("AAB", 'A'));

        // 去除尾部的特定字符
        log.info("去除尾部的特定字符: {}", StringUtils.trimTrailingCharacter("ABB", 'B'));

        // 去除头部的空白符
        log.info("去除头部的空白符: {}", StringUtils.trimLeadingWhitespace("  ABC"));

        // 去除尾部的空白符
        log.info("去除尾部的空白符: {}", StringUtils.trimTrailingWhitespace("ABC  "));

        // 去除头部和尾部的空白符
        log.info("去除头部和尾部的空白符: {}", StringUtils.trimWhitespace("  ABC  "));

        // 删除头尾和中间的空白符
        log.info("删除头尾和中间的空白符: {}", StringUtils.trimAllWhitespace("  A  B  C  "));

        // 删除指定子串
        log.info("删除指定子串: {}", StringUtils.delete("ABCDEFG", "DEFG"));

        // 删除指定字符（可以是多个）
        log.info("删除指定字符: {}", StringUtils.deleteAny("ABCDEFG", "A"));

        // 对数组的每一项执行 trim() 方法
        log.info("对数组的每一项执行trim()方法: {}", Arrays.toString(StringUtils.trimArrayElements(new String[]{"  A ", "  B", "   C"})));

        // 将 URL 字符串进行解码
        log.info("将URL字符串进行解码: {}", StringUtils.uriDecode("https://www.baidu.com", StandardCharsets.UTF_8));
    }

    /**
     * 路径相关工具方法
     */
    @Test
    public void testC() {
        String filePath = "src/main/resources/application.properties";
        // 解析路径字符串,优化其中的 .=> StringUtilsTest.java
        log.info("优化文件路径: {}", StringUtils.cleanPath("./StringUtilsTest.java"));

        // 解析路径字符串,解析出文件名部分 => application.properties
        log.info("获得文件名: {}", StringUtils.getFilename(filePath));

        // 解析路径字符串,解析出文件后缀名 => properties
        log.info("获得文件后缀名: {}", StringUtils.getFilenameExtension(filePath));

        // 比较两个两个字符串,判断是否是同一个路径. 会自动处理路径中的 '..' 和 '.'  => true
        log.info("比较两个路径是否相同: {}", StringUtils.pathEquals("../", "./.././"));

        // 删除文件路径名中的除后缀名后剩余部分  => src/main/resources/application
        log.info("删除文件路径名中的除后缀名后剩余部分: {}", StringUtils.stripFilenameExtension(filePath));

        // 以 . 作为分隔符,获取其最后一部分 => properties
        log.info("文件类型: {}", StringUtils.unqualify(filePath));

        // 以指定字符作为分隔符,获取其最后一部分 => application.properties
        log.info("以指定字符作为分隔符: {}", StringUtils.unqualify("classpath:application.properties", ':'));
    }
}
```



## Object 对象操作

ObjectUtils 使用

```java
/**
 * 测试使用 ObjectUtils
 *
 * @author Jion
 */
@Slf4j
public class ObjectUtilsTest {

    /**
     * 获取对象的基本信息
     */
    @Test
    public void testObjectA() {

        CoreUtilsApplication application = new CoreUtilsApplication();

        // 获取对象的类名 参数为 null 时，返回字符串 "null" 
        log.info("获取对象类名: {}", ObjectUtils.nullSafeClassName(application));

        // 获取对象 HashCode(十进制) 参数为 null 时，返回 0
        log.info("获取对象哈希码: {}", ObjectUtils.nullSafeHashCode(application));

        // 获取对象 HashCode(十六进制) 参数为 null 时，返回 0 
        log.info("获取对象哈希码: {}", ObjectUtils.getIdentityHexString(application));

        // 获取对象的类名和 HashCode 参数为 null 时，返回字符串："" 
        log.info("获取对象地址: {}", ObjectUtils.identityToString(application));

        // 参数为 null 时，返回字符串 "null"
        log.info("获取对象toString结果: {}", ObjectUtils.nullSafeToString(application));

        // 相当于 toString()方法 但参数为 null 时 返回字符串：""
        log.info("获取对象toString结果: {}", ObjectUtils.getDisplayString(application));
    }

    /**
     * 判断工具
     */
    @Test
    public void testObjectB() {
        String[] arrayA = new String[]{"A", "B", "C"};
        // 判断参数对象是否是数组
        log.info("判断对象是否为数组: {}", ObjectUtils.isArray(arrayA));
        // 判断数组是否为空
        log.info("判断数组是否为空: {}", ObjectUtils.isEmpty(arrayA));
        // 判断数组中是否包含指定元素
        log.info("判断数组中是否包含指定元素: {}", ObjectUtils.containsElement(arrayA, "A"));
        // 向参数数组的末尾追加新元素, 并返回一个新数组
        log.info("向参数数组的末尾追加新元素, 并返回一个新数组: {}", Arrays.toString(ObjectUtils.addObjectToArray(arrayA, "D")));

        // 原生基础类型数组 --> 包装类数组
        int[] arrayB = {1, 2, 3};
        log.info("获取原生基础类型数组的包装类数组: {}", Arrays.toString(ObjectUtils.toObjectArray(arrayB)));
    }

    @Test
    public void testObjectC() {
        // 相等，或同为 null时，返回 true
        log.info("判断对象是否相同: {}", ObjectUtils.nullSafeEquals("  ", "")); 
        
        /*
        判断参数对象是否为空，判断标准为：
            Optional: Optional.empty()
               Array: length == 0
        CharSequence: length == 0
          Collection: Collection.isEmpty()
                 Map: Map.isEmpty()
         */
        log.info("判断参数对象是否为空: {}", ObjectUtils.isEmpty(Optional.empty()));
        log.info("判断参数对象是否为空: {}", ObjectUtils.isEmpty(new Arrays[]{}));
        log.info("判断参数对象是否为空: {}", ObjectUtils.isEmpty(Collections.emptyList()));
        log.info("判断参数对象是否为空: {}", ObjectUtils.isEmpty(Collections.emptySet()));
        log.info("判断参数对象是否为空: {}", ObjectUtils.isEmpty(Collections.emptyMap()));
        log.info("判断参数对象是否为空: {}", ObjectUtils.isEmpty(""));
        log.info("判断参数对象是否为空: {}", ObjectUtils.isEmpty(" "));
    }
}
```



## Collection 集合操作

CollectionUtils 使用

```java
/**
 * CollectionUtils 集合框架工具包
 *
 * @author Jion
 */
@Slf4j
public class CollectionUtilsTest {

    /**
     * 集合判断工具
     */
    @Test
    public void testA() {
        Collection<String> list = new ArrayList<>();
        Map<String, String> map = new HashMap<>();

        // 判断 List/Set 是否为空
        log.info("判断 List/Set 是否为空: {}", CollectionUtils.isEmpty(list));

        // 判断 Map 是否为空
        log.info("判断 Map 是否为空: {}", CollectionUtils.isEmpty(map));

        // 判断 List/Set 中是否包含某个对象
        list.add("A");
        log.info("判断 List/Set 中是否包含对象A: {}", CollectionUtils.containsInstance(list, "A"));

        // 以迭代器的方式,判断 List/Set 中是否包含某个对象
        log.info("以迭代器的方式,判断 List/Set 中是否包含某个对象: {}", CollectionUtils.contains(list.iterator(), "A"));

        // 判断 List/Set 是否包含某些对象中的任意一个
        log.info("判断 List/Set 是否包含某些对象中的任意一个: {}", CollectionUtils.containsAny(list, Collections.singletonList("B")));

        // 判断 List/Set 只含有一个元素.. 即 List/Set 中不存在其他重复元素
        list.add("A");
        log.info("判断 List/Set 中不存在重复元素: {}", CollectionUtils.hasUniqueObject(list));
    }

    /**
     * 集合操作工具
     */
    @Test
    public void testB() {
        // 将 Array 中的元素都添加到 List/Set 中
        String[] array = new String[]{"A", "B", "C"};
        List<String> list = new ArrayList<>();
        CollectionUtils.mergeArrayIntoCollection(array, list);
        log.info("将 Array 中的元素都添加到 List/Set 中: {}", list);

        // 返回 List 中最后一个元素
        log.info("返回 List 中最后一个元素: {}", CollectionUtils.lastElement(list));

        // 返回 Set 中最后一个元素
        Set<String> set = Collections.singleton("A");
        log.info("返回 Set 中最后一个元素: {}", CollectionUtils.lastElement(set));

        // 返回参数 candidates 中第一个存在于参数 source 中的元素
        List<String> source = new ArrayList<>();
        source.add("A");
        source.add("B");
        source.add("C");
        Set<String> candidates = Collections.singleton("B");
        log.info("从source集合中查询第一个符合candidates集合的元素: {}", CollectionUtils.findFirstMatch(source, candidates));

        // 返回 List/Set 中指定类型的元素
        List<Object> collection = new ArrayList<>();
        collection.add("A");
        collection.add(new Date());
        collection.add(1);
        collection.add(2);
        log.info("返回指定类型,多个返回空: {}", CollectionUtils.findValueOfType(collection, Integer.class));

        // 返回 List/Set 中指定类型的元素.如果第一种类型未找到,则查找第二种类型,以此类推
        log.info("返回指定类型,如果第一种类型未找到, 则查找第二种类型, 多个返回空: {}", CollectionUtils.findValueOfType(collection, new Class[]{LocalDate.class, String.class}));

        // 返回 List/Set 中元素的类型
        log.info("返回 List/Set 中元素的类型: {}", CollectionUtils.findCommonElementType(set));

        // 将 Properties 中的键值对都添加到 Map 中
        Map<String, String> map = new HashMap<>();
        Properties properties = new Properties(System.getProperties());
        CollectionUtils.mergePropertiesIntoMap(properties, map);
        log.info("将 Properties 中的键值对都添加到 Map 中: {}", map);
    }
}
```



## File 文件拷贝

FileCopyUtils 使用

```java
/**
 * FileCopyUtils 文件工具类
 *
 * @author Jion
 */
@Slf4j
public class FileCopyUtilsTest {

    private final File in = new File("W:\\SpringBoot-Study\\core-utils\\src\\main\\resources\\application.properties");
    private final File out = new File("W:\\SpringBoot-Study\\core-utils\\test\\main\\resources\\application.properties");


    /**
     * 输入
     */
    @Test
    public void testA() throws IOException {
        // 从文件中读入到字节数组中
        byte[] bytes = FileCopyUtils.copyToByteArray(in);
        log.info("内容: {}", new String(bytes));

        // 从输入流中读入到字节数组中
        byte[] bytes1 = FileCopyUtils.copyToByteArray(new FileInputStream(in));
        log.info("内容: {}", new String(bytes1));

        // 从输入流中读入到字符串中
        String context = FileCopyUtils.copyToString(new FileReader(in));
        log.info("内容: {}", context);
    }


    /**
     * 输出
     */
    @Test
    public void testB() throws IOException {
        // 从字节数组到文件
        FileCopyUtils.copy("这是内容".getBytes(), new File("输出内容.txt"));

        // 从文件到文件
        FileCopyUtils.copy(in, out);

        // 从字节数组到输出流
        FileCopyUtils.copy("这是内容".getBytes(), System.out);

        // 从输入流到输出流
        FileCopyUtils.copy(new FileInputStream(in), System.out);

        // 从输入流到输出流
        FileCopyUtils.copy(new FileReader(in), new FileWriter(out));

        // 从字符串到输出流
        FileCopyUtils.copy("这是内容", new FileWriter(out));
    }
}
```



## Stream 流操作

StreamUtils 使用

```java
/**
 * StreamUtils 流操作
 *
 * @author Jion
 */
@Slf4j
public class StreamUtilsTest {

    private final File in = new File("W:\\SpringBoot-Study\\core-utils\\src\\main\\resources\\application.properties");

    /**
     * 输入
     */
    @Test
    public void testA() throws IOException {

        // 从输入流中读入到字节数组中
        byte[] bytes = StreamUtils.copyToByteArray(new FileInputStream(in));
        log.info("内容: {}", new String(bytes));

        // 从输入流中读入到字符串中
        String context = StreamUtils.copyToString(new FileInputStream(in), StandardCharsets.UTF_8);
        log.info("内容: {}", context);

        // 舍弃输入流中的内容
        StreamUtils.drain(new FileInputStream(in));
    }

    /**
     * 输出
     */
    @Test
    public void testB() throws IOException {
        // 从字节到输出流
        StreamUtils.copy("这是内容...".getBytes(), System.out);

        // 从字符串到输出流
        StreamUtils.copy("这是内容...", StandardCharsets.UTF_8, System.out);

        // 从输入流到输出流
        StreamUtils.copy(new FileInputStream(in), System.out);

        // 从输入流到输入流,指定范围
        StreamUtils.copyRange(new FileInputStream(in), System.out, 0, 100L);
    }
}
```





## Resource 资源操作

ResourceUtils 使用

```java
/**
 * ResourceUtils 资源获取
 *
 * @author Jion
 */
@Slf4j
public class ResourceUtilsTest {

    /**
     * 从资源路径获取文件
     */
    @Test
    public void testA() throws FileNotFoundException {
        // 判断字符串是否是一个合法的 URL 字符串
        log.info("判断字符串是否是一个合法的URL, {}", ResourceUtils.isUrl("http://www.baidu.com"));
        log.info("判断字符串是否是一个合法的URL, {}", ResourceUtils.isUrl("classpath:application.properties"));

        // 获取 URL
        log.info("获取 URL, {}", ResourceUtils.getURL("classpath:application.properties"));

        // 获取文件, 在 JAR 包内无法正常使用,需要是一个独立的文件
        File file = ResourceUtils.getFile("W:\\SpringBoot-Study\\core-utils\\src\\main\\resources\\application.properties");
        log.info("获取文件: {}", file);
    }

    /**
     * Resource 资源
     */
    @Test
    public void testB() throws IOException {

        // URL 资源 如 file://... http://...
        UrlResource urlResource = new UrlResource("file:///W:/SpringBoot-Study/core-utils/src/main/resources/application.properties");

        // 从资源中获得 URI 对象
        URI url = urlResource.getURI();
        log.info("获得");

        // 从资源中获得 URI 对象
        URL uri = urlResource.getURL();


        // 文件系统资源 D:\...
        FileSystemResource fileSystemResource = new FileSystemResource("W:\\SpringBoot-Study\\core-utils\\src\\main\\resources\\application.properties");

        // 判断资源是否存在
        fileSystemResource.exists();

        // 从资源中获得 File 对象
        File file = fileSystemResource.getFile();


        // 获得资源的 InputStream
        InputStream inputStream = fileSystemResource.getInputStream();

        // 类路径下的资源 classpth:...
        ClassPathResource classPathResource = new ClassPathResource("classpath:application.properties");

        // 获得资源的描述信息
        String description = classPathResource.getDescription();
    }
}
```



## Reflection 反射操作

ReflectionUtils 使用

```java
/**
 * ReflectionUtils 反射工具包
 *
 * @author Jion
 */
@Slf4j
@SpringBootTest
public class ReflectionUtilsTest {

    @Autowired
    HomeService homeService;

    /**
     * 获取方法
     */
    @Test
    public void testA() throws NoSuchMethodException {
        // 在类中查找指定方法
        ReflectionUtils.findMethod(HomeService.class, "hello");

        // 同上  额外提供方法参数类型作查找条件
        ReflectionUtils.findMethod(HomeService.class, "hello", String.class);

        // 获得类中所有方法  包括继承而来的
        ReflectionUtils.getAllDeclaredMethods(HomeService.class);

        // 在类中查找指定构造方法
        ReflectionUtils.accessibleConstructor(HomeServiceImpl.class);
    }

    @Test
    public void testB() {
        Method helloMethod = ReflectionUtils.findMethod(HomeService.class, "hello", String.class);
        assert helloMethod != null;

        // 检查一个方法是否声明抛出指定异常
        log.info("是否声明抛出指定异常: {}", ReflectionUtils.declaresException(helloMethod, Exception.class));

        // 是否是 equals() 方法
        log.info("是否是 equals() 方法: {}", ReflectionUtils.isEqualsMethod(helloMethod));

        // 是否是 hashCode() 方法
        log.info("是否是 hashCode() 方法: {}", ReflectionUtils.isHashCodeMethod(helloMethod));

        // 是否是 toString() 方法
        log.info("是否是 toString() 方法: {}", ReflectionUtils.isToStringMethod(helloMethod));

        // 是否是从 Object 类继承而来的方法
        log.info("是否是 Object 类继承而来的方法 方法: {}", ReflectionUtils.isObjectMethod(helloMethod));
    }

    /**
     * 执行方法
     */
    @Test
    public void testC() throws NoSuchMethodException {
        Constructor<HomeServiceImpl> homeServiceConstructor = ReflectionUtils.accessibleConstructor(HomeServiceImpl.class);
        Method sayMethod = ReflectionUtils.findMethod(HomeService.class, "say");
        assert sayMethod != null;
        Method helloMethod = ReflectionUtils.findMethod(HomeService.class, "hello", String.class);
        assert helloMethod != null;

        // 执行方法
        ReflectionUtils.invokeMethod(sayMethod, homeService);

        // 执行方法,提供方法参数
        ReflectionUtils.invokeMethod(helloMethod, homeService, "姓名");

        // 取消 Java 权限检查。以便后续执行该私有方法
        ReflectionUtils.makeAccessible(helloMethod);

        // 取消 Java 权限检查。以便后续执行私有构造方法
        ReflectionUtils.makeAccessible(homeServiceConstructor);
    }

    /**
     * 获取字段
     */
    @Test
    public void testD() {
        // 在类查找指定属性
        Field pageField = ReflectionUtils.findField(HomeServiceImpl.class, "page");
        log.info("获得属性: {}", pageField);

        // 同上  多提供了属性的类型
        Field pageField2 = ReflectionUtils.findField(HomeServiceImpl.class, "page", String.class);
        assert pageField2 != null;

        // 是否为一个 public static final 属性
        log.info("是否为一个 public static final 属性: {}", ReflectionUtils.isPublicStaticFinal(pageField2));
    }

    /**
     * 设置字段
     */
    @Test
    public void testE() {
        Field pageField = ReflectionUtils.findField(HomeServiceImpl.class, "page");
        assert pageField != null;
        // 修改为非 private 属性
        ReflectionUtils.makeAccessible(pageField);
        // 修改属性值
        ReflectionUtils.setField(pageField, homeService, "page_new.html");
    }
}
```





## Aop 代理

AopUtils 使用

```java
/**
 * AopUtils 代理工具类
 *
 * @author Jion
 */
@Slf4j
@SpringBootTest
public class AopUtilsTest {

    @Autowired
    HomeService homeService;

    /**
     * 判断代理类型
     */
    @Test
    public void testA() {
        // 判断是不是 Spring 代理对象
        log.info("判断是不是 Spring 代理对象: {}", AopUtils.isAopProxy(homeService));
        // 判断是不是 jdk 动态代理对象
        log.info("判断是不是 jdk 动态代理对象: {}", AopUtils.isJdkDynamicProxy(homeService));
        // 判断是不是 CGLIB 代理对象
        log.info("判断是不是 CGLIB 代理对象: {}", AopUtils.isCglibProxy(homeService));
    }

    /**
     * 获取被代理对象的 class
     */
    @Test
    public void testB() {
        // 获取被代理的目标 class
        log.info("获取被代理的目标的类类型: {}", AopUtils.getTargetClass(homeService));
    }
}
```

获得代理对象工具类

```java
/**
 * 代理对象.
 *
 * @author Jion
 */
@Slf4j
public class AopContextTest {

    /**
     * 获取当前对象的代理对象
     */
    @Test
    public void test() {
        Object obj = AopContext.currentProxy();
        log.info("获得代理对象: {}", obj);
    }
}
```