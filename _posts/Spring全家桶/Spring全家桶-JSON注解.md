---
title: Spring全家桶-Jackson注解
abbrlink: 316f1148
date: 2019-05-16 10:44:32
categories:
  - Java
  - Spring
tags: [Java, Spring, JSON]
---

> Spring中的 Jackson 使用

<!--more-->



# 简介

在Spring中内置了Jackson类库,可以完成Bean到JSON字符串的序列化与反序列化.

# 语法

## 常用注解预览

### 常用注解预览

**序列化注解**

`@JsonAnyGetter` Get方法注解,用在 `Map` 类型的属性中,将 `Map` 的 `key` 和 `value` 分别作为JSON的 `key` 和 `value` .
`@JsonGetter`  Get方法注解,指定属性序列化后的JSON的 `key` 
`@JsonPropertyOrder` 类注解,指定序列化后的JSON的属性排序.
`@JsonPropertyOrder(alphabetic = true)` 根据字典顺序进行排列.
`@JsonRawValue` 属性注解,该属性的为JSON字符串,并直接作为序列化后的 `value` .
`@JsonValue` 属性注解,将属性值作为JSON返回
`@JsonRootName` 类注解,序列化后JSON被新的 `key` 包裹.
`@JsonSerialize` 属性注解,该属性在被序列化时,使用的序列化类,该序列化类必须继承自`StdSerializer<T>`类

**反序列化注解**

`@JsonAnySetter`	Set方法注解,注解在 `Map` 类型中,将改属性对应的JSON的 `key` 内嵌套的JSON解析为key-value进行保存.
`@JsonSetter`		Set方法注解,当JSON字符串的 `key` 和属性不同时反序列化时便于对应.
`@JsonAlias`	属性注解,当JSON字符串的 `key` 可能不只一个时,使用该注解标识属性对应JSON的 `key` 的别名等..
`@JsonCreator` 		构造方法注解,配合 `@JsonProperty` 注解,对反序列化时,JSON的 `key` 和实际属性不同的类进行标注.
`@JacksonInject`	属性注解,反序列化值从注入中获取值,而不是通过JSON字符串
`@JsonDeserialize` 属性注解,该属性再被反序列化时,使用的反序列化类.该反序列化类必须继承自 `StdDeserializer<T>` 类.


**通用注解**

`@JsonIgnoreProperties`  类注解,标识忽略的属性列表.
`@JsonIgnoreType` 类注解,标识该类作为属性时,被自动忽略.
`@JsonIgnore` 属性注解,标识忽略的属性
`@JsonInclude` 类注解,标识解析字段的要求,排除具有空值,默认值的属性
`@JsonAutoDetect`  类注解,对于私有属性是否进行解析.默认私有属性可以被解析.
`@JsonProperty` 属性注解,标识该属性在序列化和反序列化时的JSON的 `key` .
`@JsonFormat`	属性注解,对于时间属性,序列化时指明序列格式,简便开发,避免编写转化类.
`@JsonUnwrapped` 属性注解,改类型如果是对象,则该对象的属性序列或者反序列时不作为JSON的嵌套
`@JsonView`	 属性注解,仅将注解中类的指定类型进行序列化.
`@JsonManagedReference` 属性注解,标注双向关联的一对多的关系中,一的一方.
`@JsonBackReference`  属性注解,标注双向关联的一对多的关系中,多的一方.
`@JsonFilter` 类注解,指明序列化时,需要的过滤器.

**多态注解**

`@JsonTypeInfo` 类注解, 表明序列化中包含的类型信息的详细信息,注解在父类中
`@JsonSubTypes` 类注解,表明带注释的类型的子类型.注解在父类中,指明拥有哪些子类.
`@JsonTypeName` 类注解,表明子类的类型名称

**不常用的注解**

`@JacksonAnnotationsInside` 自定义注解时,配合原有的注解进行增加.
`@JsonIdentityReference`  类注解,标识为对象的引用,序列化时不会被完整转为JSON字符.
`@JsonIdentityInfo`	类注解,标识强制在每个对象中使用对象标识.
`@JsonAppend` 类注解,对于未在类中定义的属性,进行补充,在序列时注入并转为JSON字符串.
`@JsonPropertyDescription` 属性注解,为属性增加注释说明.
`@JsonPOJOBuilder` 类注解,对于序列化和反序列化时完全不同的字段映射,新建类处理.
`@JsonTypeId` 和	`@JsonTypeIdResolver` 处理继承关系之间的引用问题.


## 常用注解示例
项目示例可能用到 `Lomback` .
`@Getter` 和 `@Setter` 注解表示对属性生成 `get` 和 `set` 方法.
`@Slf4j`  注入日志组件.可以使用日志输出.

### 序列化
序列化注解,在java的Bean转为JSON对象时使用.
序列化时,会默认为存在 `get` 方法的属性生成JSON字段.字段命名参考注解 `@JsonNaming` .

#### `@JsonAnyGetter` 

内部类 `Student` 中,有属性 `info` 和 `score` 分别为 `Map` 类型,在转为JSON时,通过`@JsonAnyGetter` 注解进行映射,将 `Map` 内的值转为JSON属性.
其中`@JsonAnyGetter(enabled = false)` 表示将 `Map` 中的属性嵌套作为JSON串子对象进行组合.默认 `enable = true` ,表示将 `Map` 中的属性与Bean对象属性同级别输出.

**代码**
```java

import com.fasterxml.jackson.annotation.JsonAnyGetter;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.util.HashMap;
import java.util.Map;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonAnyGetter 注解使用
 */
@Slf4j
public class JsonAnyGetterTest {

    /** 内部类 */
    public class Student{

        private Map<String,String> info = new HashMap<>();

        // 会将Map中的属性作为同级进行序列化
        @JsonAnyGetter
        public Map<String, String> getInfo() {
            return info;
        }

        private Map<String,String> score = new HashMap<>();

        // 会将Map中的属性作为嵌套进行序列化
        @JsonAnyGetter(enabled = false)
        public Map<String, String> getScore() {
            return score;
        }
    }


    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.getInfo().put("姓名","囧囧");
        student.getInfo().put("学号","001");
        student.getScore().put("语文","100");
        student.getScore().put("数学","99");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

**输出**
```json
{
  "score": {
    "数学": "99",
    "语文": "100"
  },
  "姓名": "囧囧",
  "学号": "001"
}
```

#### `@JsonGetter` 

内部类 `Student` 中,属性 `name` 被 `@JsonGetter` 注解使用,并指定生成的JSON对属性名称.
由于属性 `id` 存在 `get` 方法.默认也会在JSON对象的属性中出现,其名称与字段名称一致.


**代码**
```java
import com.fasterxml.jackson.annotation.JsonGetter;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonGetter 注解使用
 */
@Slf4j
public class JsonGetterTest {

    /**
     * 内部类
     */
    public class Student {

        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        @JsonGetter("name")
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

**输出**
```json
{
  "id": 1,
  "name": "囧囧"
}

```

#### `@JsonPropertyOrder`

类注解,对生成的JSON属性进行排序,
可以通过字符数组指定序列化后的JSON字段进行排序.
 `@JsonPropertyOrder(alphabetic = true)`  默认将JSON属性的排序根据字典顺序排序.


**代码**
```java
import com.fasterxml.jackson.annotation.JsonGetter;
import com.fasterxml.jackson.annotation.JsonPropertyOrder;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonPropertyOrder 注解使用
 */
@Slf4j
@JsonPropertyOrder({"id","name"})
public class JsonPropertyOrderTest {

    /**
     * 内部类
     */
    public class Student {

        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

**输出**
```json
{
  "id": 1,
  "name": "囧囧"
}
```


#### `@JsonRawValue`
属性或者 `get` 方法注解,将该属性值直接作为JSON的属性值进行序列化.
要求bean对应的属性值,格式为JSON字符串.

**代码**
```java
import com.fasterxml.jackson.annotation.JsonGetter;
import com.fasterxml.jackson.annotation.JsonRawValue;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonRawValue 注解使用
 */
@Slf4j
public class JsonRawValueTest {

    /**
     * 内部类
     */
    public class Student {


        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        private String info;

        @JsonRawValue
        public String getInfo() {
            return info;
        }
        public void setInfo(String info) {
            this.info = info;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setName("囧囧");
        // JSON字符串
        student.setInfo("{\"地址\":\"上海\",\"年龄\":\"23\"}");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

**输出**

```json
{
  "name": "囧囧",
  "info": {
    "地址": "上海",
    "年龄": "23"
  }
}
```


#### `@JsonValue`
仅被 `@JsonValue` 注解标识的该类的属性值会被直接作为序列化后的结果.
如果多个属性被 `@JsonValue` 注解,则会抛出异常.

**代码**
```java
import com.fasterxml.jackson.annotation.JsonValue;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonGetter 注解使用
 */
@Slf4j
public class JsonValueTest {

    /**
     * 内部类
     */
    public class Student {

        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        @JsonValue
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

**输出**
```json
"囧囧"
```


#### `@JsonRootName`
类注解,标识该类属性对应生成的JSON的根节点.当JSON对象被其他嵌套时,对应的节点名称.
 `SerializationFeature.WRAP_ROOT_VALUE`  表示将序列化结果进行包裹

**代码**
```java
import com.fasterxml.jackson.annotation.JsonRootName;
import com.fasterxml.jackson.annotation.JsonValue;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonRootName 注解使用
 */
@Slf4j
public class JsonRootNameTest {

    /**
     * 内部类
     */
    @JsonRootName(value = "student")
    public class Student {

        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");

        String result = new ObjectMapper()
                .enable(SerializationFeature.WRAP_ROOT_VALUE)
                .writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```


**输出**
```json
{
  "student": {
    "id": 1,
    "name": "囧囧"
  }
}
```


#### `@JsonSerialize` 
对于日期格式的,在序列化时需要指定序列化后的格式.就用 `@JsonSerialize` 指定自定义的类.
如日期进行格式化.默认的日期序列化后为时间戳


**代码**
```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.databind.ser.std.StdSerializer;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonSerialize 注解使用
 */
@Slf4j
public class JsonSerializeTest {

    /**
     * 内部类
     */
    public class Student {

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        private Date birthday;

        public void setBirthday(Date birthday) {
            this.birthday = birthday;
        }

        @JsonSerialize(using = JsonDataFormat.class)
        public Date getBirthday() {
            return birthday;
        }
    }

    /** 默认无参构造 */
    public JsonSerializeTest(){
        super();
    }


    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setName("囧囧");
        student.setBirthday(new Date());

        String result = new ObjectMapper()
                .writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}

/**
 * 格式化类,不能使用内部类.因此采用共有类
 */
class JsonDataFormat extends StdSerializer<Date> {

    /** 必须有无参数的构造方法 */
    public JsonDataFormat(){
        this(null);
    }

    /** 重写构造方法 */
    public JsonDataFormat(Class<Date> t){
        super(t);
    }

    @Override
    public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        // 日期格式化
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        // 格式结果
        jsonGenerator.writeString(simpleDateFormat.format(date));
    }
}
```

**输出**
```json
{
  "name": "囧囧",
  "birthday": "2019-05-17 14:17:25"
}
```

### 反序列化
反序列化是将JSON字符串转为Java的Bean对象.
反序列化时,会默认为存在 `set` 方法的属性生成JSON字段.字段命名参考注解 `@JsonNaming` .

#### `@JsonAnySetter`

`@JsonAnySetter` 将当前类中,未被其他属性作反序列化的JSON字符串中的属性,存入该 `Map` 中.
`@JsonAnySetter(enabled = false)`  将JSON字符串中 `score` 子串中的属性及值转为 `Map` 的key-value.

```java
import com.fasterxml.jackson.annotation.JsonAnySetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import static org.junit.Assert.*;

/**
 * @author Jion
 * \@JsonAnySetter 类的测试
 */
@Slf4j
public class JsonAnySetterTest {

    public static class Student {

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        private Map<String,String> info = new HashMap<>();

        public Map<String, String> getInfo() {
            return info;
        }

        public void setInfo(Map<String, String> info) {
            this.info = info;
        }

        // 会将Map中的属性作为同级进行反序列化
        @JsonAnySetter
        public void addInfo(String key, String value) {
            info.put(key, value);
        }

        private Map<String,String> score = new HashMap<>();


        public void setScore(Map<String, String> score) {
            this.score = score;
        }

        // 会将Map中的属性作为嵌套进行反序列化
        @JsonAnySetter(enabled = false)
        public void addScore(String key, String value) {
            score.put(key, value);
        }

        public Map<String, String> getScore() {
            return score;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "name='" + name + '\'' +
                    ", info=" + info +
                    ", score=" + score +
                    '}';
        }
    }

    @Test
    public void test() throws IOException {
        String json = "{\n" +
                "  \"name\": \"囧囧\",\n" +
                "  \"address\":\"上海\",\n" +
                "  \"age\": \"23\",\n" +
                "  \"score\": {\n" +
                "    \"语文\":\"100\",\n" +
                "    \"数学\": \"99\"\n" +
                "  }\n" +
                "}";

        Student student = new ObjectMapper()
                .readerFor(Student.class)
                .readValue(json);

        assertNotNull(student);
        log.info(student.toString());
    }
}
```

#### `@JsonSetter`
将JSON字符串中的属性根据其 `key` 的名字映射为Java类的属性.
对于属性名称不一致时,通过 `@JsonSetter` 指定名称.
另外,当名称一致时,不需要指定名称时也可以进行反序列化.

```java

import com.fasterxml.jackson.annotation.JsonSetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.io.IOException;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonSetter 类的测试
 */
@Slf4j
public class JsonSetterTest {

    public static class Student {


        private Integer id;

        @JsonSetter("name")
        private String name;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    '}';
        }
    }

    @Test
    public void test() throws IOException {
        String json = "{\n" +
                "  \"id\":\"1\",\n" +
                "  \"name\": \"囧囧\"\n" +
                "}";

        Student student = new ObjectMapper()
                .readerFor(Student.class)
                .readValue(json);

        assertNotNull(student);
        log.info(student.toString());
    }
}
```

#### `@JsonAlias`
当JSON字符串中的属性具有不唯一的 `key` 时,即存在别名,可以通过 `@JsonAlias` 指定JSON的别名.

JSON字符串中 `username` 和 `name` 属性对应Bean的 `name` 属性,此外,当JSON中出现多个别名对应的同一个Bean属性,则以最后一次出现的 `key` 的值为最终结果.

```java
import com.fasterxml.jackson.annotation.JsonAlias;
import com.fasterxml.jackson.annotation.JsonSetter;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.io.IOException;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonAlias 类的测试
 */
@Slf4j
public class JsonAliasTest {

    public static class Student {


        private Integer id;

        @JsonAlias({"name","username"})
        private String name;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    '}';
        }
    }

    @Test
    public void test() throws IOException {
        String json = "{\n" +
                "  \"id\":\"1\",\n" +
                "  \"name\": \"囧囧1\",\n" +
                "  \"username\": \"囧囧2\"\n" +
                "}";

        Student student = new ObjectMapper()
                .readerFor(Student.class)
                .readValue(json);

        assertNotNull(student);
        log.info(student.toString());
    }
}
```



#### `@JsonCreator`
在构造函数中使用,通过 `@JsonProperty` 注解配合反序列化


```java
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.io.IOException;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonCreator 类的测试
 */
@Slf4j
public class JsonCreatorTest {

    public static class Student {

        private Integer id;

        private String name;

        @JsonCreator
        public Student(@JsonProperty("id") Integer id, @JsonProperty("name") String name) {
            this.id = id;
            this.name = name;
        }


        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    '}';
        }
    }

    @Test
    public void test() throws IOException {
        String json = "{\n" +
                "  \"id\":\"1\",\n" +
                "  \"name\": \"囧囧\"\n" +
                "}";

        Student student = new ObjectMapper()
                .readerFor(Student.class)
                .readValue(json);

        assertNotNull(student);
        log.info(student.toString());
    }
}
```



#### `@JacksonInject`
被 `@JacksonInject` 注解过的属性,需要在反序列化时,为其指定值.通过 `InjectableValues` 类进行复制.

```java
import com.fasterxml.jackson.annotation.JacksonInject;
import com.fasterxml.jackson.databind.InjectableValues;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.io.IOException;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JacksonInject 类的测试
 */
@Slf4j
public class JacksonInjectTest {

    public static class Student {

        @JacksonInject
        private Integer id;

        private String name;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    '}';
        }
    }

    @Test
    public void test() throws IOException {
        String json = "{\n" +
                "  \"name\": \"囧囧\"\n" +
                "}";
        // 提供的绑定变量
        InjectableValues inject = new InjectableValues.Std().addValue(Integer.class, 1);
        // 变量赋值,并指定读取的类
        Student student = new ObjectMapper()
                .reader(inject)
                .forType(Student.class)
                .readValue(json);

        assertNotNull(student);
        log.info(student.toString());
    }
}
```

#### `@JsonDeserialize`
对于日期类型的JSON属性反序列化时,通过 `@JsonDeserialize` 注解指定反序列化时的类,将字符格式转为日期格式.
格式化类继承自 `StdDeserializer` 类,并实现 `deserialize` 方法.注意声明无参/有参的构造方法

```java

import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.deser.std.StdDeserializer;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.io.IOException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonDeserialize 注解使用
 */
@Slf4j
public class JsonDeserializeTest {

    /**
     * 内部类
     */
    public static class Student {

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        private Date birthday;

        public void setBirthday(Date birthday) {
            this.birthday = birthday;
        }

        @JsonDeserialize(using = BeanDataFormat.class)
        public Date getBirthday() {
            return birthday;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "name='" + name + '\'' +
                    ", birthday=" + birthday +
                    '}';
        }
    }

    /** 默认无参构造 */
    public JsonDeserializeTest(){
        super();
    }


    @Test
    public void test() throws IOException {

        String json = "{\n" +
                "  \"name\": \"囧囧\",\n" +
                "  \"birthday\": \"2019-01-01 12:00:00\"\n" +
                "}";

        Student student = new ObjectMapper()
                .readerFor(Student.class)
                .readValue(json);

        assertNotNull(student);
        log.info(student.toString());
    }
}

/**
 * 格式化类,不能使用内部类.因此采用共有类
 */
class BeanDataFormat extends StdDeserializer<Date> {

    /** 必须有无参数的构造方法 */
    public BeanDataFormat(){
        this(null);
    }

    @Override
    public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {

        // 日期格式化
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        // 获得JSON属性值
        String date = jsonParser.getText();
        // 格式结果
        try {
            return simpleDateFormat.parse(date);
        } catch (ParseException e) {
            e.printStackTrace();
            return new Date();
        }
    }

    /** 重写构造方法 */
    public BeanDataFormat(Class<Date> t){
        super(t);
    }
}
```


### 通用注解

#### `@JsonIgnoreProperties`
类注解,被标注的属性会被忽略.

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonIgnoreProperties 注解使用
 */
@Slf4j
public class JsonIgnorePropertiesTest {

    /**
     * 内部类
     */
    @JsonIgnoreProperties(value = {"id","address"})
    public class Student {

        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        private String address;

        public String getAddress() {
            return address;
        }

        public void setAddress(String address) {
            this.address = address;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");
        student.setAddress("上海");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```


#### `@JsonIgnoreType`
类注解,标识该类作为属性时,被自动忽略.

内部类 `Info` 在被注解 `@JsonIgnoreType` 注解时,在被作为属性时,会自动被忽略.

```java

import com.fasterxml.jackson.annotation.JsonIgnoreType;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonIgnoreType 注解使用
 */
@Slf4j
public class JsonIgnoreTypeTest {

    /**
     * 内部类
     */
    public class Student {

        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        private Info info;

        public Info getInfo() {
            return info;
        }

        public void setInfo(Info info) {
            this.info = info;
        }
    }

    @JsonIgnoreType
    public class Info{
        String address;

        String postcode;

        public String getAddress() {
            return address;
        }

        public void setAddress(String address) {
            this.address = address;
        }

        public String getPostcode() {
            return postcode;
        }

        public void setPostcode(String postcode) {
            this.postcode = postcode;
        }

        public Info(String address, String postcode) {
            this.address = address;
            this.postcode = postcode;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");
        Info info = new Info("上海","00001");
        student.setInfo(info);

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

#### `@JsonIgnore`
属性注解,标识忽略的属性

``` java

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonIgnore 注解使用
 */
@Slf4j
public class JsonIgnoreTest {

    /**
     * 内部类
     */
    public class Student {

        @JsonIgnore
        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        private String address;

        public String getAddress() {
            return address;
        }

        public void setAddress(String address) {
            this.address = address;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");
        student.setAddress("上海");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

#### `@JsonInclude`
类注解,标识解析字段的要求,例如排除具有空值,默认值的属性

`JsonInclude.Include.ALWAYS` 默认策略, 总数序列化
`JsonInclude.Include.NON_NULL` 当属性为 `null` 时, 不进行序列化
`JsonJsonInclude.Include.NON_ABSENT` 当属性为 `null` 或 `Optional.empty` 为空时, 不进行序列化
`JsonJsonInclude.Include.NON_DEFAULT` 当属性为默认值时, 不进行序列化
`JsonJsonInclude.Include.NON_EMPTY` 属性为 `null` 或 `Optional.empty` 为空, 集合数组为空, 字符串空, 基本类型默认值时, 不进行序列化
`JsonJsonInclude.Include.CUSTOM` 使用过滤规则, 进行匹配
`JsonJsonInclude.Include.USE_DEFAULTS` 使用默认级别进行处理, 根据属性访问权限而定

```java

import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonInclude 注解使用
 */
@Slf4j
public class JsonIncludeTest {

    /**
     * 内部类
     */
    @JsonInclude(JsonInclude.Include.NON_NULL)
    public class Student {

        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        private String address;

        public String getAddress() {
            return address;
        }

        public void setAddress(String address) {
            this.address = address;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");
        student.setAddress(null);

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

#### `@JsonAutoDetect`
类注解,对不同的修饰符的类属性进行控制,是否可以自动序列化,反序列,可见性控制等.


**对私有属性均可,序列化/反序列化 **
```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonAutoDetect 注解使用
 */
@Slf4j
public class JsonAutoDetectTest {

    /**
     * 内部类
     */
    @JsonAutoDetect(fieldVisibility = JsonAutoDetect.Visibility.ANY)
    public class Student {

        private Integer id;
        private String name;

        public Student(Integer id, String name) {
            this.id = id;
            this.name = name;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student(1,"囧囧");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

#### `@JsonProperty`
属性/方法注解,标识该属性在序列化和反序列化时的JSON的 `key` .
通过分别标注在 `Getter/Setter`  方法上,指定不同方法序列化/反序列化时的 `key`

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonProperty 注解使用
 */
@Slf4j
public class JsonPropertyTest {

    /**
     * 内部类
     */
    public class Student {

        @JsonProperty("id")
        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        @JsonProperty("name")
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

#### `@JsonFormat`
属性注解,对于时间属性,序列化时指明序列格式,简便开发,避免编写转化类.

`@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss",timezone="GMT+8")` 日期格式 
```java
import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.util.Date;
import java.util.TimeZone;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonFormat 注解使用
 */
@Slf4j
public class JsonFormatTest {

    /**
     * 内部类
     */
    public class Student {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss",timezone="GMT+8")
        private Date birthday;

        public Date getBirthday() {
            return birthday;
        }

        public void setBirthday(Date birthday) {
            this.birthday = birthday;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setName("囧囧");
        student.setBirthday(new Date());

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

#### `@JsonUnwrapped`
属性注解,改类型如果是对象,则该对象的属性序列或者反序列时不作为JSON的嵌套

`@JsonUnwrapped` 注解标注的对象属性,在序列化时,不会被作为嵌套解析,而是与当前对象属性同级进行序列化输出
```java
import com.fasterxml.jackson.annotation.JsonUnwrapped;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonUnwrapped 注解使用
 */
@Slf4j
public class JsonUnwrappedTest {

    /**
     * 内部类
     */
    public class Student {

        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @JsonUnwrapped
        private Info info;

        public Info getInfo() {
            return info;
        }

        public void setInfo(Info info) {
            this.info = info;
        }
    }

    public class Info{
        String address;

        String postcode;

        public String getAddress() {
            return address;
        }

        public void setAddress(String address) {
            this.address = address;
        }

        public String getPostcode() {
            return postcode;
        }

        public void setPostcode(String postcode) {
            this.postcode = postcode;
        }

        public Info(String address, String postcode) {
            this.address = address;
            this.postcode = postcode;
        }
    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");
        Info info = new Info("上海","00001");
        student.setInfo(info);

        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}
```

**输出**
```json
{
  "id": 1,
  "name": "囧囧",
  "address": "上海",
  "postcode": "00001"
}
```

#### `@JsonView`
属性注解,仅将注解中类的指定类型进行序列化.

类 `Views` 含有内部类 `show` 和 `hide` .在Java的Bean中,通过注解 `@JsonView()` 规定属性的归属管理类,指向内部类.
在序列化时,通过 `writerWithView()` 方法,指向规定显示的类,则拥有该类 `@JsonView` 注解的属性将会被序列化.
```java
import com.fasterxml.jackson.annotation.JsonView;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonView 注解使用
 */
@Slf4j
public class JsonViewTest {

    /**
     * 内部类
     */
    public class Student {

        @JsonView(Views.hide.class)
        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        @JsonView(Views.show.class)
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");

        String result = new ObjectMapper()
                .writerWithView(Views.show.class)
                .writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}

/** 视图类,是否显示隐藏 */
class Views{
    public class show{

    }

    public class hide{

    }
}
```

#### `@JsonManagedReference` / `@JsonBackReference`

`@JsonManagedReference` 属性注解,标注双向关联的一对多的关系中,一的一方.
`@JsonBackReference` 属性注解,标注双向关联的一对多的关系中,多的一方.

在Java中,如果出现对象之间相互依赖,在序列化时会出现循环指向,最终导致堆栈溢出.
因此,可以使用 `@JsonManagedReference`  注解标注在一的一方的属性中,如类中的 `Student` 属性
而 `@JsonBackReference`  注解标注在多的一方的属性中,如类中的 `List<Score>` 属性.
```java
import com.fasterxml.jackson.annotation.JsonBackReference;
        import com.fasterxml.jackson.annotation.JsonManagedReference;
        import com.fasterxml.jackson.core.JsonProcessingException;
        import com.fasterxml.jackson.databind.ObjectMapper;
        import lombok.extern.slf4j.Slf4j;
        import org.junit.Test;

        import java.util.ArrayList;
        import java.util.List;

        import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonManagedReference 和 \@sonBackReferenceTest 注解使用
 */
@Slf4j
public class JsonManagedReferenceAndJsonBackReferenceTest {


    @Test
    public void test() throws JsonProcessingException {
        // 一方
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");
        // 多方
        Score chinese = new Score();
        chinese.setSubject("语文");
        chinese.setResult("100");
        chinese.setStudent(student);
        Score math = new Score();
        math.setSubject("数学");
        math.setResult("99");
        math.setStudent(student);
        // 一方去维护多方
        List<Score> scores = new ArrayList<>();
        scores.add(chinese);
        scores.add(math);
        student.setScores(scores);

        // 输出
        String result = new ObjectMapper().writeValueAsString(student);
        assertNotNull(result);
        log.info(result);
    }
}


/** 一方,学生 */
class Student {

    private  Integer id;

    private String name;

    @JsonBackReference
    public List<Score> scores;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Score> getScores() {
        return scores;
    }

    public void setScores(List<Score> scores) {
        this.scores = scores;
    }
}



/** 多方,成绩方 */
class Score {

    private String subject;

    private String result;

    @JsonManagedReference
    private Student student;

    public String getSubject() {
        return subject;
    }

    public void setSubject(String subject) {
        this.subject = subject;
    }

    public String getResult() {
        return result;
    }

    public void setResult(String result) {
        this.result = result;
    }

    public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }
}
```

#### `@JsonFilter`
类注解,指明序列化时,需要的过滤器.

通过 `@JsonFilter("jsonFilter")` 指定过滤器 `jsonFilter` ,在序列化时,创建过滤器类.并指定可以序列化的属性,完成序列化.
```java
import com.fasterxml.jackson.annotation.JsonFilter;
import com.fasterxml.jackson.annotation.JsonView;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.ser.FilterProvider;
import com.fasterxml.jackson.databind.ser.impl.SimpleBeanPropertyFilter;
import com.fasterxml.jackson.databind.ser.impl.SimpleFilterProvider;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import static org.junit.Assert.assertNotNull;

/**
 * @author Jion
 * \@JsonFilter 注解使用
 */
@Slf4j
public class JsonFilterTest {

    /**
     * 内部类
     */
    @JsonFilter("jsonFilter")
    public class Student {

        @JsonView(Views.hide.class)
        private Integer id;

        public Integer getId() {
            return id;
        }

        public void setId(Integer id) {
            this.id = id;
        }

        @JsonView(Views.show.class)
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

    }

    @Test
    public void test() throws JsonProcessingException {
        Student student = new Student();
        student.setId(1);
        student.setName("囧囧");

        // 过滤器
        FilterProvider filters = new SimpleFilterProvider()
                .addFilter("jsonFilter", // 过滤器名称
                        SimpleBeanPropertyFilter.filterOutAllExcept("name") // 仅允许输出的类
                );

        // 输出
        String result = new ObjectMapper()
                .writer(filters)
                .writeValueAsString(student);

        assertNotNull(result);
        log.info(result);
    }
}
```

### 多态注解
不做深入了解 

### 不常用的注解
不做深入了解 



# 示例

## 自定义序列化与反序列化

属性

```java
@Column(name = "START_DATE")
@JsonSerialize(using = DateToLongSerializer.class)
@JsonDeserialize(using = LongToDateSerializer.class)
private LocalDateTime startDate;
```

时间戳转为日期

```java
public class LongToDateSerializer extends JsonDeserializer<LocalDateTime> {
    @Override
    public LocalDateTime deserialize(JsonParser parser, DeserializationContext ctxt) throws IOException, JsonProcessingException {
        Instant instant = Instant.ofEpochMilli(Long.parseLong(parser.getText()));
        return LocalDateTime.ofInstant(instant, ZoneOffset.ofHours(+8));
    }
}
```

日期转为时间戳

```java
public class DateToLongSerializer extends JsonSerializer<LocalDateTime> {

    @Override
    public void serialize(LocalDateTime localDateTime, JsonGenerator gen, SerializerProvider serializers) throws IOException {
        // 时间戳,时区为本地时间
        gen.writeNumber(localDateTime.toInstant(ZoneOffset.ofHours(+8)).toEpochMilli());
    }
}
```

## 为值集属性设置

期望的属性,通过查询值集列表, 将 `Y` 翻译为 `是` 并, 动态添加属性 `valueDesc` ,完成

```json
{"name":"Jion","value":"Y","valueDesc":"是"}
```

自定义注解

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Lookup {

    /**
     * 值集Code
     *
     * @return 值集Code
     */
    String code() default "";
}
```

具体的序列化类

```java
@Slf4j
public class LookupCodeJsonSerializer extends StdSerializer<String> implements ContextualSerializer {

    private String code;

    private String fileName;

    private final static String DESC = "Desc";

    /**
     * 无参构造器,必须存在,仅支持 String 类型的转换
     */
    public LookupCodeJsonSerializer() {
        super(String.class);
    }

    /**
     * 构造器, 传入值集 Code 参数
     *
     * @param code 值集 Code 代码
     */
    private LookupCodeJsonSerializer(String code, String fileName) {
        super(String.class);
        this.code = code;
        this.fileName = fileName;
    }

    /**
     * 序列化方法
     *
     * @param jsonValue     值
     * @param jsonGenerator 生成JSON工具类
     * @param serializers   序列化器
     * @throws IOException 序列化时异常
     */
    @Override
    public void serialize(String jsonValue, JsonGenerator jsonGenerator, SerializerProvider serializers) throws IOException {
        LookupCodeOperations lookupCodeOperations = ConsoleBeanUtils.getBean(LookupCodeOperations.class);
        String codeName = lookupCodeOperations.get(code, jsonValue);
        if(StringUtils.isBlank(codeName)){
            log.warn("in redis cache can not found this code: {}, the fileName is {} , and the json value is {}",
                    this.code, this.fileName, jsonValue);
        }
        // 首先写入值
        jsonGenerator.writeString(jsonValue);
        // 再织入新的值
        jsonGenerator.writeStringField((this.fileName + DESC), codeName);

    }

    /**
     * 选择具体的序列化方法, 优先执行
     *
     * @param serializerProvider 可选序列化器
     * @param beanProperty       Bean配置
     * @return 自定义序列化类
     */
    @Override
    public JsonSerializer<String> createContextual(SerializerProvider serializerProvider, BeanProperty beanProperty) {
        if (beanProperty != null) {
            // Bean 属性, 获得其上注解
            Lookup annotation = beanProperty.getAnnotation(Lookup.class);
            if (annotation != null) {
                // 注解上的value值, 获得值集 Code 信息
                this.code = annotation.code();
                // 序列化对象属性名
                String fileName = beanProperty.getName();
                // 返回对应的序列化器
                return new LookupCodeJsonSerializer(code, fileName);
            }
        }
        return null;
    }
}
```





测试类

```java
@Slf4j
public class LookupCodeJsonSerializerTest extends ConsoleApplicationTest {

    @Test
    public void test() throws IOException {
        LookupCodeEntity entity = new LookupCodeEntity();
        entity.setName("Jion");
        entity.setValue("Y");

        // 序列化
        ObjectMapper mapper = new ObjectMapper();
        String asString = mapper.writeValueAsString(entity);
        log.info(asString);
    }
}

/**
 *  测试类
 */
@Data
class LookupCodeEntity{
    private String name;
    
    @Lookup(code = "YES_NO")
    @JsonSerialize(using = LookupCodeJsonSerializer.class)
    private String value;
}
```

