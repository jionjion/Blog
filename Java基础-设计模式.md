---
title: Java基础-设计模式
categories:
  - Java
  - Mode
tags:
  - Java
  - Mode
abbrlink: 8e9b8537
date: 2020-08-31 22:09:41
---

# 设计思想

## 核心原则

### 单一职责
一个类/方法只负责一个职责.

### 接口隔离
客户端不应该依赖它不需要的接口; 
如:当一个接口定义过多方法时,其调用者们可能不会用到全部方法,则需要将一个大接口按照方法进行分组拆分为多个小接口,以便降低隔离.

### 依赖倒转
1. 高层模块不应该依赖低层模块,二者都因该依赖 **抽象** .
2. 抽象不应该依赖细节,细节应该依赖抽象.如:在Service层依赖Dao层接口,而非实现.
3. 依赖倒转的中心思想是 **面向接口编程** .如:声明接口及其实现,通过调用接口而非实现.
4. 抽象比具体更稳固,代码变动少.
5. 使用接口/抽象制定规范,而不涉及到具体的任何操作.

依赖传递:
- 接口传递:在调用处传递接口作为方法参数.
- 构造方法传递:在构造方法中,传递接口作为构造参数
- setter方法传递:调用setter方法,传递接口作为类的成员变量

### 里式替换
在子类中尽量不要重写父类方法;
而是通过抽取相同类的共有方法提取到父类,通过组合(component),实现调用相同类之间的依赖

### 开闭原则
1. 对扩展放开发(提供方),对使修改关闭(调用方).
2. 当软件变化时,通过扩展软件而实现变化,而不是修改原有代码.
如: 接口定义和具体的实现类.

### 迪米特法则
一个类对自己依赖的类知道的越少越好.即:将内部逻辑封装,对外提供(public)方法

### 合成复用
尽量使用组合/聚合的方式,而不是继承方式
组合: 强制依赖的类关系.
聚合: 松依赖的类关系.

## 设计模式分类
- 创建型
单例模式 / 工厂模式 / 抽象工厂模式 / 原型模式 / 建造者模式
- 结构型
适配器模式 / 桥接模式 / 装饰器模式 / 组合模式 / 外观模式 / 享元模式 / 代理模式
- 行为型
模板方法模式 / 命令模式 / 访问者模式 / 迭代器模式 / 观察者模式 / 中介者模式 / 备忘录模式 / 解释器模式 / 状态模式 / 策略模式 / 责任链模式

# 设计模式

## 单例模式

适用场景

- 节约系统资源
- 获取对象不能通过new方式,而是对外提供的方法
- 适用于频繁用到的对象,创建/消耗资源的大对象,工具类对象,数据库对象

### 饿汉式(静态常量)/(静态代码块)
1. 私有构造函数
2. 类内部创建对象,(通过静态常量实例化或者静态代码块实例化)
3. 向外暴露一个静态共有方法. (`getInstance`)

#### 优缺点
优点: 简单,类加载时完成实例化,避免线程同步问题
缺点: 类加载时即占用内存,如不使用则会浪费.

#### 示例: 静态常量

```java
/**
 * 饿汉式单例模式
 * 通过静态常量
 *
 * @author Jion
 */
public class SingleExample1 {

    /**
     * 1.私有构造方法
     */
    private SingleExample1() {

    }

    /**
     * 2.创建类实例
     */
    private final static SingleExample1 instance = new SingleExample1();

    /**
     * 3.提供共有静态方法,返回实例
     */
    public static SingleExample1 getInstance() {
        return instance;
    }
}
```

#### 示例: 静态代码块

```java
/**
 * 饿汉式单例模式
 * 通过静态代码块
 *
 * @author Jion
 */
public class SingleExample2 {

    /**
     * 1.私有构造方法
     */
    private SingleExample2() {

    }

    /** 私有变量 */
    private final static SingleExample2 instance;

    /*
     * 2.在静态代码块中创建类实例
     */
    static {
        instance = new SingleExample2();
    }

    /**
     * 3.提供共有静态方法,返回实例
     */
    public static SingleExample2 getInstance() {
        return instance;
    }
}
```




### 懒汉式
1. 私有构造函数
2. 类内部创建对象引用,但不实例化
3. 向外暴露一个静态共有方法. (`getInstance`) 
当需要时,才进行实例化,并对外提供

#### 优缺点
优点: 懒加载,但仅限于单线程下. 
缺点: 通过 `synchronized` 添加同步锁,效率低

####示例: 懒汉式-单线程安全

```java
/**
 * 懒汉式单例模式
 *
 * @author Jion
 */
public class SingleExample3 {

    /**
     * 1.私有构造方法
     */
    private SingleExample3() {

    }

    /** 2.私有变量 */
    private static SingleExample3 instance;

    /**
     * 3.提供共有静态方法,返回实例.
     *  使用时创建,否则不创建
     *
     */
    public static synchronized SingleExample3 getInstance() {
        if(instance == null){
            return new SingleExample3();
        }
        return instance;
    }
}
```





### 双重检查
1. 私有构造函数
2. 类内部创建对象引用,但不实例化. 同时添加 `volatile` 关键字
3. 向外暴露一个静态共有方法. (`getInstance`)
通过使用两次检查实例对象是否为空并对实例化过程加 `synchronized` ,判断在多线程下对象只实例化一次

推荐使用 

#### 示例: 懒汉式-双重检查

```java
/**
 * 双重检查单例模式
 *
 * @author Jion
 */
public class SingleExample4 {

    /**
     * 1.私有构造方法
     */
    private SingleExample4() {

    }

    /** 2.私有变量, 添加volatile关键字,防止指令重排序,使对象完成初始化分配,刷入主内存,多线程间可见 */
    private static volatile SingleExample4 instance;

    /**
     * 3.提供共有静态方法,返回实例.
     *  使用时创建,否则不创建.
     */
    public static SingleExample4 getInstance() {
        if(instance == null){
            // 同步代码块.锁对象为实例类
            synchronized (SingleExample4.class){
                // 再次检查,避免多线程进入 getInstance , 导致最初判断失效
                if(instance == null){
                    return new SingleExample4();
                }
            }
        }
        return instance;
    }
}
```



### 静态内部类
1.当主类加载时,静态内部类不会被加载,因此可以做到延迟.
2.在用到静态内部类时,才进行装载,类的静态属性只会在第一次加载的时候初始化,所以线程安全.

推荐使用

#### 示例:静态内部类单例

```java
/**
 * 静态内部类实现单例模式
 *
 * @author Jion
 */
public class SingleExample5 {

    /**
     * 1.私有构造方法
     */
    private SingleExample5() {

    }

    /** 2.私有变量 */
    private static SingleExample5 instance;

    /** 3.声明一个静态内部类 */
    private static class SingleExample5Instance {
        // 类装载机制,在外部类装载完成后.内部类在使用时装载.装载时线程安全.
        private static final SingleExample5 INSTANCE = new SingleExample5();
    }

    /** 4.将静态内部类维护实例返回 */
    public static SingleExample5 getInstance(){
        return SingleExample5Instance.INSTANCE;
    }
}
```



### 使用枚举
JDK中提供的枚举,避免线程同步和反序列化时对象重复创建.

推荐使用

#### 示例:枚举单例

```java
/**
 * 枚举方式实现单例模式
 *
 * @author Jion
 */
public enum SingleExample6 {
	// 单例对象
    INSTANCE;
}
```



### 示例: JDK中使用单例模式
`Runtime` 类中使用,饿汉式


## 工厂模式

### 业务场景
模拟订购 Pizza 业务.
准备 -> 烘烤 -> 切割 -> 打包


### 传统方式
1. 创建基类,随后创建不同实现类
2. 根据传入参数,确认不同的创建类.
3. 调用方法
缺点,每次新增都要新建子类,并修改传入参数的判断逻辑.凡是涉及到创建的地方,都要进行判断

思路:
将创建不同子类对象过程封装到一个工厂类中,如果需要新的子类,只需要修改该工厂类即可

#### 示例:

披萨类,使用抽象方式,设置基础类及其子类

**基类:**

```java
/**
 * 披萨,抽象类
 *
 * @author Jion
 */
public abstract class Pizza {

    /** 披萨类型 */
    protected String name;


    /** 准备披萨,交给子类实现 */
    public abstract void perpare();

    /** 烘烤 */
    public void bake(){
        System.out.println(name + "烘烤披萨");
    }

    /** 切割 */
    public void cut(){
        System.out.println(name + "切割披萨");

    }
    /** 打包 */
    public void box(){
        System.out.println(name + "打包披萨");

    }

    /** 披萨名赋值 */
    public void setName(String name) {
        this.name = name;
    }
}
```

**子类:**

```java
/** 希腊披萨 */
public class GreekPizza extends Pizza{

    @Override
    public void perpare() {
        System.out.println("希腊披萨准备原材料");
    }
}

/** 起司披萨 */
public class CheesePizza extends Pizza{

    @Override
    public void perpare() {
        System.out.println("起司披萨准备原材料");
    }
}
```

**传统方式生成实例类**

```java
/**
 * 订购披萨
 *
 * @author Jion
 */
public class OrderPizza {

    /**
     * 订购披萨
     */
    public void order(String type) {
        Pizza pizza;
        System.out.println(type);
        if (type == "cheese") {
            pizza = new CheesePizza();
            pizza.setName("cheese");
        } else if (type == "greek") {
            pizza = new GreekPizza();
            pizza.setName("greek");
        } else {
            return;
        }
        // 准备, 烘烤, 切割, 打包
        pizza.perpare();
        pizza.bake();
        pizza.cut();
        pizza.box();
    }
}
```



### 简单工厂/静态工厂模式

由一个工厂进行某类对象的创建,封装了关于此对象的实例化信息
应用:当需要大量创造某个类时,或者某类型类时



#### 示例: 简单工厂模式

通过不同的类型,创建子类.不必对外暴露创建过程..

```java
/**
 * 简单工厂模式
 * 创建Pizza对象
 *
 * @author Jion
 */
public class PizzaSimpleFactory {

    /**
     * 根据类型不同,返回不同的子类
     */
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        System.out.println(type);
        if (type.equalsIgnoreCase("cheese")) {
            pizza = new CheesePizza();
            pizza.setName("cheese");
        } else if (type.equalsIgnoreCase("greek")) {
            pizza = new GreekPizza();
            pizza.setName("greek");
        }
        return pizza;
    }
}
```



### 工厂方法模式
定义了一个创建对象的方法,由子类决定要实例化的类.工厂方法模式将对象的实例化过程推迟到了子类
将简单工厂中,实例化对象功能改为抽象方法,在不同创建子类中去实现.

#### 业务模型升级
在原来披萨种类的基础中,添加地理纬度.如北京的奶酪披萨..
那么,则将原来的`perpare` 方法进行划分,根据地理位置多加一层..如: 北京起司披萨, 南京起司披萨

**示例: 模型类**

```java
/** 披萨,抽象类 */
public abstract class Pizza {

    /** 披萨类型 */
    protected String name;

    /** 准备披萨,交给子类实现 */
    public abstract void perpare();

    /** 烘烤 */
    public void bake(){
        System.out.println(name + "烘烤披萨");
    }

    /** 切割 */
    public void cut(){
        System.out.println(name + "切割披萨");

    }
    /** 打包 */
    public void box(){
        System.out.println(name + "打包披萨");

    }

    /** 披萨名赋值 */
    public void setName(String name) {
        this.name = name;
    }
}
/** 北京希腊披萨 */
public class BJGreekPizza extends Pizza {

    @Override
    public void perpare() {
        System.out.println("北京的希腊披萨准备原材料");
    }
}
/** 北京起司披萨 */
public class BJCheesePizza extends Pizza {

    @Override
    public void perpare() {
        System.out.println("北京的起司披萨准备原材料");
    }
}
/** 南京起司披萨 */
public class NJCheesePizza extends Pizza {

    @Override
    public void perpare() {
        System.out.println("南京起司披萨准备原材料");
    }
}
/** 南京希腊披萨 */
public class NJGreekPizza extends Pizza {

    @Override
    public void perpare() {
        System.out.println("南京希腊披萨准备原材料");
    }
}
```

####示例: 方法工厂方式

将 `createPizza` 方法进行抽象(创建披萨工厂), 交由不同的子类(不同地区的创建披萨工厂)去实现.并分别生成对应的披萨类

**抽象方法工厂**

```java
/**
 *  抽象工厂方法,抽象工厂
 * @author Jion
 */
public abstract class PizzaMethodFactory {

    /** 抽象方法,根据不同类型,去交由子类创建 */
    protected abstract Pizza createPizza(String type);
}
```

**具体的子类工厂实现**

```java
/** 抽象工厂方法,实现工厂 */
public class NJPizzaMethodFactory extends PizzaMethodFactory{

    /** 抽象方法,在该子类中实现. */
    @Override
    public Pizza createPizza(String type){
        Pizza pizza = null;
        System.out.println(type);
        if (type.equalsIgnoreCase("cheese")) {
            pizza = new NJCheesePizza();
            pizza.setName("南京 cheese");
        } else if (type.equalsIgnoreCase("greek")) {
            pizza = new NJGreekPizza();
            pizza.setName("南京 greek");
        }
        return pizza;
    }
}

/** 抽象工厂方法,实现工厂 */
public class BJPizzaMethodFactory extends PizzaMethodFactory{

    /** 抽象方法,在该子类中实现. */
    @Override
    public Pizza createPizza(String type){
        Pizza pizza = null;
        System.out.println(type);
        if (type.equalsIgnoreCase("cheese")) {
            pizza = new BJCheesePizza();
            pizza.setName("北京 cheese");
        } else if (type.equalsIgnoreCase("greek")) {
            pizza = new BJGreekPizza();
            pizza.setName("北京 greek");
        }
        return pizza;
    }
}
```



### 抽象工厂模式
通过接口创建相关工厂簇,无需指定具体的类.
将工厂分为接口类和具体实现, 接口工厂(定义创建披萨工厂方法)和实现的工厂子类(不同地区的创建披萨工厂).

#### 示例: 抽象工厂模式

**定义接口方法**

```java
/**
 *  抽象工厂接口,定义相关依赖
 * @author Jion
 */
public interface AbstractPizzaFactory {

    /** 接口方法,根据不同类型,去交由子类创建 */
    Pizza createPizza(String type);
}
```

**实现接口的工厂类**

```java
/** 抽象工厂方法,实现工厂 */
public class BJPizzaMethodFactory implements AbstractPizzaFactory{

    /** 接口方法,在该子类中实现. */
    public Pizza createPizza(String type){
        Pizza pizza = null;
        System.out.println(type);
        if (type.equalsIgnoreCase("cheese")) {
            pizza = new BJCheesePizza();
            pizza.setName("北京 cheese");
        } else if (type.equalsIgnoreCase("greek")) {
            pizza = new BJGreekPizza();
            pizza.setName("北京 greek");
        }
        return pizza;
    }
}
/** 抽象工厂方法,实现工厂 */
public class NJPizzaMethodFactory implements AbstractPizzaFactory{

    /** 接口方法,在该子类中实现. */
    public Pizza createPizza(String type){
        Pizza pizza = null;
        System.out.println(type);
        if (type.equalsIgnoreCase("cheese")) {
            pizza = new NJCheesePizza();
            pizza.setName("南京 cheese");
        } else if (type.equalsIgnoreCase("greek")) {
            pizza = new NJGreekPizza();
            pizza.setName("南京 greek");
        }
        return pizza;
    }
}
```



### 示例: JDK中使用
`java.util.Calendar` 类中使用简单工厂模式



## 原型模式
将一个对象 `.clone()` 拷贝多个实例.

### `clone` 拷贝

实现 `java.lang.Cloneable` 接口并重新 `clone` 方法.否则抛出 `CloneNotSupportedException` 异常



#### 示例: `clone` 拷贝

声明一个原型类, 并实现 `Cloneable` 接口

```java
/**
 * 原型模式,羊
 *
 * @author Jion
 */
public class Sheep implements Cloneable{
    private String name;

    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Sheep() {
    }

    public Sheep(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    /** 复制该实例 */
    @Override
    protected Object clone() throws CloneNotSupportedException {
        // 默认示例
        return super.clone();
    }
}
```

**测试类**

```java
public class SheepTest {

    @Test
    public void test() throws CloneNotSupportedException {
        // 浅拷贝
        Sheep sheep1 = new Sheep("Jion", 1);
        Sheep sheep2 = (Sheep) sheep1.clone();
        Sheep sheep3 = (Sheep) sheep1.clone();

        // 查看
        System.out.println("sheep1 " + sheep1.toString());
        System.out.println("sheep2 " + sheep2.toString());
        System.out.println("sheep3 " + sheep3.toString());
    }
}
```



 ### 深拷贝与浅拷贝

浅拷贝在拷贝时,只是将引用数据类型的引用地址拷贝,基本类型数值拷贝.

必须要重写 `.clone()`方法, 另外实现以下接口:
`Cloneable` 便于克隆,否则抛出 `CloneNotSupportedException`
`Serializable` 便于序列化,否则抛出 `NotSerializableException`

#### 示例: 浅拷贝

创建拷贝依赖类

```java
public class Sheep implements Cloneable {
    private String name;

    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Sheep() {
    }

    public Sheep(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    /** 复制改实例 */
    @Override
    protected Object clone() throws CloneNotSupportedException {
        // 默认示例
        return super.clone();
    }
}
```

浅拷贝的对象

```java
public class ShallowProtoType implements Cloneable  {

    // 引用类型
    private Sheep sheep;


    public Sheep getSheep() {
        return sheep;
    }

    public void setSheep(Sheep sheep) {
        this.sheep = sheep;
    }

    public ShallowProtoType() {

    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

测试.浅拷贝

```java
public class ShallowProtoTypeTest {

    @Test
    public void test() throws CloneNotSupportedException {
        // 浅拷贝
        ShallowProtoType shallowProtoType1 = new ShallowProtoType();
        shallowProtoType1.setSheep(new Sheep("Jion", 1));
        ShallowProtoType shallowProtoType2 = (ShallowProtoType) shallowProtoType1.clone();
        ShallowProtoType shallowProtoType3 = (ShallowProtoType) shallowProtoType1.clone();

        // 查看,对象地址不同,但是成员变量引用相同
        System.out.println("DeepProtoType1 " + shallowProtoType1.toString());
        System.out.println("DeepProtoType1 " + shallowProtoType1.getSheep().hashCode());

        System.out.println("DeepProtoType2 " + shallowProtoType2.toString());
        System.out.println("DeepProtoType2 " + shallowProtoType2.getSheep().hashCode());

        System.out.println("DeepProtoType3 " + shallowProtoType3.toString());
        System.out.println("DeepProtoType3 " + shallowProtoType3.getSheep().hashCode());
    }
}
```

#### 示例: 深拷贝

拷贝目标类.

1. 可以用过重写 `clone` 方法并实现 `Cloneable` 接口, 对每一个引用类型数据依次进行拷贝复制.
2. 使用序列化方法并实现 `Serializable` 接口, 将对象依次序列化再反序列化, 最后生成.

```java

import java.io.*;

/**
 *  深拷贝
 * @author Jion
 */
public class DeepProtoType implements Serializable, Cloneable {

    // 引用类型
    private Sheep sheep;

    public Sheep getSheep() {
        return sheep;
    }

    public void setSheep(Sheep sheep) {
        this.sheep = sheep;
    }

    public DeepProtoType() {

    }

    /** 深拷贝, 重写Clone方法 */
    @Override
    protected Object clone() throws CloneNotSupportedException {
        // 返回对象
        Object deepProto = null;
        // 1.调用父类的clone方法,对基础数据类型进行浅拷贝
        deepProto = super.clone();
        DeepProtoType clone = (DeepProtoType) deepProto;
        // 2.单独处理引用类型的成员属性
        clone.setSheep((Sheep) this.sheep.clone());
        return super.clone();
    }

    /** 深拷贝,通过序列化 */
    public Object deepClone() throws IOException, ClassNotFoundException {
        // 字节对象
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
        // 序列化对象
        objectOutputStream.writeObject(this);
        // 反序列化对象
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
        ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);
        // 返回结果
        return objectInputStream.readObject();
    }
}
```

测试类

```java
public class DeepProtoTypeTest {


    @Test
    public void test() throws CloneNotSupportedException {
        // 浅拷贝
        DeepProtoType deepProtoType1 = new DeepProtoType();
        deepProtoType1.setSheep(new Sheep("Jion", 1));
        DeepProtoType deepProtoType2 = (DeepProtoType) deepProtoType1.clone();
        DeepProtoType deepProtoType3 = (DeepProtoType) deepProtoType1.clone();

        // 查看
        System.out.println("DeepProtoType1 " + deepProtoType1.toString());
        System.out.println("DeepProtoType1 " + deepProtoType1.getSheep().hashCode());

        System.out.println("DeepProtoType2 " + deepProtoType2.toString());
        System.out.println("DeepProtoType1 " + deepProtoType2.getSheep().hashCode());

        System.out.println("DeepProtoType3 " + deepProtoType3.toString());
        System.out.println("DeepProtoType1 " + deepProtoType3.getSheep().hashCode());
    }


    @Test
    public void deepClone() throws IOException, ClassNotFoundException {
        // 浅拷贝
        DeepProtoType deepProtoType1 = new DeepProtoType();
        deepProtoType1.setSheep(new Sheep("Jion", 1));
        DeepProtoType deepProtoType2 = (DeepProtoType) deepProtoType1.deepClone();
        DeepProtoType deepProtoType3 = (DeepProtoType) deepProtoType1.deepClone();

        // 查看
        System.out.println("DeepProtoType1 " + deepProtoType1.toString());
        System.out.println("DeepProtoType1 " + deepProtoType1.getSheep().hashCode());

        System.out.println("DeepProtoType2 " + deepProtoType2.toString());
        System.out.println("DeepProtoType1 " + deepProtoType2.getSheep().hashCode());
        
        System.out.println("DeepProtoType3 " + deepProtoType3.toString());
        System.out.println("DeepProtoType1 " + deepProtoType3.getSheep().hashCode());
    }
}
```



## 建造者模式
将最终产品与建造过程相分离,只需要提供每一步所需要的对象,即可完成最后的产品

**四个角色**
产品: 最终生成的对象
抽象建造者: 接口或抽象类,指定每一步所需的部件
具体建造者: 实现接口,并为产品赋值组件
指挥者(调用方): 实例化具体的构造者,并控制产品的生产过程.

### 示例: 建造者方法

产品角色

```java
/**
 * 产品
 *
 * @author Jion
 */
public class House {

    private String basic;
    private String walls;
    private String roofed;

    public String getBasic() {
        return basic;
    }

    public void setBasic(String basic) {
        this.basic = basic;
    }

    public String getWalls() {
        return walls;
    }

    public void setWalls(String walls) {
        this.walls = walls;
    }

    public String getRoofed() {
        return roofed;
    }

    public void setRoofed(String roofed) {
        this.roofed = roofed;
    }
}
```

抽象建造者

```java
/**
 * 抽象建造者
 *
 * @author Jion
 */
public abstract class AbstractHouseBuilder {

    protected House house = new House();

    public abstract void buildBasic();

    public abstract void buildWalls();

    public abstract void buildRoofed();

    /** 建造完成后,返回产品 */
    public House build() {
        return house;
    }
}
```

具体建造者

```java
/**
 * 具体建造者
 *
 * @author Jion
 */
public class TowerHouseBuilder extends AbstractHouseBuilder {

    @Override
    public void buildBasic() {
        super.house.setBasic("地基");
        System.out.println("高楼打地基...");
    }

    @Override
    public void buildWalls() {
        super.house.setWalls("砌墙");
        System.out.println("高楼砌墙...");
    }

    @Override
    public void buildRoofed() {
        super.house.setRoofed("封顶");
        System.out.println("高楼封顶...");
    }
}
```

指挥者

```java
/**
 * 指挥者,限定建造顺序
 *
 * @author Jion
 */
public class HouseDirector {

    private AbstractHouseBuilder abstractHouseBuilder;

    public AbstractHouseBuilder getAbstractHouseBuilder() {
        return abstractHouseBuilder;
    }

    public void setAbstractHouseBuilder(AbstractHouseBuilder abstractHouseBuilder) {
        this.abstractHouseBuilder = abstractHouseBuilder;
    }

    /** 指定具体的顺序 */
    public House constructHouse(){
        abstractHouseBuilder.buildBasic();
        abstractHouseBuilder.buildWalls();
        abstractHouseBuilder.buildRoofed();
        return abstractHouseBuilder.house;
    }
}
```

测试类

```java
public class HouseDirectorTest {

    @Test
    public void test(){
        // 指挥者
        HouseDirector houseDirector = new HouseDirector();
        // 具体建造类
        houseDirector.setAbstractHouseBuilder(new TowerHouseBuilder());
        // 建造
        House house = houseDirector.constructHouse();
        // 完成
        System.out.println("House " + house.toString());
    }
}
```



### 示例: JDK中
`java.lang.String` 产品
`java.lang.Appendable` 抽象建造者
`java.lang.AbstractStringBuilder` 具体建造者,但不能实例化
`java.lang.StringBuilder` 指挥者兼具体建造者



## 桥接模式

将抽象和实现两个维度分别独立实现,从而替代多层继承实现.
基于类的最小设计原则,通过封装/继承/组合等行为,将不同的类承担不同的责任.

**角色:**
抽象层: 分类依据,一方接口一方抽象类,并在抽象类中组合接口.
实现层: 具体实现.常用构造器组合接口实现.

### 适用于
1. 解决多层次继承导致类的数量急剧增加,桥接模式较为适合
2. 应用场景: JDBC.

### 业务场景
手机分类.首先是品牌分类,然后是样式分类.(两个维度)
`Brand`品牌类接口,约定不同的手机品牌.
分别具有不同的品牌实现,如`XiaoMi`和`Vivo`
`Phone`手机样式抽象类,约定不同的手机样式.同时,将`Brand`接口通过构造器注入.
子类为不同样式的手机,如`FoldedPhone`和`TouchPhone`.
其中,子类通过构造器,中传入的品牌实现,去调用不同的品牌方法.`Phone` 抽象类充当桥梁,连接子类样式与具体的品牌实现

### 示例: 桥接模式

模拟手机业务,不同的手机有不同的品牌,不同的品牌有不同的样式,尝试解决类多层继承导致类爆炸问题.

品牌类,定义品牌及手机基础功能

```java
/**
 * 品牌类,定义手机品牌
 *
 * @author Jion
 */
public interface Brand {

    void open();

    void close();

    void call();
}
```

具体的品牌实现,通过实现接口完成

```java
/** Vivo品牌手机 */
public class Vivo implements Brand {
    public void open() {
        System.out.println("Vivo手机开机...");
    }

    public void close() {
        System.out.println("Vivo手机关机...");
    }

    public void call() {
        System.out.println("Vivo手机打电话...");
    }
}

/** XiaoMi品牌手机 */
public class XiaoMi implements Brand {
    public void open() {
        System.out.println("小米手机开机...");
    }

    public void close() {
        System.out.println("小米手机关机...");
    }

    public void call() {
        System.out.println("小米手机打电话...");
    }
}
```

定义手机类,抽象类. 将品牌类通过构造器进行组合

```java
/**
 * 手机,抽象类,组合品牌类型.
 * 起到桥接的作用.
 *
 * @author Jion
 */
public abstract class Phone {

    /** 桥,一方组合品牌 */
    private Brand brand;

    /** 构造器传入,组合 */
    public Phone(Brand brand){
        super();
        this.brand = brand;
    }

    /** 桥,让另一方被继承子类调用组合方法 */
    protected void open(){
        this.brand.open();
    }

    /** 调用组合方法 */
    protected void close(){
        this.brand.close();
    }

    /** 调用组合方法 */
    protected void call(){
        this.brand.call();
    }
}
```

具体的手机子类,通过继承父类,并构造器传入接口类.实现从抽象类实现方调用接口实现类方的过程.其过程类似过桥...
新增手机类型,只需要新增抽象实现类即可,无需多层继承.

```java
/**
 * 组合, 触摸屏手机
 * 在调用时,传入不同的品牌,完成桥接组合
 */
public class TouchPhone extends Phone {

    /**
     * 继承自父类,父类无隐式父类构造器,必须显示调用父类有构造器.传入对象
     */
    public TouchPhone(Brand brand) {
        super(brand);
    }

    /**
     * 重写抽象父类方法
     */
    @Override
    protected void open() {
        super.open();
        System.out.println("触摸屏样式手机打开");
    }

    @Override
    protected void close() {
        super.close();
        System.out.println("触摸屏样式手机关闭");
    }

    @Override
    protected void call() {
        super.call();
        System.out.println("触摸屏样式手机打电话");
    }
}

/** 组合, 折叠手机 */
public class FoldedPhone extends Phone {

    /** 继承自父类,父类无隐式父类构造器,必须显示调用父类有构造器.传入对象 */
    public FoldedPhone(Brand brand) {
        super(brand);
    }

    /** 重写抽象父类方法 */
    @Override
    protected void open() {
        super.open();
        System.out.println("折叠样式手机打开");
    }

    @Override
    protected void close() {
        super.close();
        System.out.println("折叠样式手机关闭");
    }

    @Override
    protected void call() {
        super.call();
        System.out.println("折叠样式手机打电话");
    }
}
```

