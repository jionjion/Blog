---
title: Java基础-Lombok语法
abbrlink: b6309923
date: 2019-06-05 11:28:33
categories:
  - Java
  - Lombok
tags: [Java, Lombok]
---


## 简介

简要介绍 `Lombok`

## 常用注解

| 注解 | 说明 |
| ---- | ---- |
|`@NonNull`  |    检查参数是否为空,为`null`抛出异常 |
|`@Cleanup`  |    对于`IO`流等需要关闭的,可以通过该注解自动关闭 |
|`@Getter`   |    对属性生成`Getter`方法,布尔值为`Is`方法.通过`@Getter(lazy=true)`进行懒加载 |
|`@Setter`   |    对非`final`属性生成`Setter`方法 |
|`@ToString` |    对类生成`toString`方法 |
|`@EqualsAndHashCode` |   重写`haseCode()`和`equal`方法 |
|`@NoArgsConstructor`,`@RequiredArgsConstructor`, `@AllArgsConstructor`| 定义构造函数 |
|`@Data`     |    简写注解,可变的Bean对象 |
|`@Value`    |    简写注解,不可变的Bean的对象 |
|`@SneakyThrows`| 将编译时异常改为运行时异常 |
|`@Synchronized`| 方法锁 |
|`@Log`      |    日志组件,实例化 `java.util.logging.Logger` 类 |
|`@CommonsLog`|   日志组件,实例化 `org.apache.commons.logging.Log` 类 |
|`@Slf4j`     |   日志组件,实例化 `org.slf4j.Logger` 类 |
|`@Flogger`   |   日志组件,实例化 `com.google.common.flogger.FluentLogger` 类 |
|`@JBossLog`  |   日志组件,实例化 `org.jboss.logging.Logger` 类 |
|`@Log4j`     |   日志组件,实例化 `org.apache.log4j.Logger` 类 |
|`@Log4j2`    |   日志组件,实例化 `org.apache.logging.log4j.Logger` 类 |
|`@XSlf4j`    |   日志组件,实例化 `org.slf4j.ext.XLogger` 类 |


## 注解示例

### `@NonNull`
检查方法中传参是否为`null`,如果是则抛出异常.
标注在方法参数的前面,用于检测.
```java
import lombok.NonNull;
import org.junit.Test;

/**
 * @author Jion
 *  \@NonNull 注解使用,用于检查变量是否为空
 */
public class NonNullAnnotateExample {


    /** 参数 name 不能为空 */
    public String sayHello(@NonNull String name){
        return ("Hello " + name.toUpperCase());
    }

    @Test
    public void testSayHello(){
        System.out.println( sayHello(null) );
    }

}
```



### `@Cleanup`

对于`IO`流等需要关闭的,可以通过该注解自动关闭
`value`属性值指调用的关闭方法名.
**关闭IO流**
```java
import lombok.Cleanup;
import org.junit.Test;

import java.io.*;

/**
 * @author Jion
 *  \@close 注解,用于安全释放连接等.默认调用value中的方法
 */
public class CleanupAnnotateExample {

    public String readBook(String src) throws IOException {

        @Cleanup
        InputStream in = new FileInputStream(new File(src));
        @Cleanup(value = "close")
        InputStreamReader reader = new InputStreamReader(in);

        StringBuilder stringBuilder = new StringBuilder();
        char[] c = new char[10000];

        while (true) {
            int r = reader.read(c);
            if (r == -1) {
                break;
            }
            stringBuilder.append(new String(c));
        }
        return stringBuilder.toString();
    }

    @Test
    public void testReadBook() throws IOException {
        String result = readBook("F:\\JAVA_WorkSpace\\JavaBase\\src\\lomback\\README.md");
        System.out.println(result);
    }
}
```



### `@Getter`

对属性生成`Getter`方法,布尔值为`Is`方法.
`Value`默认为生成方法的访问修饰符.由枚举类`lombok.AccessLevel`中的成员限制.
可选的成员修饰有 `PUBLIC`, `MODULE`, `PROTECTED`, `PACKAGE`, `PRIVATE`, `NONE`

另外,可以通过`@Getter(lazy=true)`可以进行懒加载,创建简单的单例模式

```java

import lombok.AccessLevel;
import lombok.Getter;
import org.junit.Assert;
import org.junit.Test;

/**
 * @author Jion
 *  \@Getter 注解使用
 *  可以选择访问控制等级,默认的是共有方法. PUBLIC, MODULE, PROTECTED, PACKAGE, PRIVATE, NONE
 *  可以选择是否懒加载,默认false,如果懒加载需要为属性在创建时赋初值
 */
public class GetterAnnotateExample {

    @Getter(AccessLevel.PUBLIC)
    private String name = "Jion";

    @Getter(AccessLevel.NONE)
    private String address = "ShangHai";

    @Getter(lazy = true)
    private final String post = "000A";

    /** 注意boolean类型的为is */
    @Getter
    private boolean boy = false;

    public GetterAnnotateExample(){
        super();
    }

    @Test
    public void testGetter(){
        GetterAnnotateExample example = new GetterAnnotateExample();
        Assert.assertNotNull(example.getName());
        System.out.println(example.getName());
        // Assert.assertNotNull(example.getAddress());
        Assert.assertNotNull(example.getPost());
        System.out.println(example.getPost());
        System.out.println(example.isBoy());
    }
}
```

### `@Setter`
对非`final`修饰的属性生成`Setter`方法
`Value`默认为生成方法的访问修饰符.由枚举类`lombok.AccessLevel`中的成员限制.
可选的成员修饰有 `PUBLIC`, `MODULE`, `PROTECTED`, `PACKAGE`, `PRIVATE`, `NONE`

```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;
import org.junit.Assert;
import org.junit.Test;

/**
 * @author Jion
 *  \@Setter 注解使用,为非final修饰的生成setter()
 *  可以选择访问控制等级,默认的是共有方法. PUBLIC, MODULE, PROTECTED, PACKAGE, PRIVATE, NONE
 *  可以选择是否懒加载,默认false,如果懒加载需要为属性在创建时赋初值
 */
public class SetterAnnotateExample {

    @Setter(AccessLevel.PUBLIC)
    private String name;

    @Setter(AccessLevel.NONE)
    private String address;

    @Setter
    private String post;

    @Setter
    private boolean boy;

    public SetterAnnotateExample(){
        super();
    }

    @Override
    public String toString() {
        return "SetterAnnotateExample{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", post='" + post + '\'' +
                ", boy=" + boy +
                '}';
    }

    @Test
    public void testGetter(){
        SetterAnnotateExample example = new SetterAnnotateExample();
        example.setName("Jion");
        // Assert.assertNotNull(example.setAddress("ShangHai"));
        example.setPost("000A");
        Assert.assertNotNull(example.toString());
        example.setBoy(true);
        System.out.println(example.toString());
    }
}
```

### `@ToString`
对类生成`toString`方法,常用注解属性有
`callSuper` 是否将构造方法一同输出.
`includeFieldNames` 将类的属性字段和属性字段值一同输出.
`exclude` 排除类的某些属性,不做输出,当`exclude`和`of`同时使用时,`exclude`会被忽略
`of`      指定类的那些属性,可以做输出
`doNotUseGetters` 当进行toString时,如果为`true`则不使用`getter`方法获得属性值,而是直接访问,默认`false`
`onlyExplicitlyIncluded` 默认为`false`,当为true时,只有被 `@ToString.Include` 标注的属性才会被输出,`@ToString.Exclude` 不会.


```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.ToString;
import org.junit.Assert;
import org.junit.Test;

/**
 * @author Jion
 */
public class ToStringAnnotateExample {


    /** 将构造函数和字段名称全部输出 */
    @ToString(callSuper = true, includeFieldNames = true)
    class UserA {

        public int id;

        public String name;

        public String address;

        public UserA(int id, String name, String address) {
            this.id = id;
            this.name = name;
            this.address = address;
        }
    }

    @Test
    public void testUserA(){
        UserA userA = new UserA(1,"Jion","ShangHai");
        Assert.assertNotNull(userA.toString());
        System.out.println(userA.toString());
    }

    /** 当exclude和of同时使用时,exclude会被忽略
     *  exclude 排除那些属性可以被输出
     *  of      指定那些属性可以被输出,未被包括属性的不输出
     */
    @ToString(exclude = {"id"},of = {"name"})
    class UserB {
        public int id;

        public String name;

        public String address;
    }

    @Test
    public void testUserB(){
        UserB userB = new UserB();
        userB.id = 1;
        userB.name = "Jion";
        userB.address = "ShangHai";
        Assert.assertNotNull(userB.toString());
        System.out.println(userB.toString());
    }

    /** doNotUseGetters 当进行toString时,如果为true则不使用getter方法获得属性值,而是直接访问.
     *                  默认false,即通过getter方法获得属性值,无论get方法是否存在 */
    @ToString(doNotUseGetters = false)
    class UserC {
        private int id;

        private String name;

        private String address;

        public UserC(int id, String name, String address) {
            this.id = id;
            this.name = name;
            this.address = address;
        }
    }

    @Test
    public void testUserC(){
        UserC userC = new UserC(1,"Jion","ShangHai");
        Assert.assertNotNull(userC.toString());
        System.out.println(userC.toString());
    }

    /**
     *  onlyExplicitlyIncluded 默认false.
     *      当为true时,只有被\@ToString.Include 标注的属性才会被输出
     *  \@ToString.Exclude 属性不被输出
     *  \@ToString.Include 属性被输出
     *  */
    @ToString(onlyExplicitlyIncluded = true)
    class UserD{

        @ToString.Exclude
        public int id;

        @ToString.Include
        public String name;

        public String address;

        public UserD(int id, String name, String address) {
            this.id = id;
            this.name = name;
            this.address = address;
        }
    }
    @Test
    public void testUserD(){
        UserD userD = new UserD(1,"Jion","ShangHai");
        Assert.assertNotNull(userD.toString());
        System.out.println(userD.toString());
    }
}
```

### `@EqualsAndHashCode`
类基础的 `haseCode()`和`equal`方法的重写,常用注解属性有
`callSuper` 是否对继承自的父类进行比较.默认为`false`
`exclude`   排除那些属性比较,当`exclude`和`of`同时使用时,`exclude`会被忽略
`of`        指定那些属性比较,未被包括属性的不进行比较
`doNotUseGetters`   当进行toString时,如果为true则不使用getter方法获得属性值,而是直接访问.默认false,即通过getter方法获得属性值,无论get方法是否存在
`onlyExplicitlyIncluded` 默认false.当为true时,只有被 `EqualsAndHashCode.Include` 标注的属性才会被比较 `EqualsAndHashCode.Exclude` 属性不被比较


```java
import lombok.EqualsAndHashCode;
import lombok.ToString;
import lombok.experimental.Helper;
import org.junit.Assert;
import org.junit.Test;

/**
 * @author Jion
 *  \@EqualsAndHashCode 注解
 */
public class EqualsAndHashCodeAnnotateExample {

    /** 父类 */
    class User{

    }
    /** 子类
     *  \@EqualsAndHashCode 默认callSuper=false
     *  当为false时,只比对子类之间的属性值差异,而不比较继承的父类
     *  当为true时,如果继承的父类实例一致,则返回相同的hashcode */
    @EqualsAndHashCode(callSuper = false)
    class UserA extends User{

        public int id;

        public String name;

        public String address;

        public UserA(int id, String name, String address) {
            super();
            this.id = id;
            this.name = name;
            this.address = address;
        }
    }

    @Test
    public void testUserA(){
        UserA userA1 = new UserA(1,"Jion","ShangHai");
        UserA userA2 = new UserA(1,"Jion","ShangHai");
        Assert.assertEquals(userA1.hashCode(), userA2.hashCode());
        System.out.println(userA1.hashCode());
        System.out.println(userA2.hashCode());
    }

    /** 当exclude和of同时使用时,exclude会被忽略
     *  exclude 排除那些属性比较
     *  of      指定那些属性比较,未被包括属性的不进行比较
     *
     */
    @EqualsAndHashCode(exclude = {"id"}, of = {"name"})
    class UserB{
        public int id;

        public String name;

        public String address;

        public UserB(int id, String name, String address) {
            this.id = id;
            this.name = name;
            this.address = address;
        }
    }

    @Test
    public void testUserB(){
        UserB userB1 = new UserB(1,"Jion","HeNan");
        UserB userB2 = new UserB(2,"Jion","ShangHai");
        Assert.assertEquals(userB1.hashCode(), userB2.hashCode());
        System.out.println(userB1.hashCode());
        System.out.println(userB2.hashCode());
    }

    /** doNotUseGetters 当进行toString时,如果为true则不使用getter方法获得属性值,而是直接访问.
     *                  默认false,即通过getter方法获得属性值,无论get方法是否存在 */
    @EqualsAndHashCode(doNotUseGetters = true)
    class UserC{
        private int id;

        private String name;

        private String address;

        public UserC(int id, String name, String address) {
            this.id = id;
            this.name = name;
            this.address = address;
        }
    }

    @Test
    public void testUserC(){
        UserC userC1 = new UserC(1,"Jion","HeNan");
        UserC userC2 = new UserC(2,"Jion","HeNan");
        Assert.assertNotEquals(userC1.hashCode(), userC2.hashCode());
        System.out.println(userC1.toString());
        System.out.println(userC2.toString());
    }

    /**
     *  onlyExplicitlyIncluded 默认false.
     *      当为true时,只有被\@EqualsAndHashCode.Include 标注的属性才会被比较
     *  \@EqualsAndHashCode.Exclude 属性不被比较
     *  \@EqualsAndHashCode.Include 属性被比较
     *  */
    @EqualsAndHashCode(onlyExplicitlyIncluded = true)
    class UserD{
        @EqualsAndHashCode.Exclude
        private int id;

        @EqualsAndHashCode.Include
        private String name;

        private String address;

        public UserD(int id, String name, String address) {
            this.id = id;
            this.name = name;
            this.address = address;
        }
    }

    @Test
    public void testUserD(){
        UserD userD1 = new UserD(1,"Jion","HeNan");
        UserD userD2 = new UserD(2,"Jion","ShangHai");
        Assert.assertEquals(userD1.hashCode(), userD2.hashCode());
        System.out.println(userD1.toString());
        System.out.println(userD2.toString());
    }
}
```

### `@NoArgsConstructor`,`@RequiredArgsConstructor`, `@AllArgsConstructor`
定义构造函数

`@NoArgsConstructor` 无参数的构造函数
`@RequiredArgsConstructor` 仅对需要的属性生成构造函数
`@AllArgsConstructor` 对每一个属性生成构造函数
注解通用属性有
`access` 构造器是修饰符
`force`  对于final修饰的变量赋值 0/false/null
`staticName` 是否对外提供工厂类方法.

```java
import lombok.*;
import org.junit.Assert;
import org.junit.Test;

/**
 * @author Jion
 * \@NoArgsConstructor 无参数构造注解.
 *      access 构造器是修饰符
 *      force  对于final修饰的变量赋值 0/false/null
 *      staticName 是否对外提供工厂类方法.
 * \@RequiredArgsConstructor 需要的构造注解
 *      会对 @NonNull 修饰的参数生成注解
 * \@ AllArgsConstructor  构造方法包含每一个参数
 *
 */
public class NoArgsConstructorAndRequiredArgsConstructorAndAllArgsConstructorAnnotateExample {

    @NoArgsConstructor
    class UserA{
        public int id;

        public String name;

        public String address;
    }
    @Test
    public void testUserA(){
        // 默认的无参数的构造方法
        UserA userA = new UserA();
        Assert.assertNotNull(userA);
    }

    @NoArgsConstructor(force = true, access = AccessLevel.PUBLIC)
    class UserB{
        public int id;

        public String name;

        public final String address;
    }

    @Test
    public void testUserB(){
        UserB userB1 = new UserB();
        Assert.assertNotNull(userB1);
        Assert.assertNull(userB1.address);
    }

    @RequiredArgsConstructor(access = AccessLevel.PUBLIC)
    class UserC{

        @NonNull
        public int id;

        @NonNull
        public String name;

        public String address;
    }

    @Test
    public void testUserC(){
        UserC userC = new UserC(1,"Jion");
        Assert.assertNull(userC.address);
        Assert.assertNotNull(userC);
    }

    @AllArgsConstructor
    class UserD{
        public int id;

        public String name;

        public String address;
    }

    @Test
    public void testUserD(){
        UserD userD = new UserD(1,"Jion","HeNan");
        Assert.assertNotNull(userD);
    }



    /*
    @Test
    public void testUser(){
        User user = User.of();
        Assert.assertNotNull(user);
    }
    */
}

/*
测试 staticName 静态方法
@NoArgsConstructor(staticName = "of")
class User{

    @NonNull
    private int id;

    @NonNull
    private String name;

    private String address;
}
*/
```

### `@Data`
简写注解,可变的Bean对象
综合 `@Getter`，`@Setter`所有非final字段,`@RequiredArgsConstructor`,`@EqualsAndHashCode`,`@ToString`注解

```java
import lombok.Data;
import lombok.NonNull;
import org.junit.Assert;
import org.junit.Test;

/**
 * @author Jion
 *  \@Data 注解.
 *      包含
 *          \@Getter，
 *          \@Setter所有非final字段,
 *          \@RequiredArgsConstructor,
 *          \@EqualsAndHashCode,
 *          \@ToString,
 */
public class DataAnnotateExample {

    @Data
    class UserA{

        @NonNull
        private int id;

        @NonNull
        private String name;

        private String address;
    }

    @Test
    public void testUserA(){
        // @RequiredArgsConstructor 要求@NonNull注解的字段作为构造函数
        UserA userA = new UserA(1,"Jion");

        // @Setter
        userA.setAddress("HeNan");

        // @Getter
        Assert.assertNotNull(userA.getAddress());

        // @EqualsAndHashCode
        System.out.println(userA.hashCode());

        // @ToString
        System.out.println(userA.toString());
    }
}
```

### `@Value`
简写注解,不可变的Bean的对象
综合 `@AllArgsConstructor`,`@Gette`,`@FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE)`,`@EqualsAndHashCode`,`@ToString` 注解

```java
import lombok.Value;
import org.junit.Assert;
import org.junit.Test;

/**
 * @author Jion
 *  \@Value 注解 不可变的数据体
 *      \@AllArgsConstructor
 *      \@Gette
 *      \@FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE)
 *      \@EqualsAndHashCode
 *      \@ToString
 */
public class ValueAnnotateExample {

    @Value
    class UserA{

        private int id;

        private String name;

        private String address;

    }

    @Test
    public void testUserA(){
        // @AllArgsConstructor
        UserA userA = new UserA(1,"Jion","HeNan");

        // @Gette
        Assert.assertNotNull(userA.getName());

        // @EqualsAndHashCode
        System.out.println(userA.hashCode());

        // @ToString
        System.out.println(userA.toString());
    }
}
```
### `@SneakyThrows`
将编译时异常改为运行时异常

```java
import lombok.SneakyThrows;
import org.junit.Test;

import java.io.*;

/**
 * @author Jion
 *  \@SneakyThrows  将当前编译时异常转为运行时异常,在运行时抛出而不是编码时.
 *      但是这样不好,接口调用者不知道有哪些可能异常.
 */
public class SneakyThrowsAnnotateExample {

    @SneakyThrows({FileNotFoundException.class, IOException.class})
    private void read(){

        File file = new File("XX");
        InputStream inputStream = new FileInputStream(file);
        inputStream.close();
    }

    @Test
    public void testRead(){
        // 运行时抛出 FileNotFoundException , IOException
        read();
    }
}
```

### `@Synchronized`
 方法锁,保持方法的线程独有性.与 `synchronized` 相似

 ```java
 import lombok.Synchronized;
 import org.junit.Assert;
 import org.junit.Test;

 /**
  * @author Jion
  *      \@Synchronized 注解
  *          与 synchronized 相似,修饰静态和实例方法.保持线程一致
  */
 public class SynchronizedAnnotateExample {

     /** 定义锁 */
     private final Object readLock = new Object();

     @Synchronized
     public static void sayHello(String name) {
         System.out.println("Hello, " + name.toUpperCase());
     }

     @Synchronized
     public double getRandom() {
         return Math.floor(Math.random() * 1000);
     }

     @Synchronized("readLock")
     public void sayHi(String name) {
         System.out.println("Hi, " + name.toUpperCase());
     }

     @Test
     public void testSayHello(){
         sayHello("Jion");
     }

     @Test
     public void testGetRandom(){
         Assert.assertNotNull(getRandom());
     }

     @Test
     public void testSayHi(){
         sayHi("Jion");
     }
 }
 ```

### `@Log`
日志组件,实例化 `java.util.logging.Logger` 类

```java
import lombok.extern.java.Log;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

import java.util.logging.Level;

/**
 * @author Jion
 *  \@Log 组件
 *  java.util.logging.Logger
 */
@Log
public class LogAnnotateExample {

    @Test
    public void testLog(){
    /*
        SEVERE > WARNING > INFO > CONFIG > FINE > FINER > FINEST
        OFF 关闭所有日志
        ALL 启用所有日志
    */
        log.setLevel(Level.FINEST);
        log.info("你好");
    }
}
```

### `@CommonsLog`
日志组件,实例化 `org.apache.commons.logging.Log` 类

```java
import lombok.extern.apachecommons.CommonsLog;
import org.junit.Test;

import java.util.logging.Level;

/**
 * @author Jion
 *  \@CommonsLog 组件
 *  org.apache.commons.logging.Log
 */
@CommonsLog
public class CommonsLogAnnotateExample {

    @Test
    public void testLog(){
    /*
        trace > debug > info > warn > error > fatal
    */
        log.trace("trace 级别");
        log.debug("debug 级别");
        log.info("info   级别");
        log.warn("warn   级别");
        log.error("error 级别");
        log.fatal("fatal 级别");
    }
}
```

### `@Slf4j`
日志组件,实例化 `org.slf4j.Logger` 类

``` java
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;

/**
 * @author Jion
 *  \@Flogger 组件
 *  org.slf4j.Logger
 */
@Slf4j
public class Slf4jAnnotateExample {

    @Test
    public void testLog(){
        /* trace < debug < info < warn < error */
        log.trace("trace 级别");
        log.debug("debug 级别");
        log.info("info   级别");
        log.warn("warn   级别");
        log.error("error 级别");
    }
}
```

### `@Flogger`
日志组件,实例化 `com.google.common.flogger.FluentLogger` 类

### `@JBossLog`
日志组件,实例化 `org.jboss.logging.Logger` 类

### `@Log4j`
日志组件,实例化 `org.apache.log4j.Logger` 类

### `@Log4j2`
日志组件,实例化 `org.apache.logging.log4j.Logger` 类

### `@XSlf4j`
日志组件,实例化 `org.slf4j.ext.XLogger` 类
