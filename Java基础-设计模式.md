---
title: Java基础-设计模式
abbrlink: 8e9b8537
date: 2020-08-31 22:09:41
categories:
  - Java
  - Mode
tags: [Java, Mode]
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

## 设计模式适用

| 设计模式     | 解决问题                                                     | 场景示例     |
| ------------ | ------------------------------------------------------------ | ------------ |
| 单例模式     | 唯一实例创建问题                                             | 工具类       |
| 工厂模式     | 同一基类,不同实现类的创建问题                                | 工厂类       |
| 抽象工厂模式 | 相似基类,不同实现类的创建问题                                | 抽象工厂类   |
| 原型模式     | 对象克隆创建问题                                             |              |
| 建造者模式   | 将繁琐的对象创建过程拆分                                     |              |
| 适配器模式   | 不同类和方法间调用问题                                       | 日志适配器类 |
| 桥接模式     | 类多继承导致类数量爆炸问题                                   |              |
| 装饰器模式   | 动态地将新功能添加到新对象上,解决多实例间组合问题            | `IO` 流      |
| 组合模式     | 处理的对象具有树形结构,且叶子与枝杈具有相似的方法属性        |              |
| 外观模式     | 通过组合以屏蔽子系统细节,使得调用只跟接口发生,无关子系统内部 |              |
| 享元模式     | 解决重复对象的内存浪费问题,共享相同的对象                    | 连接池类     |
| 代理模式     | 通过该代理完成对象的调用,以期增强其特性,而无需修改原有代码   | 代理类       |
| 模板模式     | 为了完成某个过程,需要一系列步骤,但是稍有不同的问题           | 模板类       |
| 命令模式     | 将请求发送与接收动作相解耦,无需关注相互细节                  |              |
| 访问者模式   | 在系统中有一个稳定的数据结构,但有经常变化的功能需求          |              |
| 迭代器模式   | 提供统一的迭代器完成遍历,而不用暴露内部数据结构              | 迭代类       |
| 观察者模式   | 当一方变化时,通知多方观察者                                  | 天气预报     |
| 中介者模式   | 用一个中介者封装一系列的对象交互,使对象之间无需重复交互引用  |              |
| 备忘录模式   | 提供一个可以恢复对象状态的机制                               | 数据状态恢复 |
| 解释器模式   | 对自定义语言进行解释执行                                     |              |
| 状态模式     | 对象多种状态间转换时,对外输出不同的执行                      |              |
| 策略模式     | 通过定义算法族,分别封装,以便相互替换                         |              |
| 责任链模式   | 为请求创建一个接受请求的链                                   |              |



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



## 装饰者模式

动态地将一组相似新功能的对象附加到装饰类上, 避免重复创建相似对象,频繁修改装饰类

被装饰者: 具体一组子类,实现在抽象类/接口中约定公共属性/方法.
装饰者: 连接被装饰者与饰品
装饰品: 装饰者的子类,并修改被装饰者属性.

### 业务场景

咖啡厅提供很多单体咖啡和调料,为了将其便于调制咖啡,计算价格,因此使用装饰者模式
调料(装饰者),包装了单体咖啡(被装饰者),完成组合的构建(构造器注入)

饮料类,抽象约定一组相似方法.如单体价格和计算总价

```java
public abstract class Drink {
    /** 描述 */
    private String description;

    /** 价格 */
    private Double price;

    /** 计算价格方法 */
    protected abstract Double cose();

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }
}
```

单体咖啡类,继承自抽象类,并实现其约定方法与规定属性

```java
/** 开发类中间类. */
public class Coffee extends Drink{
    /** 计算价格方法 */
    @Override
    public Double cose() {
        return super.getPrice();
    }
}

/** 美式咖啡 */
public class LongBlackCoffee extends Coffee {

    /** 构造器 */
    public LongBlackCoffee() {
        // 为其指定属性
        setDescription("美式咖啡");
        setPrice(5D);
    }
}

/** 黑咖啡 */
public class ShortBlackCoffee extends Coffee {
    public ShortBlackCoffee() {
        setDescription("黑咖啡");
        setPrice(4D);
    }
}
```

装饰者, 作为装饰品的父类,连接被装饰对象与装饰品. 实现父类约定的方法与属性

```java
/** 装饰者.连接单体咖啡与其调料 */
public class Decorator extends Drink {
    /** 引入单体咖啡 */
    private Drink obj;

    public Decorator(Drink obj){
        this.obj = obj;
    }

    /** 计算价格,装饰者 */
    @Override
    public Double cose() {
        // 自己的价格 + 引入单体咖啡的价格
        return super.getPrice() + obj.cose();
    }

    /** 描述: 调料 + 价格 + 单体咖啡  */
    @Override
    public String getDescription() {
        return super.getDescription() + ":" + super.getPrice() + " && " + obj.getDescription();
    }
}
```

装饰品,继承自装饰者类,完善具体装饰品信息

```java
/** 具体的调味品,牛奶 */
public class Milk extends Decorator{
    /** 调味品,种类与价格 */
    public Milk(Drink obj) {
        super(obj);
        setDescription("牛奶调味品..");
        setPrice(2.2D);
    }
}

/** 具体的调味品,巧克力 */
public class Chocolate extends Decorator{
    /** 调味品,种类与价格 */
    public Chocolate(Drink obj) {
        super(obj);
        setDescription("巧克力调味品..");
        setPrice(3.3D);
    }
}
```

测试,使用装饰者模式计算价格

```java
public class DecoratorTest {

    @Test
    public void test(){
        // 1.确定单体咖啡
        Drink coffee = new LongBlackCoffee();
        // 尝试..计算价格..=> 5
        System.out.println(coffee.getDescription());
        System.out.println(coffee.cose());

        // 2.加入调料,新的饮料,构造器注入之前的实例
        Milk coffeeMilk = new Milk(coffee);
        // 尝试..计算价格..=> 5 + 2.2 = 7.7
        System.out.println(coffeeMilk.getDescription());
        System.out.println(coffeeMilk.cose());

        // 3.加入调料,新的饮料,构造器注入之前的实例
        Chocolate coffeeMilkChocolate = new Chocolate(coffeeMilk);
        // 尝试..计算价格..=> 5 + 2.2 + 3.3 = 10.5
        System.out.println(coffeeMilkChocolate.getDescription());
        System.out.println(coffeeMilkChocolate.cose());
    }
}
```

### 示例: JDK中使用

`IO` 流的使用
`InputSteam` :抽象类
`FileInputStream` : 子类,装饰者
`SocketInputStream` : 子子类,被装饰者

## 组合模式

创建了对象的树形结构,将对象组合.无需考虑具体节点层级与操作



### 业务场景
构成一个学校的 校/院/系, 且具有相同的操作习惯

组织类,抽象类,约定共有方法,操作树节点

```java
/** 定义节点方法 */
public abstract class Organization {

    /** 名字 */
    private String name;

    /** 说明 */
    private String description;

    public Organization(String name, String description){
        super();
        this.name = name;
        this.description = description;
    }

    /** 添加 */
    protected void add(Organization organization){

    }

    /** 删除 */
    protected void remove(Organization organization){

    }

    /** 子类必须实现, 抽象方法 */
    protected abstract void print();


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

第一层,学校节点,持有子节点(用抽象类代替)集合

```java
/** 大学,管理下属学院 */
public class University extends Organization {

    /** 包含学院, 用抽象类代替学院实现 */
    private List<Organization> universityList = new ArrayList<Organization>();

    /** 构造器 */
    public University(String name, String description) {
        super(name, description);
    }

    /** 添加 */
    @Override
    protected void add(Organization organization){
        universityList.add(organization);
    }

    /** 删除 */
    @Override
    protected void remove(Organization organization){
        universityList.remove(organization);
    }

    /** 打印 */
    @Override
    public void print() {
        System.out.println("当前学院:" + universityList.toString());
    }
}
```

第二层, 学院节点.

```java
/** 学院,管理下属院系 */
public class College extends Organization {

    /** 包含院系, 用抽象类代替院系实现 */
    private List<Organization> collegeList = new ArrayList<Organization>();

    /** 构造器 */
    public College(String name, String description) {
        super(name, description);
    }

    /** 添加 */
    @Override
    protected void add(Organization organization){
        collegeList.add(organization);
    }

    /** 删除 */
    @Override
    protected void remove(Organization organization){
        collegeList.remove(organization);
    }

    /** 打印 */
    @Override
    public void print() {
        System.out.println("当前院系:" + collegeList.toString());
    }
}

```

第三层,最小单位,叶子节点,有限度地支持部分方法

```java
/** 系..最小单位 */
public class Department extends Organization {

    /** 构造器 */
    public Department(String name, String description) {
        super(name, description);
    }

    /** 打印 */
    @Override
    public void print() {
        System.out.println("当前系:" + super.getName());
    }
}
```

测试类

```java
public class OrganizationTest {
    @Test
    public void test(){
        // 大学
        Organization university = new University("河南大学", "河南自己的大学...");
        // 学院
        Organization collegeA = new College("计算机学院", "这是计算机学院");
        Organization collegeB = new College("机械学院", "这是机械学院");
        // 系
        Organization DepartmentA1 = new Department("软件工程", "这是软件工程...");
        Organization DepartmentA2 = new Department("网络工程", "这是网络工程...");
        Organization DepartmentB1 = new Department("机械制造", "这是机械制造...");
        Organization DepartmentB2 = new Department("车辆制造", "这是车辆制造...");

        // 动作组合
        collegeA.add(DepartmentA1);
        collegeA.add(DepartmentA2);
        collegeB.add(DepartmentB1);
        collegeA.add(DepartmentB2);
        university.add(collegeA);
        university.add(collegeB);

        // 组合结果, 使用学校级别, 学院级别, 学系级别
        university.print();
        collegeA.print();
        DepartmentA1.print();
    }
}
```

## 外观模式
为子系统的一组接口提供了一个一致的界面,用以屏蔽子系统细节,使得调用只跟接口发生,无关子系统内部.
当系统特别复杂时,可以更好的控制访问的层次

### 业务场景
遥控器控制家庭影院,如DVD, TV...

各种家电类,

```java
/** DVD 播放器 */
public class DvdPlayer {

    public void on(){
        System.out.println("DvD 打开了");
    }

    public void off(){
        System.out.println("DvD 关闭了");
    }

    public void play(){
        System.out.println("DvD 播放了");
    }
}

/** Mp3 播放器 */
public class Mp3Player {

    public void on(){
        System.out.println("Mp3 打开了");
    }

    public void off(){
        System.out.println("Mp3 关闭了");
    }

    public void fm(){
        System.out.println("Mp3 收音了");
    }
}

/** TV 电视机 */
public class TvPlayer {

    public void on(){
        System.out.println("TV 打开了");
    }

    public void off(){
        System.out.println("TV 关闭了");
    }

    public void change(){
        System.out.println("TV 换台了");
    }
}

```

外观类,组合当前家电

```java
/** 外观类 */
public class HomeFacade {

    // 各个子系统对象
    /** DVD */
    private final DvdPlayer dvdPlayer;

    /** Mp3 */
    private final Mp3Player mp3Player;

    /** Tv */
    private final TvPlayer tvPlayer;

    /** 构造器注入依赖 */
    public HomeFacade(){
        dvdPlayer = new DvdPlayer();
        mp3Player = new Mp3Player();
        tvPlayer = new TvPlayer();
    }

    /** 组织各个方法 */
    public void ready(){
        dvdPlayer.on();
        mp3Player.on();
        tvPlayer.on();
    }

    /** 组织各个方法 */
    public void enjoy(){
        dvdPlayer.play();
        mp3Player.fm();
        tvPlayer.change();
    }

    /** 组织各个方法 */
    public void end(){
        dvdPlayer.off();
        mp3Player.off();
        tvPlayer.off();
    }
}
```

测试类

```java
public class HomeFacadeTest {

    @Test
    public void test(){
        HomeFacade homeFacade = new HomeFacade();
        homeFacade.ready();
        homeFacade.enjoy();
        homeFacade.end();
    }
}
```

## 享元模式

共享相同的对象,解决代码重复问题.常用作池对象,解决内存重复占用问题.

角色:
产品抽象角色: 定义外部状态(不同的代码)和内部状态(相同的代码),并声明为对应接口
具体的产品角色: 实现抽象角色定义的相关方法
不可共享角色: 不共享的角色,不会出现在享元工厂中
享元工厂: 构建池容器,集合维护产品对象.

### 业务场景
网站,根据其分类分为新闻类, 运动类, 并交由不同的用户去订阅使用.根据享元模式,可以将新闻类创建放入池中,交由用户调用.

用户信息

```java
/** 不同对象间的不同实例 */
public class User {

    private final String username;

    public User(String username) {
        this.username = username;
    }

    public String getUsername() {
        return username;
    }
}
```



基础网站, 差异点为不同用户

```java
/** 抽象类,定义抽象方法 */
public abstract class Website {

    /** 相似的抽相方法, User为不同点 */
    protected abstract void use(User user);
}

```

具体的新闻网站,定义使用方法 `use` .

```java
/** 具体子类,新闻类型 */
public class NewWebsite extends Website {

    /** 具体的新闻类别,相似的部分 */
    private final String type;


    public NewWebsite(String type) {
        this.type = type;
    }

    @Override
    protected void use(User user) {
        System.out.println("当前对象使用者: " + user.getUsername());
        System.out.println("新闻网站的类型为: " + type + " ,对象哈希为: " + this.hashCode());
    }
}
```

工厂类, 维护池中站点信息

```java
/** 网站工厂类,根据需求返回池中的对象 */
public class WebsitePool {

    /** 集合对象 */
    private final HashMap<String, NewWebsite> pool = new HashMap<String, NewWebsite>(10);

    /** 获得对象,存在拿去,否则创建返回并存放 */
    public Website getWebsite(String type){
        // 不存在,放入
        if(!pool.containsKey(type)){
            pool.put(type, new NewWebsite(type));
        }
        return pool.get(type);
    }
}
```

测试

```java
public class WebsiteTest {

    @Test
    public void test(){
        WebsitePool pool = new WebsitePool();

        // 获得,并存放一个
        User jion = new User("Jion");
        Website newWebsite = pool.getWebsite("NEW");
        newWebsite.use(jion);
        // 再次获得一个
        User arise = new User("Arise");
        Website musicWebsite = pool.getWebsite("Music");
        musicWebsite.use(arise);
        // 重试从缓存中,相同对象,不同调用
        User bom = new User("Bom");
        Website cacheWebsite = pool.getWebsite("NEW");
        cacheWebsite.use(bom);
    }
}
```

### 示例: JDK中使用

`java.lang.Integer` 中使用, 正范围[-128,127]内的对象,通过 `valueOf` 方法获得的为缓存数组中的同一个对象.



## 代理模式
为对象提供一个代理,通过该代理完成对象的调用,以期增强其特性,而无需修改原有代码

### 静态代理
定义接口或者父类,代理类与目标类实现相同的父类.通过调用相同的方法,实现对目标类的调用.

优点: 简单,只要实现接口和父类即可
缺点: 如果接口增加,需要实现相同的方法

#### 示例: 静态代理

接口, 定义方法. 被代理对象与代理类均实现

```java
/** 代理类与目标类约定方法 */
public interface Teacher {
    /** 共有方法 */
    void tell();
}
```

被代理对象, 实现方法

```java
/** 目标类 */
public class MathTeacher implements Teacher {

    /** 目标类,方法 */
    public void tell() {
        System.out.println("老师正在授课...");
    }
}
```

代理类, 通过实现共同的接口, 并聚合静态代理对象, 扩展方法

```java
/** 代理类,通过静态代理,扩展方法 */
public class ProxyTeacher implements Teacher{

    /** 静态代理对象 */
    private final Teacher teacher = new MathTeacher();

    /** 代理类,方法 */
    public void tell() {
        System.out.println("代理类执行...");
        teacher.tell();
    }
}
```



### JDK代理
代理类无需实现接口,但是**被代理对象需要实现接口**.通过jdk中的方法

#### 示例: JDK代理

接口类,定义方法

```java
/** 目标类约定方法 */
public interface Teacher {
    /** 共有方法 */
    void tell();
}
```

被代理类,必须实现接口

```java
/** 目标类,必须实现接口 */
public class MathTeacher implements Teacher {
    /** 目标类,方法 */
    public void tell() {
        System.out.println("老师正在授课...");
    }
}

```

代理类,通过 `Proxy.newProxyInstance()` 传入类加载器,获得代理对象,并执行 `InvocationHandler` 中扩展方法

```java
/** 通过JDK动态生成代理对象 */
public class ProxyTeacherFactory {

    /** 获得代理对象 */
    public Object getProxyInstance(final Object target) {

        // 类加载器 ; 目标类的所有实现接口 ; 事件处理器
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("代理开始...");
                        Object result = method.invoke(target, args);
                        System.out.println("代理结束...");
                        return result;
                    }
                });
    }
}
```



### CGlib代理

**被代理对象无需实现接口或父类**,即可完成代理(被代理对象不能为final类/static类).
通过内存中添加代理类的子类,完成对目标类的代理.



被代理对象, 无需实现接口

```java
/** 目标类,不必实现接口 */
public class MathTeacher  {
    /** 目标类,方法 */
    public void tell() {
        System.out.println("老师正在授课...");
    }
}
```

代理类

```java
/** 创建代理类 */
public class ProxyTeacherFactory implements MethodInterceptor {

    /** 维护一个目标对象 */
    private final Object target;

    public ProxyTeacherFactory(Object target){
        this.target = target;
    }

    /** 返回代理对象 */
    public Object getInstance(){
        // 1. 创建工具类
        Enhancer enhancer = new Enhancer();

        // 2. 设置父类
        enhancer.setSuperclass(target.getClass());

        // 3. 设置回调函数
        enhancer.setCallback(this);

        // 4. 创建子类及代理
        return enhancer.create();
    }

    /**
     *  使用Cglib进行代理, 改方法中调用目标方法
     * @param obj 被代理对象
     * @param method 代理方法
     * @param args  代理方法参数
     * @param methodProxy 调用方法返回结果
     * @return 调用方法返回结果
     * @throws Throwable 各种异常
     */
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("代理开始...");
        // 代理方法反射调用,传入源目标类及方法参数
        Object result = method.invoke(target, args);
        System.out.println("代理结束...");
        return result;
    }
}
```

## 模板模式
在抽象类中定义了一组执行方法,子类继承实现具体的方法.
但是调用仍然以抽象类中的定义顺序执行.

钩子方法:
父类中定义的方法,默认不作任何事情,子类选择性覆盖.

### 业务场景: 
制作饮料,口味不同.具有相似的步骤.

抽象类,定义模板方法. 注意方法用 `final` 修饰

```java
/** 抽象类,定义模板 */
public abstract class AbstractTea {

    /** 模板方法,不允许修改 */
    public final void make() {
        select();

        boil();

        finish();

        // 钩子方法
        if (isPack()) {
            pack();
        }
    }

    /** 1. 选材,抽象类,子类必须实现 */
    protected abstract void select();


    /** 2. 煮沸,子类选择实现 */
    protected void boil() {
        System.out.println("煮沸中...");
    }

    /** 3. 完成 */
    public void finish() {
        System.out.println("制作完成...");
    }

    /** 4. 钩子方法,是否打包 */
    public void pack() {
        System.out.println("打包....");
    }

    public boolean isPack() {
        return false;
    }
}
```

红茶制作, 重写模板中 `boil` 某些步骤的方法.

```java
/** 红茶的制作 */
public class BlackTea extends AbstractTea {

    @Override
    protected void select() {
        System.out.println("选择红茶...");
    }

    @Override
    protected void boil() {
        super.boil();
        System.out.println("红茶煮沸中...");
    }
}

```

绿茶制作, 重写钩子 `isPack` 方法, 动态修改模板方法.

```java
/** 绿茶的制作 */
public class GreenTea extends AbstractTea {

    @Override
    protected void select() {
        System.out.println("选择绿茶...");
    }

    @Override
    public void pack() {
        super.pack();
    }

    /** 重写父类方法,钩子方法执行 */
    @Override
    public boolean isPack(){
        return true;
    }
}
```



## 命令模式
将请求发送者与接收者相解耦.
每一个命令作为一个对象,使用不同的参数代表不同的命令,支持命令的撤销

场景: 模拟CMD命令, 订单撤销/恢复, 触发-反馈机制

### 角色
调用者:   发布命令
命令:     定义命令(支持撤销)
具体命令: 具体内容,持有被调用者    
被调用者: 接受命令,执行具体的请求

### 业务场景
家电有多个遥控器,需要使用命令模式进行操作

被调用者, 具体的家电及其提供具体的功能.

```java
/** 电灯遥控器 */
public class LightReceiver {

    /** 开灯 */
    public void on() {
        System.out.println("灯打开....");
    }

    /** 关灯 */
    public void off() {
        System.out.println("灯关闭....");
    }
}

/** 电视遥控器 */
public class TvReceiver {

    /** 开电视 */
    public void on() {
        System.out.println("电视打开....");
    }

    /** 关电视 */
    public void off() {
        System.out.println("电视关闭....");
    }
}

```

命令,定义接口,且支持撤销

```java
/** 命令接口 */
public interface Command {

    /** 执行动作 */
    public void execute();

    /** 撤销动作 */
    public void undo();
}
```

具体调用命令, 聚合持有具体的接收执行对象. 这里定义空命令,以便在程序初始化中调用.

```java
/** 空命令,控制性.用于初始化,当调用时,什么也不做 */
public class NoCommand implements Command {
    public void execute() { }

    public void undo() { }
}

/** 灯打开命令,聚合具体的执行者.
 *  并将命令与执行者相关 */
public class LightOnCommand implements Command {

    /** 聚合命令具体执行者 */
    private final LightReceiver lightReceiver;

    public LightOnCommand(LightReceiver lightReceiver) {
        this.lightReceiver = lightReceiver;
    }

    public void execute() {
        lightReceiver.on();
    }

    public void undo() {
        lightReceiver.off();
    }
}
/** 关灯命令 */
public class LightOffCommand implements Command {

    private final LightReceiver lightReceiver;

    public LightOffCommand(LightReceiver lightReceiver) {
        this.lightReceiver = lightReceiver;
    }
    public void execute() {
        lightReceiver.off();
    }

    public void undo() {
        lightReceiver.on();
    }
}

/** 电视开机命令 */
public class TvOnCommand implements Command {

    private final TvReceiver tvReceiver;

    public TvOnCommand(TvReceiver tvReceiver) {
        this.tvReceiver = tvReceiver;
    }

    public void execute() {
        tvReceiver.on();
    }

    public void undo() {
        tvReceiver.off();
    }
}

/** 电视机关命令 */
public class TvOffCommand implements Command {

    private final TvReceiver tvReceiver;

    public TvOffCommand(TvReceiver tvReceiver) {
        this.tvReceiver = tvReceiver;
    }
    public void execute() {
        tvReceiver.off();
    }

    public void undo() {
        tvReceiver.on();
    }
}
```

调用者,遥控器类. 将具体命令进行调用分发.
内部使用数组维护命令组,调用时传入数组索引进行执行; 缓存上一个动作命令,支持撤销

```java
/** 遥控器,持有多组命令 */
public class RemoteController {
	/** 开机命令组 */
    private final Command[] onCommands;
	/** 关机命令组 */
    private final Command[] offCommands;

    /** 撤销的命令 */
    private Command undoCommand;

    /** 构造器初始化按钮组 */
    public RemoteController() {
        onCommands = new Command[5];
        offCommands = new Command[5];

        // 初始化,为空命令. 这里初始化5组命令
        for (int i = 0; i < 5; i++) {
            onCommands[i] = new NoCommand();
            offCommands[i] = new NoCommand();
        }
    }

    /** 为按钮组设置命令 */
    public void setCommand(int index, Command onCommand, Command offCommand){
        onCommands[index] = onCommand;
        offCommands[index] = offCommand;
    }

    /** 对外提供按钮,启动 */
    public void onButtonClick(int index){
        // 找到具体对的家电按钮,并执行
        onCommands[index].execute();
        // 记录操作,以备撤销
        undoCommand = onCommands[index];
    }

    /** 对外提供按钮,关闭 */
    public void offButtonClick(int index){
        // 找到具体对的家电按钮,并执行
        offCommands[index].execute();
        // 记录操作,以备撤销
        undoCommand = offCommands[index];
    }

    /** 对外提供按钮,撤销 */
    public void undoButtonClick(){
        // 执行上次的撤销命令
        undoCommand.undo();
    }
}
```

测试类

```java
public class RemoteControllerTest {

    @Test
    public void test(){
        // 接收者: 电灯
        LightReceiver lightReceiver = new LightReceiver();

        // 命令: 开,关
        LightOnCommand lightOnCommand = new LightOnCommand(lightReceiver);
        LightOffCommand lightOffCommand = new LightOffCommand(lightReceiver);

        // 调用者: 遥控器
        RemoteController remoteController = new RemoteController();
        // 遥控器,命令数组初始化
        remoteController.setCommand(0, lightOnCommand, lightOffCommand);

        // 具体操作... 开..关..撤销
        remoteController.onButtonClick(0);
        remoteController.offButtonClick(0);
        remoteController.undoButtonClick();

        // 接收者: 电视机
        TvReceiver tvReceiver = new TvReceiver();

        // 命令: 开,关
        TvOnCommand tvOnCommand = new TvOnCommand(tvReceiver);
        TvOffCommand tvOffCommand = new TvOffCommand(tvReceiver);

        // 遥控器,命令数组初始化
        remoteController.setCommand(1, tvOnCommand, tvOffCommand);

        // 具体操作... 开..关..撤销
        remoteController.onButtonClick(1);
        remoteController.offButtonClick(1);
        remoteController.undoButtonClick();
    }
}
```



### 示例: JDK中使用
在 `SpringJDBCTemplate` 中用到, 以便提供统一的一组命令,无需关注具体数据操作类型



## 访问者模式
将数据结构与数据操作相分离,在不改变原有数据结构的情况下,定义对这些数据的新操作.
通过在被访问类中,添加对外提供接待访问的接口

角色:
访问者: 抽象访问者,为对象声明访问操作
具体访问: 具体子类,实现访问操作
数据结构: 枚举访问集合,提供高层次接口,允许访问者进行访问 
对象:    抽象元素,接受访问者
具体对象: 具体元素,让访问者操作

场景:
当系统有一个稳定的数据结构,但有经常变化的功能需求
需要某个对象结构的对象进行很多不同没有关联的操作,避免因为操作不当污染原有类

### 双分派

首先在客户端程序中,将具体的状态 `Action` 作为参数传入到 `Woman`中(第一次分派,动态分配 `Action` 的具体子类).
随后在 `Woman` 类中调用参数的具体方法中的 `getWoManResult` , 将自己 `this` 作为参数(第二次分派,动态分配`People` 子类).
达到解耦的目的

### 业务场景

观众对歌手打分,是否晋升/失败/待定....

访问者, 分为抽象类和具体实现类, 定义并实现访问方法

```java
/** 访问者, 定义访问接口 */
public abstract class Action {

    /** 获得一个男歌手的测评 */
    public abstract void getManResult(Man man);

    /** 获得一个女歌手的测评 */
    public abstract void getWoManResult(Woman woman);

}

/** 具体访问者 - 打分成功 */
public class Success extends Action {

    @Override
    public void getManResult(Man man) {
        System.out.println("男歌手晋升成功..." + man.name);
    }

    @Override
    public void getWoManResult(Woman woman) {
        System.out.println("女歌手晋升成功..." + woman.name);
    }
}
/** 具体访问者 - 打分失败 */
public class Fail extends Action {

    @Override
    public void getManResult(Man man) {
        System.out.println("男歌手晋升失败..." + man.name);
    }

    @Override
    public void getWoManResult(Woman woman) {
        System.out.println("女歌手晋升失败..." + woman.name);
    }
}

/** 具体访问者 - 扩展,打分状态. 等待晋升 */
public class Wait extends Action {

    @Override
    public void getManResult(Man man) {
        System.out.println("男歌手等待晋升..." + man.name);
    }

    @Override
    public void getWoManResult(Woman woman) {
        System.out.println("男歌手等待晋升..." + woman.name);
    }
}
```

定义数据层, 分为抽象类及其具体子类

```java
/** 数据,抽象层. 传入访问器,子类调用访问器中的访问方法,获得结果 */
public abstract class People {

    protected String name;

    /** 提供方法,让访问者可以访问 */
    public abstract void accept(Action action);
}

/** 数据, 男歌手 */
public class Man extends People {

    public Man(String name) {
        super();
        super.name = name;
    }

    @Override
    public void accept(Action action) {
        action.getManResult(this);
    }
}

/** 数据, 女歌手 */
public class Woman extends People {

    public Woman(String name) {
        super();
        super.name = name;
    }

    @Override
    public void accept(Action action) {
        action.getWoManResult(this);
    }
}

```

数据结构

```java
/** 维护成员的数据结构 */
public class ObjectStructures {

    /** 维护集合 */
    private final List<People> peoples = new LinkedList<People>();

    /** 增加 */
    public void attach(People people) {
        peoples.add(people);
    }

    /** 移除 */
    public void detach(People people) {
        peoples.remove(people);
    }

    /** 测试 */
    public void dispaly(Action action) {
        for (People people : peoples) {
            people.accept(action);
        }
    }
}
```

测试

```java
/** 访问者模式使用 */
public class ObjectStructuresTest {

    @Test
    public void test(){
        // 对象数据结构
        ObjectStructures context = new ObjectStructures();

        // 添加对象
        context.attach(new Man("男一号"));
        context.attach(new Woman("女一号"));

        // 打分,成功
        Success success = new Success();
        context.dispaly(success);

        // 打分,失败
        Fail fail = new Fail();
        context.dispaly(fail);

        //扩展状态.等待
        Wait wait = new Wait();
        context.dispaly(wait);
    }
}
```

## 迭代器模式

提供统一的迭代器完成遍历,而不用暴露内部数据结构

优点: 提供统一的遍历方法,隐藏内部细节
缺点: 每个集合对象均产生一个迭代器,不好管理.

迭代器接口: 系统提供 `java.util.Iterator`
具体迭代器: 管理迭代过程,实现 `Iterator` 接口
聚合接口: 统一的聚合接口,将客户端和具体迭代器解耦
容器类: 含有具体的容器,持有对象的集合并提供方法返回迭代器



### `Iterator` 接口

在 `java.util.Iterator<T>` 接口中定义了一系列迭代方法.对外提供迭代方法

- `boolean hasNext()` 是否含有下一个成员
- `E next()` 获取下一个成员
- `default void remove()` 迭代时,删除某个元素
- `default void forEachRemaining(Consumer<? super E> action)` 传入消费者接口函数,遍历循环执行

### 业务场景
学校,学院,学系的遍历

定义学系类, 作为迭代器遍历的对象. 

```java
/** 学系,被迭代器遍历的成员 */
public class Department {

    /** 名字 */
    private final String name;

    /** 描述 */
    private final String description;

    public Department(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }
}
```

学院迭代器具体实现,实现接口 `java.util.Iterator`. 

```java
/** 学院迭代器, 迭代内部的学系 内部为数组 */
public class CollegeArrayIterator implements Iterator<Department> {

    /** 迭代的对象 */
    private final Department[] departments;

    /** 遍历的位置 */
    private int position = 0;

    public CollegeArrayIterator(Department[] departments){
        this.departments = departments;
    }

    /** 数组是否含有下一个 */
    @Override
    public boolean hasNext() {
        return position < departments.length && departments[position] != null;
    }

    /** 遍历下一个 */
    @Override
    public Department next() {
        Department department = departments[position];
        position++;
        return department;
    }

    /** 删除方法.不作处理 */
    @Override
    public void remove() {

    }
}

```

学院类,  定义对外提供方法, 其中包括 `iterator` 对外提供迭代器

```java
/** 学院类, */
public interface College {

    /** 获得学院描述 */
    String getName();

    /** 增加学系 */
    void addDepartment(String name, String description);

    /** 返回迭代器,遍历 */
    Iterator<Department> iterator();
}

```

学院具体实现, 通过聚合一个数组,将对象存入.并对外提供迭代器进行访问

```java
/** 计算机学院, 具体的学院. */
public class ComputerCollege implements College{

    /** 聚合,一个数组. */
    Department[] departments;

    /** 保存当前数据组,对象个数 */
    int countOfDepartment = 0;

    public ComputerCollege(){
        departments = new Department[8];
    }

    @Override
    public String getName() {

        return "计算机学院";
    }

    @Override
    public void addDepartment(String name, String description) {
        Department department = new Department(name, description);
        departments[countOfDepartment] = department;
        countOfDepartment++;
    }

    /** 获得迭代器,自定义的迭代器 */
    @Override
    public Iterator<Department> iterator() {
        return new CollegeArrayIterator(departments);
    }
}

```

测试用例

```java
/** 计算机学院, 测试迭代器模式 */
public class ComputerCollegeTest {

    @Test
    public void test(){
        // 学院
        ComputerCollege college = new ComputerCollege();
        // 添加学系
        college.addDepartment("网络工程", "网络工程学系");
        college.addDepartment("软件工程", "软件工程学系");
        college.addDepartment("通信工程", "通信工程学系");

        // 迭代遍历
        Iterator<Department> iterator = college.iterator();
        while (iterator.hasNext()){
            Department next = iterator.next();
            System.out.println(next.getName()  + ": " + next.getDescription());
        }
    }
}
```



### 示例: JDK中使用

`ArrayList` 使用迭代器模式
具体迭代器: `java.util.ArrayList.Itr`: 具体迭代器
聚合接口:   `java.util.List`: 作为聚合集合,提供 `iterator` 迭代方法
容器类:    `java.util.ArrayList`: 容器类,提供具体数组,作为存放容器



## 观察者模式
对象之间一对多的设计模式,一方是被观察对象,多方是观察者.

被观察对象核心方法: 注册/移除/通知
观察者核心方法: 更新

### 业务场景

将当前天气信息主题推送到不同的观察者中.

主题接口,并定义核心方法. 注册 / 移除 / 观察 

```java
/** 主题 */
public interface Subject {

    /** 注册 */
    void registerObserver(Observer observer);

    /** 移除 */
    void removeObserver(Observer observer);

    /** 观察 */
    void notifyObservers(Float temperature);
}
```

观察者接口, 定义更新方法

```java
/** 观察者 */
public interface Observer {

    /** 更新方法,传入温度 */
    void update(Float temperature);
}
```

具体的主题对象, 天气信息. 内部维护了所有的观察者对象列表, 并实现 注册 / 移除 / 观察 相关方法

```java
/** 天气信息,被观察对象 */
public class WeatherData implements Subject{

    /** 所有观察者 */
    List<Observer> observer = new ArrayList<>(4);

    @Override
    public void registerObserver(Observer observer) {
        this.observer.add(observer);
    }

    @Override
    public void removeObserver(Observer observer) {
        this.observer.remove(observer);
    }

    @Override
    public void notifyObservers(Float temperature) {
        observer.forEach(o -> o.update(temperature));
    }
}

```

具体观察者

```java
/** 观察者, 订阅用户天气. */
public class WeatherUser implements Observer{

    /** 观察内容: 天气温度 */
    public void use(Float temperature){
        System.out.println("用户观察天气: " + temperature);
    }

    @Override
    public void update(Float temperature) {
        // 执行内部方法
        use(temperature);
    }
}

/** 观察者, 订阅用户天气.手机 */
public class WeatherMobile implements Observer{

    /** 观察内容: 天气温度 */
    public void use(Float temperature){
        System.out.println("手机观察天气: " + temperature);
    }

    @Override
    public void update(Float temperature) {
        // 执行内部方法
        use(temperature);
    }
}

```

测试用例

```java
/** 测试观察者模式 */
public class ObserverTest {

    @Test
    public void test(){
        // 被观察者对象
        WeatherData weatherData = new WeatherData();

        // 观察者, 用户
        WeatherUser user = new WeatherUser();
        // 观察者, 手机
        WeatherMobile mobile = new WeatherMobile();

        // 注册观察者
        weatherData.registerObserver(user);
        weatherData.registerObserver(mobile);

        // 更新事件
        weatherData.notifyObservers(52F);
    }

}
```



### 示例: JDK使用

`java.util.Observer` 接口,观察者.约定更新方法
`java.util.Observable` 实现类,被观察对象,提供对观察者的一系列方法.如 添加 / 移除  / 通知



## 中介者模式
用一个中介者封装一系列的对象交互,中介者使对象之间无需重复交互引用.
降低子系统模式之间的耦合

角色:
中介接口: 接口,约定同事对象到中介类的关系
具体中介者: 实现接口,并处理同事关系,持有各同事对象
抽象同事: 一系列对象的共有父类,定义相似方法
具体同事类: 一系列对象,每个同事类只了解自身方法.

### 业务场景

不同家电之间调用, 通过中介模式实现.

中介接口,  定义注册同事类的方法, 定义协调不同家电类之间调用的方法. 

```java
/** 中介接口 */
public interface Mediator {

    /** 注册同事类 */
    void register(String name, Colleague colleague);

    /** 核心,处理不同家电的发送消息 */
    void getMessage(int status, String name);

    /** 发送消息 */
    void sendMessage();

}

```

抽象同事类, 内部维护中介类. 并在构造函数中定义中介类,子类在构造函数中,将其自身注入到中介类中.

```java
/** 同事类,定义各子家电共有方法,维持中介类 */
public abstract class Colleague {

    /** 中介类 */
    private final Mediator mediator;

    /** 家电名 */
    public String name;

    public Colleague(Mediator mediator, String name){
        this.mediator = mediator;
        this.name = name;
    }

    public Mediator getMediator() {
        return mediator;
    }

    /** 发送信号 */
    public abstract void sendMessage(int status);
}
```

具体同事类.  在构造函数中,将其自身注入到中介类中.

```java
/** 闹钟,具体同事类 */
public class Alarm extends Colleague {

    public Alarm(Mediator mediator, String name) {
        super(mediator, name);
        // 创建具体同事类时,将自己注入到中介类中.
        mediator.register(name, this);
    }

    /** 发送消息 */
    public void sendAlarm(int status){
        sendMessage(status);
    }

    @Override
    public void sendMessage(int status) {
        // 得到中介,并处理相应消息
        this.getMediator().getMessage(status, this.name);
    }
}

/** 电视,具体同事类 */
public class Tv extends Colleague {

    public Tv(Mediator mediator, String name) {
        super(mediator, name);
        // 创建具体同事类时,将自己注入到中介类中.
        mediator.register(name, this);
    }

    /** 发送消息 */
    public void sendAlarm(int status){
        sendMessage(status);
    }

    @Override
    public void sendMessage(int status) {
        // 得到中介,并处理相应消息
        this.getMediator().getMessage(status, this.name);
    }
}

/** 点灯,具体同事类 */
public class Light extends Colleague {

    public Light(Mediator mediator, String name) {
        super(mediator, name);
        // 创建具体同事类时,将自己注入到中介类中.
        mediator.register(name, this);
    }

    /** 发送消息 */
    public void sendAlarm(int status){
        sendMessage(status);
    }

    @Override
    public void sendMessage(int status) {
        // 得到中介,并处理相应消息
        this.getMediator().getMessage(status, this.name);
    }
}

```

具体的中介类. 提供注册方法, 在具体家电类创建时,将其注入到 `HashMap` 中. 并实现 `getMessage` 方法.来处理不同子系统对某个消息状态的具体响应..

```java
/** 具体的中介者 */
public class ConcreteMediator implements Mediator {

    private final HashMap<String, Colleague> colleagues = new HashMap<>();

    @Override
    public void register(String name, Colleague colleague) {
        colleagues.put(name, colleague);
    }


    /** 核心方法,在该方法中,完成各个子系统间的协调办公 */
    @Override
    public void getMessage(int status, String name) {
        // 根据发出消息的来源不同,处理不同的请求
        if(colleagues.get(name) instanceof Alarm){
            System.out.println("闹钟发出的消息..." + status);
        }else if(colleagues.get(name) instanceof Tv){
            System.out.println("电视发出的消息..." + status);
        }else if(colleagues.get(name) instanceof Light){
            System.out.println("电灯发出的消息..." + status);
        }else{
            System.out.println("其他消息...");
        }
    }

    @Override
    public void sendMessage() { }
}

```

测试

```java
/** 中介者模式测试 */
public class ConcreteMediatorTest {

    @Test
    public void test(){
        // 创建一个中介者
        Mediator mediator = new ConcreteMediator();
        // 创建各个子系统
        Alarm alarm = new Alarm(mediator, "alarm");
        Tv tv = new Tv(mediator, "tv");
        Light light = new Light(mediator, "light");
        // 发送消息
        alarm.sendMessage(1);
        light.sendMessage(2);
        tv.sendMessage(3);
    }
}
```



## 备忘录模式
在不破坏封装的情况下,捕获一个对象的内部状态,并在对象之外保存状态.这样就可以恢复到保存之前的状态
多与原型模式配合使用,减少内存消耗

角色:
目标对象: 普通类,额外提供保存和恢复方法
纪录: 一组存放目标对象属性的类
备忘录: 持有一组纪录类,提供增删方法

使用场景: 存档; 后退一步; 数据库事务

###业务场景

将一个类的状态属性恢复到之前的状态

被保存的目标对象, 要保存其 `status` 属性. 对外提供保存后生成纪录类 `Memento` 和通过纪录类恢复状态的方法

```java
/** 要被保存的对象 */
public class Originator {

    /** 保存内容 */
    public String status;

    /** 保存,到备忘录中 */
    public Memento save(){
        return new Memento(status);
    }

    /** 通过备忘录中,获得对象的状态,并恢复 */
    public void recoverStatusFromMemento(Memento memento){
        this.status = memento.status;
    }
}

```

纪录类,保存 `status` 状态信息, 并在构造函数中引入

```java
/** 状态封装类 */
public class Memento {

    /** 保存对象的属性 */
    public String status;

    Memento(String status){
        this.status = status;
    }
}

```

备忘录, 维护一系列保存纪录类信息. 对外提供增添和获得方法

```java
/** 备忘录集合 */
public class MementoCollect {

    private final List<Memento> mementos = new ArrayList<>(4);

    /** 添加状态 */
    public void add(Memento memento){
        mementos.add(memento);
    }

    /** 获得状态 */
    public Memento get(int index){
        return mementos.get(index);
    }
}
```

测试类. 备忘录模式

```java
/** 备忘录模式 */
public class MementoCollectTest {

    @Test
    public void test(){
        Originator originator = new Originator();

        // 备忘录集合
        MementoCollect collect = new MementoCollect();

        // 状态 1
        originator.status = "状态#1";
        // 保存状态
        collect.add(originator.save());

        // 状态 2
        originator.status = "状态#2";
        collect.add(originator.save());

        // 状态 3
        originator.status = "状态#3";
        collect.add(originator.save());

        // 获得之前状态
        System.out.println(collect.get(2).status);
        // 恢复
        originator.recoverStatusFromMemento(collect.get(2));
    }
}
```



## 解释器模式

定义一个语言表达式,并对其进行解释执行. 如各种表达式解析器

角色:
环境角色: 包含上下文信息.
抽象表达式: 声明抽象解释操作.
终结符表达式: 完成终结符的解释操作.
非终结符表达式: 完成非终结符的解释操作.

## 状态模式
对象多种状态间转换时,对外输出不同的执行.
将某个状态单独定义类,并对其约定的该状态下方法进行编写.
相对于`if-else`判断状态,代码简洁.

角色:
上下文角色: 维护状态信息
抽象状态角色: 定义角色动作,定义上下文中的操作
具体状态角色: 每个状态为一个具体子类,并携带有在改状态下可执行上下文的相关动作



### 业务场景

针对类的初始化 / 工作 / 销毁 状态分别作处理

抽象状态接口, 定义动作方法. 包括 `doInit`, `doWork`, `doDestroy` 分别表示生命周期

```java
/** 状态接口,定义操作 */
public interface State {

    /** 新建状态 */
    default void doInit(){ }

    /** 工作状态 */
    void doWork();

    /** 销毁状态 */
    void doDestroy();
}
```

具体的状态类. 分别实现对应接口

```java
/** 初始化状态 */
public class InitState implements State{
    @Override
    public void doInit() {
        System.out.println("执行初始方法...");
    }

    @Override
    public void doWork() {
        System.out.println("禁止");
    }

    @Override
    public void doDestroy() {
        System.out.println("禁止");
    }
}

/** 工作状态 */
public class WorkState implements State{
    @Override
    public void doInit() {
        System.out.println("禁止");
    }

    @Override
    public void doWork() {
        System.out.println("执行工作方法...");
    }

    @Override
    public void doDestroy() {
        System.out.println("禁止");
    }
}

/** 工作状态 */
public class DestroyState implements State{

    @Override
    public void doInit() {
        System.out.println("禁止");
    }

    @Override
    public void doWork() {
        System.out.println("禁止");
    }

    @Override
    public void doDestroy() {
        System.out.println("执行销毁方法...");
    }
}

```

上下文类,将状态引入,并在构造器中聚合..

```java
/** 各种状态. */
public class Activity {

    /** 状态 */
    public State state;

    public Activity(State state){
        this.state = state;
    }
}
```

测试类.状态模式

```java
/** 状态模式 */
public class ActivityTest {
    @Test
    public void test(){
        // 活动,设置状态
        Activity activity = new Activity(new InitState());
        // 可以调用初始化方法
        activity.state.doInit();
        // 修改新状态,并执行
        activity.state = new WorkState();
        activity.state.doWork();
        // 修改新状态,并执行
        activity.state = new DestroyState();
        activity.state.doDestroy();
        // 尝试使用非当前状态的; 不能使用
        activity.state.doInit();
    }
}
```

## 策略模式
通过定义算法族,分别封装,以便相互替换.
使变化的代码从不变中相分离

角色:
策略接口: 定义策略方法.
策略实现类: 实现具体的策略方法.
抽象目标对象:聚合策略接口,并定义同名策略方法
具体目标对象: 将不同策略方法实现类进行聚合,并调用其策略方法作为抽象类约定的具体实现.

### 业务场景
鸭子模型. 将鸭子的不同特性作为不同的策略,通过聚合具体的策略接口实现类.扩展鸭子的特性.

定义飞行策略, 及其具体实现类.

```java
/** 鸭子飞行行为 */
public interface FlyBehavior {
    /** 飞行 */
    void fly();
}

/** 飞行策略实现, 不会飞 */
public class NoFlyBehavior implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("不会飞翔的鸭子...");
    }
}

/** 飞行策略实现, 擅长飞翔 */
public class GoodFlyBehavior implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("擅长飞翔的鸭子...");
    }
}
```

定义叫声策略, 及其具体实现类

```java
/** 鸭子叫行为 */
public interface QuackBehavior {
    /** 叫 */
    void quack();
}

/** 叫声策略实现, 嘎嘎叫 */
public class GoodQuackBehavior implements QuackBehavior {
    /** 叫 */
    @Override
    public void quack() {
        System.out.println("鸭子嘎嘎叫...");
    }
}
```

抽象目标类, 聚合策略接口和定义策略同名方法. 期待被子类重写

```java
/** 抽象,鸭子类. 定义方法,被子类重写 */
public abstract class Duck {

    /** 飞行策略 */
    protected FlyBehavior flyBehavior;

    /** 叫策略 */
    protected QuackBehavior quackBehavior;

    /** 飞行 */
    protected void fly(){ }

    /** 叫声 */
    protected void quack(){ }
}
```

具体的目标类, 指定具体的策略接口,并调用策略方法

```java
/** 玩具鸭子 */
public class ToyDuck extends Duck {

    /** 在构造器中定义具体策略 */
    public ToyDuck(){
        // 飞行,叫声 行为
        super.flyBehavior = new NoFlyBehavior();
        super.quackBehavior = new GoodQuackBehavior();
    }

    /** 飞行 */
    @Override
    protected void fly() {
        super.flyBehavior.fly();
    }

    /** 叫声 */
    @Override
    protected void quack() {
        super.quackBehavior.quack();
    }
}

/** 野鸭 */
public class WildDuck extends Duck {

    /** 在构造器中定义具体策略 */
    public WildDuck(){
        // 飞行,叫声 行为
        super.flyBehavior = new GoodFlyBehavior();
        super.quackBehavior = new GoodQuackBehavior();
    }

    /** 飞行 */
    @Override
    protected void fly() {
        super.flyBehavior.fly();
    }

    /** 叫声 */
    @Override
    protected void quack() {
        super.quackBehavior.quack();
    }
}
```

测试, 策略模式

```java
/** 测试. 策略模式 */
public class DuckTest {

    @Test
    public void test(){
        WildDuck wildDuck = new WildDuck();
        wildDuck.fly();
        wildDuck.quack();

        ToyDuck toyDuck = new ToyDuck();
        toyDuck.fly();
        toyDuck.quack();
        // 动态修改行为
        toyDuck.flyBehavior=new GoodFlyBehavior();
        toyDuck.fly();
    }
}
```

### JDK应用
在 `java.util.Comparator` 接口中实现排序策略方法 `int compare(T o1, T o2)` 
在 `java.util.Arrays` 中排序方法 `sort(T[], java.util.Comparator<? super T>)` 方法中传入具体策略实现, 完成排序调用.




## 责任链模式

为请求创建一个接受请求的链.
通常每个接受者都包含对另一个接受者的引用,如果不能处理,则交给下一个接受者处理.
常用于处理多级请求

角色:
抽象处理者: 定义处理接口,关联自身.
具体处理者: 实现处理接口,并关联其他具体处理者.
请求者: 具体的请求动作

### 业务场景
采购金额审批.
金额小于100, 员工审批, 小于10000, 经理审批, 剩余老板审批.

定义处理对象, 商品订单

```java
/** 商品订单 */
public class GoodOrder {

    /** 商品 */
    private final String name;

    /** 价格 */
    private final Integer price;

    public GoodOrder(String name, Integer price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public Integer getPrice() {
        return price;
    }

    @Override
    public String toString() {
        return "GoodOrder{" +
                "name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```

抽象处理者, 定义抽象处理过程 `process` 进行审批动作. 并维护下一个处理处理者 `other` .

```java
/** 抽象处理者 */
public abstract class Approver {

    /** 处理过程 */
    protected abstract void process(GoodOrder goodOrder);

    /** 关联其他处理者 */
    private Approver other;

    /** 如果处理不了,交由其他审批人处理.环状结构调用 */
    public void setOther(Approver approver){
        this.other = approver;
    }

    /** 获取其他请求处理人 */
    public Approver getOther() {
        return other;
    }
}
```

具体处理者. 在处理过程 `process` 中定义具体的审批过程,如果条件不满足, 交给下一个

```java
/** 员工处理 */
public class EmployeeApprover extends Approver {

    /** 处理过程 */
    @Override
    protected void process(GoodOrder goodOrder) {
        if(goodOrder.getPrice() <= 100){
            System.out.println("金额小于100, 员工审批..." + goodOrder);
        }else{
            System.out.println("员工审批不了...");
            // 否则,交给其他审批人处理
            super.setOther(new ManagerApprover());
            super.getOther().process(goodOrder);
        }
    }
}

/** 经理处理 */
public class ManagerApprover extends Approver {

    /** 处理过程 */
    @Override
    protected void process(GoodOrder goodOrder) {
        if(goodOrder.getPrice() <= 10000){
            System.out.println("金额小于1万, 经理审批..." + goodOrder);
        }else{
            System.out.println("经理审批不了...");
            // 否则,交给其他审批人处理
            super.setOther(new BossApprover());
            super.getOther().process(goodOrder);
        }
    }
}
/** 老板审批人,最终处理 */
public class BossApprover extends Approver {

    /** 处理过程 */
    @Override
    protected void process(GoodOrder goodOrder) {
        System.out.println("最终Boss审批..." + goodOrder);
    }
}

```

测试类.责任链模式

```java
/** 责任链模式 */
public class ApproverTest {

    @Test
    public void test(){
        GoodOrder good = new GoodOrder("电脑", 40000);
        // 处理者, 链模式
        EmployeeApprover employeeApprover = new EmployeeApprover();
        // 尝试用员工审批
        employeeApprover.process(good);
    }
}
```




### 应用
Spring中的拦截器处理条.
`org.springframework.web.servlet.HandlerInterceptor` 拦截器,通过实现相关方法,作为具体的处理.
顺序根据`Order`而定
`org.springframework.web.servlet.DispatcherServlet#doDispatch`执行具体的请求时,调用相关拦截器