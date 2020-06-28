---
title: Spring全家桶-SpringBoot源码篇
date: 2020-06-23 22:43:34
categories:
  - Java
  - Spring
tags: [Java, Spring]
---

# 简介

介绍SpringBoot的启动原理,场景启动器,Bean管理等...



## 环境信息

- JDK14
- spring-boot 2.3.X
- idea



# 启动流程

1. 框架初始化
2. 框架启动
3. 自动化装配



## 框架初始化

1. 配置资源加载器
2. 配置 `primarySources`
3. 应用环境监测
4. 配置系统初始化器
5. 配置应用监听器
6. 配置 `main` 方法所在类



## 启动框架

1. 计时器启动
2. `Headless` 模式赋值
3. 发送 `ApplicationStartingEven` 事件
4. 配置环境模块
5. 发送 `ApplicationEnvironmentPreparedEvent` 事件
6. 打印 `banner`
7. 创建应用上下文对象
8. 初始化失败分析器
9. 关联SpringBoot组件与应用上下文对象
10. 发送 `ApplicationContextInitializedEvent` 事件
11. 加载 `source` 到 `context`
12. 发送 `ApplicationPreparedEvent` 事件
13. 刷新上下文
14. 计时器停止计数
15. 发送 `ApplicationStartedEvent` 事件
16. 调用框架启动扩展类
17. 发送 `ApplicationReadyEvent` 事件



## 框架自动化装配步骤

1. 收集配置文件中的配置工厂类
2. 加载组件工厂
3. 注册组件内定义的 `Bean`



# 框架初始化

## 系统初始化器

接口 `org.springframework.context.ApplicationContextInitializer`
在Spring容器刷新 `refresh` 方法 前进行函数回调, 以便向容器中注入属性.
通过继承接口并各自实现.



### 作用

- 上下文刷新 `refresh` 方法前调用
- 编码设置一些属性变量.通常在 web 环境中
- 通过 `order` 接口进行排序



### 示例 - 自定义初始化器

其中, `@Order` 注解中的值越小,则越先执行



####  工厂文件注册

首先通过实现 `org.springframework.context.ApplicationContextInitializer` 接口中的初始化方法,随后在项目创建 `META-INF/spring.factories` 文件, 注册初始化器

**实现接口**

```java
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.annotation.Order;
import org.springframework.core.env.ConfigurableEnvironment;

/**
 *  自定义容器启动器. 通过 META-INF/spring.factories 注册
 *      在容器注入时,进行操作
 * @author Jion
 */
@Order(1)
public class WebApplicationInitializerFirst implements ApplicationContextInitializer<ConfigurableApplicationContext> {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        // 获得容器中环境变量
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        System.out.println("--- 方法一 ---");
        System.out.println("容器初始化器启动...");
        System.out.println("------");
    }
}
```

**注册初始化器**

```properties
# 注册自定义系统初始化器
org.springframework.context.ApplicationContextInitializer=top.jionjion.initializer.WebApplicationInitializerFirst
```



#### 启动类注册

实现 `org.springframework.context.ApplicationContextInitializer` 接口,并在启动类中注册初始化器

**实现接口**

```java
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.annotation.Order;
import org.springframework.core.env.ConfigurableEnvironment;

/**
 *  自定义容器启动器. 通过启动类注册
 *      在容器注入时,进行操作
 * @author Jion
 */
@Order(2)
public class WebApplicationInitializerSecond implements ApplicationContextInitializer<ConfigurableApplicationContext> {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        // 获得容器中环境变量
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        System.out.println("--- 方法二 ---");
        System.out.println("容器初始化器启动...");
        System.out.println("------");
    }
}
```

**注册启动器**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContextInitializer;
import top.jionjion.initializer.WebAppInitializerSecond;

import java.util.Collections;

/**
 *  主启动类
 * @author 14345
 */
@SpringBootApplication
public class WebApplication {

    public static void main(String[] args) {
        // 正常的启动方式
        //SpringApplication.run(WebApplication.class, args);

        // 自定义容器的初始化器,并调用run方法
        SpringApplication springApplication = new SpringApplication(WebApplication.class);
        springApplication.setInitializers(Collections.singleton(new WebApplicationInitializerSecond()));
        springApplication.run(args);
    }
}
```



#### 配置文件注册

**实现接口**

```java
package top.jionjion.initializer;

import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.annotation.Order;
import org.springframework.core.env.ConfigurableEnvironment;

/**
 *  自定义容器启动器. 通过属性 context.initializer.classes 指定调用
 *      在容器注入时,进行操作
 * @author Jion
 */
@Order(3)
public class WebApplicationInitializerThird implements ApplicationContextInitializer<ConfigurableApplicationContext> {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        // 获得容器中环境变量
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        System.out.println("--- 方法三 ---");
        System.out.println("容器初始化器启动...");
        System.out.println("------");
    }
}
```

**配置文件**

```yaml
context:
  initializer:
    classes: top.jionjion.initializer.WebApplicationInitializerThird
```



### 初始化器调用位置

在 `run` 方法中, 调用 `prepareContext(context, environment, listeners, applicationArguments, printedBanner)` 方法.

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        configureIgnoreBeanInfo(environment);
        Banner printedBanner = printBanner(environment);
        context = createApplicationContext();
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                                                         new Class[] { ConfigurableApplicationContext.class }, context);
        // 准备容器 
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        refreshContext(context);
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

在其中调用 `applyInitializers(context) ` 调用初始化容器方法

```java
private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
                            SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
    context.setEnvironment(environment);
    postProcessApplicationContext(context);
    // 初始化容器
    applyInitializers(context);
    listeners.contextPrepared(context);
    if (this.logStartupInfo) {
        logStartupInfo(context.getParent() == null);
        logStartupProfileInfo(context);
    }
    // Add boot specific singleton beans
    ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
    beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
    if (printedBanner != null) {
        beanFactory.registerSingleton("springBootBanner", printedBanner);
    }
    if (beanFactory instanceof DefaultListableBeanFactory) {
        ((DefaultListableBeanFactory) beanFactory)
        .setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
    }
    if (this.lazyInitialization) {
        context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
    }
    // Load the sources
    Set<Object> sources = getAllSources();
    Assert.notEmpty(sources, "Sources must not be empty");
    load(context, sources.toArray(new Object[0]));
    listeners.contextLoaded(context);
}
```

这里, 断言进行判断.只有在当前容器为泛型中定义容器子类的对象才有必要进行初始化
如: `implements ApplicationContextInitializer<ConfigurableApplicationContext>`

```java
protected void applyInitializers(ConfigurableApplicationContext context) {
    for (ApplicationContextInitializer initializer : getInitializers()) {
        // 断言,只有该容器对象类为其泛型的子类时,才能进行初始化操作
        Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(),
                                                                        ApplicationContextInitializer.class);
        Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
        initializer.initialize(context);
    }
}
```

#### 扩展

定义在 `spring.factories` 文件中的初始化器被 `SpringFactoriesLoader` 发现并注册

`SpringApplication` 初始化后再手动添加注册

通过备配置文件注册的初始化器,会由 `org.springframework.boot.context.config.DelegatingApplicationContextInitializer` 类进行注册调用, 该类由Spring加载, 其配置优先级最高,会使其属性文件 `context.initializer.classes` 定义的初始化器最先调用.不受 `Order` 排序影响.



### 加载步骤

1. 判断缓存中是否存在,如果存在则返回.否则继续
2. 读取指定资源文件
3. 构造 `properties` 对象
4. 获取指定 `key` 对应的 `value` 值
5. 逗号分隔到 `value`
6. 保存结果到缓存
7. 依次实例化到结果对象
8. 对结果进行排序
9. 返回实例结果



### 扩展 - `SpringFactoriesLoader `

Spring 提供的通用工厂加载机制.
在 `META-INF/spring.factories`  从 `classpath` 下多个 `jar` 内的特定位置读取文件并初始化类.
其配置文件必须为 `properties` 文件, `key` 为全限定名, `value` 为具体的实现,可以有多个,并用逗号分隔.



### 源码步骤

#### 0-0-0-0 启动入口

通过传入主启动类进行启动.调用 `run` 方法

```java
@SpringBootApplication
public class WebApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }
}
```



#### 1-1-0-0 `run` 方法调用前

在静态方法中, 传入主启动类和系统参数,调用静态方法 `run`

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class<?>[] { primarySource }, args);
}
```

在静态 `run` 方法中,创建容器 `SpringApplication` 对象, 并调用实例 `run` 方法

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```

#### 1-2-0-0 初始化 `SpringApplication`

带 `Class` 参数的构造器, 传入 `primarySources` , 即主启动类, 调用构造器

```java
public SpringApplication(Class<?>... primarySources) {
    this(null, primarySources);
}
```

带 `ResourceLoader` 资源加载器和 `primarySources` 的构造器.其中完成了 `SpringApplication` 的对象初始化.

1. 存放资源加载器 `ResourceLoader`
2. 判断是否为 `WEB` 环境
3. 加载初始化器
4. 加载监听器
5. 推断主要启动类

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    // 1. 资源加载器, 一般为空
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    // 2. 主启动类
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    // 3. 判断是否为WEB环境
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    // 4. 加载,初始化器
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    // 5. 加载,监听器
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    // 6. 推断主要启动类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

####1-2-4-0 加载容器初始化器

通过传入 `org.springframework.context.ApplicationContextInitializer` 的接口类型,加载其具体实现

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}
```

这里主要有三步

1. 使用加载器加载具体实现类的全限定名
2. 创建工厂具体实例对象
3. 对其进行排序. 如使用 @Order 注解

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    // 类加载器
    ClassLoader classLoader = getClassLoader();
    // 1. 使用加载器加载具体实现类的全限定名
    Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 2. 创建加载器加载的具体实例对象
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
    // 3. 对其进行排序. 如使用 @Order 注解
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

#### 1-2-4-1 获取加载器的具体实现类的全限定名

通过 `SpringFactoriesLoader.loadFactoryNames(type, classLoader)` 方法,获得Spring的 `META-INF/spring.factories` 中定义的接口和实现类.
在使用 `getOrDefault(factoryTypeName, Collections.emptyList())` 传入具体要获得加载的类的全限定名 `factoryTypeName` 如果为空,则返回空列表.

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    String factoryTypeName = factoryType.getName();
    return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
}
```

#### - - - - 获得工厂类及其实例

具体的加载原理如下.
首先尝试从缓存中加载,如果能获得到需要的接口及其实现类,则返回结束.

从当前项目/系统目录中的各个 Jar 包中,获取 `META-INF/spring.factories` 文件,其为属性文件.进行读取; 
将其封装为 `Properties` 对象,并对每个 `Properties` 对象进行遍历;
属性对象中的 `key` 为各种接口的全限定名,  `value` 为各个接口的具体的实现类,如果有多个则进行逗号拼接.

读取属性文件,并将其转为 `Map<String, List<String>>` 结构. `key` 为各种接口的全限定名, `value` 为具体实现类的全限定名的列表.

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    // 尝试从缓存中加载,如果能获得到需要的接口及其实现类,则返回结束
    MultiValueMap<String, String> result = cache.get(classLoader);
    if (result != null) {
        return result;
    }

    try {
        Enumeration<URL> urls = (classLoader != null ?
                                 // 获得 META-INF/spring.factories 文件位置
                                 classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
                                 ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        result = new LinkedMultiValueMap<>();
        // 加载,读取
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            UrlResource resource = new UrlResource(url);
            // 封装为 Properties
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);
            for (Map.Entry<?, ?> entry : properties.entrySet()) {
                // 获得 key , 接口全限定名
                String factoryTypeName = ((String) entry.getKey()).trim();
                // 对 value 进行逗号拆分, 实现类全限定名
                for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                    // 放入结果,并返回
                    result.add(factoryTypeName, factoryImplementationName.trim());
                }
            }
        }
        cache.put(classLoader, result);
        return result;
    }
    catch (IOException ex) {
        // 如果 META-INF/spring.factories 文件加载失败,抛出异常
        throw new IllegalArgumentException("Unable to load factories from location [" +
                                           FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}
```

#### 1-2-4-2 创建工厂类的实例对象

通过 `names` 传入 `Set<String>` 集合, 集合中包括了要创建的实例对象的全限定名. 调用 `BeanUtils.instantiateClas` 方法传入类的构造器及参数,创建实例对象.

```java
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
                                                   ClassLoader classLoader, Object[] args, Set<String> names) {
    // 默认单利模式,因此传入多少类,创建多少实例
    List<T> instances = new ArrayList<>(names.size());
    for (String name : names) {
        try {
            // 获得 Class 信息
            Class<?> instanceClass = ClassUtils.forName(name, classLoader);
            Assert.isAssignable(type, instanceClass);
            // 获得 Constructor 构造器对象
            Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
            // 调用构造器及其参数 创建对象实例并返回
            T instance = (T) BeanUtils.instantiateClass(constructor, args);
            instances.add(instance);
        }
        catch (Throwable ex) {
            throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
        }
    }
    return instances;
}

```

#### 1-2-4-3 对加载器对象进行排序

使用 `AnnotationAwareOrderComparator.sort(instances)` 方法对 `List` 中的实例进行排序



## 监听/广播器

### 监听器/广播模式 

- 事件
- 监听器 
-  广播器:
-  触发机制



### 系统监听器

接口 `org.springframework.context.ApplicationListener` Spring提供的监听器类,用以监听事件.
其继承自 `java.util.EventListener` 标准监听器;
泛型中定义的事件同样继承自 `org.springframework.context.ApplicationEvent` 进行筛选事件.
`void onApplicationEvent(E event)` 当事件发生时,进行处理

```java
package org.springframework.context;

import java.util.EventListener;

@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

	/**
	 * 事件发生时进行处理
	 * @param 广播事件
	 */
	void onApplicationEvent(E event);

}

```



### 系统广播器

接口 `org.springframework.context.event.ApplicationEventMulticaster`  
其定义了添加/删除监听器的方法和广播方法

```java
public interface ApplicationEventMulticaster {

    // 添加监听器
	void addApplicationListener(ApplicationListener<?> listener);

    // 添加监听器
	void addApplicationListenerBean(String listenerBeanName);
	
    // 移除监听器
	void removeApplicationListener(ApplicationListener<?> listener);

    // 移除监听器
	void removeApplicationListenerBean(String listenerBeanName);

    // 移除所有监听器
	void removeAllListeners();

    // 广播事件
	void multicastEvent(ApplicationEvent event);

    // 广播事件给特定的监听器
	void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType);
}

```



### 示例 - 自定义监听器

通过实现 `org.springframework.context.ApplicationListener<T>` 接口,对单一事件进行监听



#### 工厂文件注册

通过实现 `org.springframework.context.ApplicationListener<T>` 接口,对泛型中的时间进行监听.并在 `META-INF/spring.factories` 中配置监听器,监听容器事件.

**实现接口**

```java
import org.springframework.boot.context.event.ApplicationStartedEvent;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.core.annotation.Order;

/**
 *  自定义实现监听器, 在 META-INF/spring.factories 中注册
 *      泛型指定其感兴趣的事件
 * @author Jion
 */
@Order(1)
public class WebApplicationListenerFirst implements ApplicationListener<ApplicationStartedEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("--- 方法一 ---");
        System.out.println("监听器: Spring 准备启动..");
        System.out.println("------");
    }
}
```

**注册初始化器**

```properties
# 注册自定义系统监听器
org.springframework.context.ApplicationListener=top.jionjion.listener.WebApplicationListenerFirst
```



#### 启动类注册

通过实现 `org.springframework.context.ApplicationListener<T>` 接口,对泛型中的时间进行监听.并在主启动类中,通过 `addListeners()` 方法注册监听器.

**实现接口**

```java
import org.springframework.boot.context.event.ApplicationStartedEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.core.annotation.Order;

/**
 *  自定义实现监听器, 通过启动类注册
 *      泛型指定其感兴趣的事件
 * @author Jion
 */
@Order(2)
public class WebApplicationListenerSecond implements ApplicationListener<ApplicationStartedEvent> {
 
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("--- 方法二 ---");
        System.out.println("监听器: Spring 准备启动..");
        System.out.println("------");
    }
}
```

**注册启动器**

```java
@SpringBootApplication
public class CoreWebApplication{
    public static void main(String[] args) {
        // 自定义容器的初始化器,并调用run方法
        SpringApplication springApplication = new SpringApplication(CoreWebApplication.class);
        // 监听器
        springApplication.addListeners(new WebApplicationListenerSecond());
        springApplication.run(args);
    }
}
```



#### 配置文件注册

通过实现 `org.springframework.context.ApplicationListener<T>` 接口,对泛型中的时间进行监听;并在`application.yml` 中配置属性 `context.listener.classes` 指定配置监听器.优先级最高.

**实现接口**

```java
import org.springframework.boot.context.event.ApplicationStartedEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.core.annotation.Order;

/**
 *  自定义实现监听器, 在 CoreWebApplication 中注册
 *      泛型指定其感兴趣的事件
 * @author Jion
 */
@Order(3)
public class WebApplicationListenerThird implements ApplicationListener<ApplicationStartedEvent> {
    
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("--- 方法三 ---");
        System.out.println("监听器: Spring 准备启动..");
        System.out.println("------");
    }
}

```

**配置文件**

```yaml
# 容器配置信息
context:
  listener:
    classes: top.jionjion.listener.WebApplicationListenerThird
```



#### 扩展 - 其他接口实现

通过实现 `org.springframework.context.event.SmartApplicationListener`
可以对多个事件进行监听, 注册方式与之前相同.

```java
import org.springframework.boot.context.event.ApplicationStartedEvent;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.event.SmartApplicationListener;
import org.springframework.core.annotation.Order;

/**
 *  自定义实现监听器
 *      泛型指定其感兴趣的事件
 * @author Jion
 */
@Order(4)
public class WebApplicationListenerFourth implements SmartApplicationListener {

    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> eventType) {
        // 感兴趣的事件  ApplicationStartedEvent 事件
        return  ApplicationStartedEvent.class.isAssignableFrom(eventType);
    }

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        // 监听执行时间
        System.out.println("--- 方法四 ---");
        System.out.println("监听器: Spring 准备启动..");
        System.out.println("------");
    }
}
```



### 监听器调用位置

在 SpringBoot 启动的不同节点,会进行不同的广播. 相关监听器在注册到容器中后,如果收到相关事件且感兴趣,会执行其对应的方法.
容器事件参考 `org.springframework.boot.context.event.SpringApplicationEvent` 类及其子类.
容器监听器,参考配置文件

```java

```

#### 扩展

定义在 `spring.factories` 文件中的监听器被 `SpringFactoriesLoader` 发现并注册

`SpringApplication` 初始化后再手动添加监听器

通过备配置文件注册的监听器,会由 `org.springframework.boot.context.config.DelegatingApplicationListener`  类进行注册调用, 该类由Spring加载, 其配置优先级最高,会使其属性文件 `context.listener.classes` 定义的初始化器最先调用.不受 `Order` 排序影响.



### 系统事件

Spring容器提供的各种事件. 
首先继承自 `java.util.EventObject` 事件类的抽象类`org.springframework.context.ApplicationEvent` , Spring再对此类作继承,提供一系列子类对象. 其中跟容器启动相关的为 `org.springframework.boot.context.event.SpringApplicationEvent` 抽象类, 其子类具体定义了各种事件.

#### 具体事件顺序

1.  `ApplicationStartingEvent` 框架启动事件, 启动时发送
2.  `ApplicationEnvironmentPreparedEvent` 环境准备完成事件, 已经加载系统配置属性
3.  `ApplicationContextInitializedEvent` 容器初始化完成事件,  `SpringBoot` 中已经准备好上下文容器,尚未装在`Bean` 对象
4.  `ApplicationPreparedEvent` 容器准备完成事件, 上下文对象容器完成,尚未完全加载完 `Bean`
5.  `ApplicationStartedEvent` 容器启动完成事件,  `Bean` 已经装载完成,尚未调用 `run` 扩展方法
6.  `ApplicationReadyEvent` 容器准备完成事件
7.  `ApplicationFailedEvent` 容器失败事件

![系统事件](./Spring全家桶-SpringBoot源码篇/系统事件-1.png)



### 加载步骤

1. 获得监听器列表
2. 尝试从缓存中读取监听器列表,不存在则计算
3. 遍历监听器,  从中找到对当前事件及触发节点感兴趣的监听器
4. 加入感兴趣的监听器列表,排序后依次执行 监听器 .



### 源码步骤

#### 1-2-5-0 加载容器监听器

与系统初始化器相同,在创建 `SpringApplication` 对象时, 通过 `SpringFactoriesLoader` 从 `META-INF/spring.factories` 中加载需要的监听器类. 并创建具体实例.

#### 2-0-0-0 `run` 方法调用

静态方法, 在创建 `SpringApplication` 后, 执行 `run` 方法

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```

具体 `run` 如下.

```java
public ConfigurableApplicationContext run(String... args) {
    // 1. 启用秒表,并开始计时
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // 2. 准备可配置的容器上下文对象
    ConfigurableApplicationContext context = null;
    // 3. 异常报告列表, 失败分析器
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    // 4. 配置 Headless 模式赋值
    configureHeadlessProperty();
    // 5. 获得已经通过 类工厂加载器 加载的 SpringApplicationRunListener 监听器, 具体为 EventPublishingRunListener
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 监听器启动, EventPublishingRunListener 发布 ApplicationStartingEvent 事件
    listeners.starting();
    try {
        // 6. 配置环境信息
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 发布 ApplicationEnvironmentPreparedEvent 时间
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        configureIgnoreBeanInfo(environment);
        // 7. 打印 banner
        Banner printedBanner = printBanner(environment);
        // 8. 创建应用上下文对象
        context = createApplicationContext();
        // 9. 初始化失败分析器
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                                                         new Class[] { ConfigurableApplicationContext.class }, context);
        // 10. 关联SpringBoot组件与应用上下文对象
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        // 11. 刷新上下文
        refreshContext(context);
        // 12. 刷新完成
        afterRefresh(context, applicationArguments);
        // 13. 计时器停止计数
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        // 14. 容器启动完成事件
        listeners.started(context);
        // 15. 调用框架启动扩展类
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```

#### 2-5-0-0 监听器启动

```java
// 1. 获得监听器启动对象
SpringApplicationRunListeners listeners = getRunListeners(args);
// 2. 启动监听器
listeners.starting();
```

#### 2-5-2-0 启动监听器

调用 `org.springframework.boot.SpringApplicationRunListener` 中的 `starting()` 方法

```java
void starting() {
    for (SpringApplicationRunListener listener : this.listeners) {        
        listener.starting();
    }
}
```

其接口实现类为 `org.springframework.boot.context.event.EventPublishingRunListener` ,其调用内部广播器`SimpleApplicationEventMulticaster` 发布一个 `ApplicationStartingEvent` 事件

```java
@Override
public void starting() {
    this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
}
```

**当然, `EventPublishingRunListener` 还负责发布跟容器启动的其他相关事件**

#### 2-5-3-1 创建事件

在 `SimpleApplicationEventMulticaster` 广播器创建事件时,创建了一个 `ApplicationStartingEvent`. 传入当前容器和启动参数.

#### 2-5-3-2 发布事件

发布事件,传入 `ApplicationStartingEvent` 事件, 并对其 `Class` 进行包装 `resolveDefaultEventType` 

```java
@Override
public void multicastEvent(ApplicationEvent event) {
    multicastEvent(event, resolveDefaultEventType(event));
}
```

具体方法为 

```java
@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
    // 获得事件类型
    ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
    // 获得事件的执行者. null
    Executor executor = getTaskExecutor();
    // 获得对当前事件该兴趣的监听器列表.循环调用
    for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
        if (executor != null) {
            // 如果存在事件执行者,则事件执行者调用. 触发监听器
            executor.execute(() -> invokeListener(listener, event));
        }
        else {
            // 触发监听器
            invokeListener(listener, event);
        }
    }
}
```

#### 2-5-3-3 获得对当前事件感兴趣的监听器

计算获得对当前事件感兴趣的监听器,并返回

```java
protected Collection<ApplicationListener<?>> getApplicationListeners(
    ApplicationEvent event, ResolvableType eventType) {
	// 获得 SpringApplication 及其对应的 Class
    Object source = event.getSource();
    Class<?> sourceType = (source != null ? source.getClass() : null);
    // 计算 hash码
    ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);
		
    // 尝试从缓存中,获取感兴趣的 key 及其 调用来源
    ListenerRetriever retriever = this.retrieverCache.get(cacheKey);
    if (retriever != null) {
        // 存在缓存,则返回
        return retriever.getApplicationListeners();
    }
	
    // 不存在缓存
    if (this.beanClassLoader == null ||
        (ClassUtils.isCacheSafe(event.getClass(), this.beanClassLoader) &&
         (sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
        // 其他线程是否完成计算,并存入缓存
        synchronized (this.retrievalMutex) {
            // 是否有其他线程完成计算
            retriever = this.retrieverCache.get(cacheKey);
            if (retriever != null) {
                return retriever.getApplicationListeners();
            }
            // 没有,自行计算
            retriever = new ListenerRetriever(true);
            // 调用计算逻辑
            Collection<ApplicationListener<?>> listeners =
                retrieveApplicationListeners(eventType, sourceType, retriever);
            // 将计算结果存入缓存
            this.retrieverCache.put(cacheKey, retriever);
            // 返回感兴趣的监听器
            return listeners;
        }
    }
    else {
        // 没有对事件感兴趣的
        return retrieveApplicationListeners(eventType, sourceType, null);
    }
}
```



#### 2-5-3-4 计算感兴趣的监听器

遍历当前监听器,调用 `supportsEvent` 方法,判断其是否为对该事件感兴趣的监听器

```java
private Collection<ApplicationListener<?>> retrieveApplicationListeners(
    ResolvableType eventType, @Nullable Class<?> sourceType, @Nullable ListenerRetriever retriever) {
	// 返回结果
    List<ApplicationListener<?>> allListeners = new ArrayList<>();
    // 所有的监听器
    Set<ApplicationListener<?>> listeners;
    // 所有的监听器 Bean
    Set<String> listenerBeans;
    synchronized (this.retrievalMutex) {
        listeners = new LinkedHashSet<>(this.defaultRetriever.applicationListeners);
        listenerBeans = new LinkedHashSet<>(this.defaultRetriever.applicationListenerBeans);
    }

    // 获得对当前事件感兴趣的监听器 类
    for (ApplicationListener<?> listener : listeners) {
        if (supportsEvent(listener, eventType, sourceType)) {
            if (retriever != null) {
                retriever.applicationListeners.add(listener);
            }
            allListeners.add(listener);
        }
    }

    // 获得对当前事件感兴趣的监听器 Bean
    if (!listenerBeans.isEmpty()) {
        ConfigurableBeanFactory beanFactory = getBeanFactory();
        for (String listenerBeanName : listenerBeans) {
            try {
                if (supportsEvent(beanFactory, listenerBeanName, eventType)) {
                    ApplicationListener<?> listener =
                        beanFactory.getBean(listenerBeanName, ApplicationListener.class);
                    if (!allListeners.contains(listener) && supportsEvent(listener, eventType, sourceType)) {
                        if (retriever != null) {
                            if (beanFactory.isSingleton(listenerBeanName)) {
                                retriever.applicationListeners.add(listener);
                            }
                            else {
                                retriever.applicationListenerBeans.add(listenerBeanName);
                            }
                        }
                        allListeners.add(listener);
                    }
                }
                else {
                    // Remove non-matching listeners that originally came from
                    // ApplicationListenerDetector, possibly ruled out by additional
                    // BeanDefinition metadata (e.g. factory method generics) above.
                    Object listener = beanFactory.getSingleton(listenerBeanName);
                    if (retriever != null) {
                        retriever.applicationListeners.remove(listener);
                    }
                    allListeners.remove(listener);
                }
            }
            catch (NoSuchBeanDefinitionException ex) {
                // Singleton listener instance (without backing bean definition) disappeared -
                // probably in the middle of the destruction phase
            }
        }
    }

    // 监听器,根据 Order 值进行排序
    AnnotationAwareOrderComparator.sort(allListeners);
    if (retriever != null && retriever.applicationListenerBeans.isEmpty()) {
        retriever.applicationListeners.clear();
        retriever.applicationListeners.addAll(allListeners);
    }
    // 返回结果
    return allListeners;
}

```

具体判断逻辑如下

```java
protected boolean supportsEvent(
    ApplicationListener<?> listener, ResolvableType eventType, @Nullable Class<?> sourceType) {

    GenericApplicationListener smartListener = (listener instanceof GenericApplicationListener ?
                                                (GenericApplicationListener) listener : new GenericApplicationListenerAdapter(listener));
    return (smartListener.supportsEventType(eventType) && smartListener.supportsSourceType(sourceType));
}
```



# 启动框架

## Bean管理

### `IOC` 思想

将类的初始化过程交由容器完成,使用时只用向容器申请相应对象,容器会自动完成创建.

- 松耦合
- 灵活
- 可维护性提高

XML配置

- 低耦合
- 对象关系清晰
- 集中管理
- 配置繁琐
- 效率低
- 文件解析耗时

注解配置

- 使用简单
- 效率高
- 高内聚
- 配置分散
- 对象关系不明细
- 修改后需重新编译

### `XML` 方式

- 属性注入
- 构造器注入
- 静态工厂注入
- 实例工厂注入



#### 属性注入

定义对象 `Dog` 类

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 *  使用 xml 完成 IOC
 * @author Jion
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dog {

    private String name;

    private Integer age;
}
```

在配置文件中,注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 属性构造注入 -->
    <bean id="dog" class="top.jionjion.ioc.xml.Dog">
        <property name="name" value="大黄"/>
        <property name="age" value="3"/>
    </bean>
</beans>
```

测试用例

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class BeanConfigTest {

    @Autowired
    Dog dog;

    @Test
    public void test() {
        System.out.println("获得Bean:" + dog);
    }
}
```



#### 构造器注入

添加引用类 `DogHouse` 引用类 `Dog`  

```java
import lombok.Getter;
import lombok.Setter;

/**
 *  引用
 * @author Jion
 */
public class DogHouse {

    /** 引用 */
    @Getter
    @Setter
    private Dog dog;
}
```

在配置文件中,配置构造器注入和引用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- 有参构造器注入 -->
    <bean id="dog" class="top.jionjion.ioc.xml.Dog">
        <constructor-arg index="0" value="二黄"/>
        <constructor-arg index="1" value="1"/>
    </bean>

    <!-- 使用Bean -->
    <bean id="dogHouse" class="top.jionjion.ioc.xml.DogHouse">
        <!-- 引用 -->
        <property name="dog" ref="dog"/>
    </bean>
</beans>    
```

测试用例

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class DogHouseTest {

    @Autowired
    DogHouse dogHouse;

    @Test
    public void test(){
        Dog dog = dogHouse.getDog();
        System.out.println("获得Bean:" + dogHouse);
    }
}
```



#### 静态工厂注入

创建抽象父类.

```java
/**
 *  抽象父类
 * @author Jion
 */
public abstract class Animal {

    /**
     *  获得动物名称
     * @return 名称
     */
     abstract String getName();
}
```

及其子类

```java
public class Cat extends Animal {
    @Override
    String getName() {
        return "我是猫... ";
    }
}
```

静态工厂类

```java
public class StaticAnimalFactory {
    public static Animal getAnimal(String type){
        // 工厂方法.
        if("cat".equalsIgnoreCase(type)){
            return new Cat();
        }
        return null;
    }
}
```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 工厂类调用静态工厂方法,创建对应Bean -->
	<bean id="cat" class="top.jionjion.ioc.xml.factory.StaticAnimalFactory" factory-method="getAnimal">
        <constructor-arg value="cat"/>
    </bean>
</beans>
```

测试用例

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;

/**
 *  测试静态工厂方法获得Bean
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(locations = "classpath:ioc/xml/bean-config.xml")
public class StaticAnimalFactoryTest {

    @Autowired
    Cat cat;

    @Test
    public void test(){
        System.out.println("获得Bean:" + cat.toString());
    }
}
```



#### 实例工厂注入

实例工厂类

```java
public class InstanceAnimalFactory {
    public Animal getAnimal(String type){
        // 工厂方法.
        if("cat".equalsIgnoreCase(type)){
            return new Cat();
        }
        return null;
    }
}
```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 实例工厂方法 -->
    <bean id="instanceAnimalFactory" class="top.jionjion.ioc.xml.factory.InstanceAnimalFactory"/>
    <!-- 工厂类调用实例工厂方法,创建对应Bean  -->
    <bean id="cat" factory-bean="instanceAnimalFactory" factory-method="getAnimal">
        <constructor-arg value="cat"/>
    </bean>
</beans>

```

测试用例

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;

/**
 *  测试实例工厂方法获得Bean
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(locations = "classpath:ioc/xml/bean-config.xml")
public class InstanceAnimalFactoryTest {

    @Autowired
    Cat cat;

    @Test
    public void test(){
        System.out.println("获得Bean:" + cat.toString());
    }
}
```



### `Annotation` 方式

- 使用 `@Component` 注解
- 配置类中使用 `@Bean`
- 实现 `FactoryBean`
- 实现 `BeanDefinitionRegistryPostProcessor`
- 实现 `ImportBeanDefinitionRegistrar`

#### 使用 @Component 注解

创建 `DogHouse` 类

```java
@Component
public class DogHouse {

}
```

测试用例

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 *  测试 @Component 注入Bean
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class DogHouseTest {

    @Autowired
    DogHouse dogHouse;

    @Test
    public void test(){
        System.out.println("获得Bean:" + dogHouse);
    }
}
```



#### 配置类中使用 `@Bean`

声明一个空类 `Dog`

```java
public class Dog {

}
```

使用 `@Configuration` 注解一个类为配置信息类, 并使用  `@Bean` 注入

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 *  注解配置文件
 * @author Jion
 */
@Configuration
public class BeanConfig {

    @Bean
    public Dog dog(){
        return new Dog();
    }
}
```

测试用例

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 *  测试,通过 @Configuration 结合 @Bean 注入Bean
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class BeanConfigTest {

    @Autowired
    Dog dog;

    @Test
    public void test() {
        System.out.println("获得Bean:" + dog);
    }
}
```



#### 实现 `FactoryBean<T>`

声明抽象类 `Animal` 

```java
public abstract class Animal {

    /**
     *  获得动物名称
     * @return 名称
     */
     abstract String getName();
}
```

具体的一个子类 `Cat`

```java
public class Cat extends Animal {

    @Override
    String getName() {
        return "我是猫... ";
    }
}
```

通过实现 `FactoryBean<T>` 接口传入父类的类型,并在 `@Component()` 中指定生成的Bean的名字

```java
import org.springframework.beans.factory.FactoryBean;
import org.springframework.stereotype.Component;

/**
 *  使用 FactoryBean<T> 注入Bean
 * @author Jion
 */
@Component("cat")
public class CatFactoryBean implements FactoryBean<Animal> {


    @Override
    public Animal getObject() throws Exception {
        return new Cat();
    }

    @Override
    public Class<?> getObjectType() {
        return Animal.class;
    }
}
```

测试用例, 多实现类,需要通过 `@Qualifier` 指定具体的子类的 `Bean` 名称

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 *  测试 FactoryBean<T> 注入Bean
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class CatFactoryBeanTest {

    @Autowired
    @Qualifier("cat") // 指定具体Bean
    private Animal cat;

    @Test
    public void test(){
        System.out.println("获得Bean:" + cat);
    }
}
```



#### 实现 `BeanDefinitionRegistryPostProcessor`

创建子类 `Monkey`

```java
public class Monkey extends Animal {

    @Override
    String getName() {
        return "我是猴子... ";
    }
}
```

通过实现 `BeanDefinitionRegistryPostProcessor` 接口, 并在其 `postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)` 方法中向容器注入Bean的类型和名称
`@Component` 将该实现类注入容器

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.stereotype.Component;

/**
 *  使用 BeanDefinitionRegistryPostProcessor 注入 Bean
 * @author Jion
 */
@Component
public class MonkeyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        // 注入Bean
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
        rootBeanDefinition.setBeanClass(Monkey.class);
        // 注入Bean名和定义
        registry.registerBeanDefinition("monkey", rootBeanDefinition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

    }
}
```

测试用例

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 *  测试 BeanDefinitionRegistryPostProcessor 注入Bean
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class MonkeyBeanDefinitionRegistryPostProcessorTest {

    @Autowired
    @Qualifier("monkey")
    private Animal monkey;

    @Test
    public void test(){
        System.out.println("获得Bean:" + monkey);
    }
}
```



#### 实现 `ImportBeanDefinitionRegistrar`

创建子类 `Duck`

```java
public class Duck extends Animal {

    @Override
    String getName() {
        return "我是鸭子... ";
    }
}
```

通过重写 `ImportBeanDefinitionRegistrar`  中的方法, 将自定义的Bean注入容器.
`@Component`  将该实现类注入容器

```java
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.stereotype.Component;

/**
 *  使用 ImportBeanDefinitionRegistrar 注入Bean
 * @author Jion
 */
@Component
public class DuckImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        // 注入Bean
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition();
        rootBeanDefinition.setBeanClass(Duck.class);
        // 注入Bean名和定义
        registry.registerBeanDefinition("duck", rootBeanDefinition);
    }
}

```

测试用例, 需要使用 `@Import(DuckImportBeanDefinitionRegistrar.class)` 引入具体的实现类

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Import;
import org.springframework.test.context.junit4.SpringRunner;

/**
 *  使用 ImportBeanDefinitionRegistrar 注入Bean
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest
@Import(DuckImportBeanDefinitionRegistrar.class)
public class DuckImportBeanDefinitionRegistrarTest {

    @Autowired
    Duck duck;

    @Test
    public void test(){
        System.out.println("获得Bean:" + duck);
    }
}
```



### `BeanDefinition` 类

在Spring中,对类的描述信息, 封装为 `org.springframework.beans.factory.config.BeanDefinition` 类. 
通过操作 `BeanDefinition` 完成 `Bean` 的实例化和属性注入

常见实现类为 `org.springframework.beans.factory.support.RootBeanDefinition`

#### 类图

其类图如下.

![类图信息](./Spring全家桶-SpringBoot源码篇/BeanDefinition类图-1.png)



#### 自定义 `Bean` 初始化

通过实现 `org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor` 中的方法, 自定义创建 `Bean` 对象

```java
@Component
public class PeopleBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        // 在类初始化前调用, 若有返回值,则将该返回值作为实例化结果
        if ("people".equals(beanName)){
            return new People();
        }
        // 返回为null, 继续初始化
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        // 在类实例化后调用.
        if ("people".equals(beanName)){
            // 名称一致, 实例化, 修改属性
            People people = (People)  bean;
            people.setName("Aires");
        }
        // 返回 true 继续执行
        return true;
    }
}
```



## 容器刷新



### 作用

1. Bean的配置读取
2. Spring框架启动流程



### 源码步骤

#### 2-11-0-0 容器刷新

在 `org.springframework.boot.SpringApplication#run(java.lang.String...)` 中,通过 `refreshContext(context)` 调用容器刷新.

```java
private void refreshContext(ConfigurableApplicationContext context) {
    // 刷新容器
    refresh((ApplicationContext) context);
    if (this.registerShutdownHook) {
        try {
            context.registerShutdownHook();
        }
        catch (AccessControlException ex) {
            // Not allowed in some environments.
        }
    }
}
```

其中需要根据不同的环境会有不同的刷新,步骤.其父类 `org.springframework.context.support.AbstractApplicationContext#refresh` 中定义了主要的刷新步骤.

其方法为 `synchronized` 同步方法, 线程独占. 

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 1. 准备刷新
        prepareRefresh();

        // 2. 获取容器的 beanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // 3. 初始化 beanFactory, 设置相关属性,后置处理器
        prepareBeanFactory(beanFactory);

        try {
            // 4. 注册后置处理器, 具体子类实现
            postProcessBeanFactory(beanFactory);

            // 5. 注册BeanFactory 定义, 并对注册后的 Bean实例属性修改
            invokeBeanFactoryPostProcessors(beanFactory);

            // 6. 为Bean 添加初始化前后定义方法
            registerBeanPostProcessors(beanFactory);

            // 7. 初始化国际化资源
            initMessageSource();

            // 8. 初始化事件广播器
            initApplicationEventMulticaster();

            // 9. 根据不同的环境, 创建特护的类. 如 Web 容器
            onRefresh();

            // 10. 注册监听器到广播器
            registerListeners();

            // 11. 初始化单例的 Bean
            finishBeanFactoryInitialization(beanFactory);

            // 12. 结束刷新, 发布事件 ContextRefreshedEvent
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // 13. 清理相关数据
            resetCommonCaches();
        }
    }
}
```



#### 2-11-1-0 准备刷新

主要进行 容器状态设置,  必备属性检查 ,初始化属性设置(环境, 应用监听器)

```java
protected void prepareRefresh() {
    // 设置容器状态
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);
    // 日志
    if (logger.isDebugEnabled()) {
        if (logger.isTraceEnabled()) {
            logger.trace("Refreshing " + this);
        }
        else {
            logger.debug("Refreshing " + getDisplayName());
        }
    }

    // 1. 初始化环境上下文信息
    initPropertySources();

    // 2. 检查容器环境中,启动时需要配置的属性.
    getEnvironment().validateRequiredProperties();

    // 3. 存放需要的系统监听器, this.applicationListeners 之前通过 loadSpringFactories 加载
    if (this.earlyApplicationListeners == null) {
        // this.earlyApplicationListeners 为 null, 需要重新复制
        this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    }
    else {
        // 不为null, 重置 this.applicationListeners 系统监听器
        this.applicationListeners.clear();
        this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // 初始化 属性 earlyApplicationEvents
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```



#### 2-11-1-1 初始化环境下文

`org.springframework.web.context.support.GenericWebApplicationContext#initPropertySources` 中调用.初始化环境上下文.
当前调用环境一般为 `org.springframework.web.context.support.StandardServletEnvironment`

```java
// 准备跟 Servlet 相关的环境信息
protected void initPropertySources() {
    ConfigurableEnvironment env = getEnvironment();
    // StandardServletEnvironment    
    if (env instanceof ConfigurableWebEnvironment) {
        // 初始化 Servlet信息, 传入 null
        ((ConfigurableWebEnvironment) env).initPropertySources(this.servletContext, null);
    }
}
```

初始化Servlet信息, 传入 `null` 并不会实际调用

```java
@Override
public void initPropertySources(@Nullable ServletContext servletContext, @Nullable ServletConfig servletConfig) {
    WebApplicationContextUtils.initServletPropertySources(getPropertySources(), servletContext, servletConfig);
}
```



#### 2-11-2-0 获取容器BeanFactory

调用 `org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory` 方法.  告诉子类需要刷新 `BeanFactory`

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    // 刷新
    refreshBeanFactory();
    // 获得
    return getBeanFactory();
}
```



#### 2-11-2-1 刷新 `BeanFactory` 状态

设置容器刷新状态; 设置序列化ID

```java
protected final void refreshBeanFactory() throws IllegalStateException {
    // 设置状态,标识容器正在刷新. 乐观锁
    if (!this.refreshed.compareAndSet(false, true)) {
        throw new IllegalStateException(
            "GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
    }
    // 设置序列化ID org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@5e1fa5b1
    this.beanFactory.setSerializationId(getId());
}
```



#### 2-11-2-2 返回 `BeanFactory` 实例

调用子类的 `org.springframework.context.support.AbstractRefreshableApplicationContext#getBeanFactory` 方法,返回 `org.springframework.beans.factory.support.DefaultListableBeanFactory` 实例

```java
	@Override
	public final ConfigurableListableBeanFactory getBeanFactory() {
		DefaultListableBeanFactory beanFactory = this.beanFactory;
		if (beanFactory == null) {
			throw new IllegalStateException("BeanFactory not initialized or already closed - " +
					"call 'refresh' before accessing beans via the ApplicationContext");
		}
		return beanFactory;
	}
```



#### 2-11-3-0 初始化 `BeanFactory`

将当前 ` BeanFactory` 与环境信息相绑定
调用 `org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory`
设置 BeanFactory的一些属性; 添加后置处理器; 设置或略的自动装配接口; 注册组件

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 设置类加载器
    beanFactory.setBeanClassLoader(getClassLoader());
    // 设置解析器. 解析SPEL表达式
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    // 设置属性转换器. 转换属性
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // 设置 Bean的后置处理器
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    // 忽略自动装配的接口
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // 添加解析依赖.
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // 后置处理器
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    // 是否包含  loadTimeWeaver
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        // 临时的ClassLoader
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // 是否包含 environment 
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        // 注册默认环境相关的Bean 
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    // systemProperties
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    // systemEnvironment
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```



#### 2-11-4-0 设置 `BeanFactory` 后置处理器

调用子类 `org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext#postProcessBeanFactory` 方法进行设置

一般由子类重写,在`BeanFactory` 完成创建后进一步作设置, 如 `Web` 相关的一些组件

```java
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 1. 设置 ServletContextAwareProcessor 后置处理器
    super.postProcessBeanFactory(beanFactory);
    // 包扫描
    if (this.basePackages != null && this.basePackages.length > 0) {
        this.scanner.scan(this.basePackages);
    }
    // 注解
    if (!this.annotatedClasses.isEmpty()) {
        this.reader.register(ClassUtils.toClassArray(this.annotatedClasses));
    }
}
```



#### 2-11-4-1 设置 `ServletContextAwareProcessor` 后置处理器

```java
@Override
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 设置后置处理器
    beanFactory.addBeanPostProcessor(new WebApplicationContextServletContextAwareProcessor(this));
    // 忽略装配的接口
    beanFactory.ignoreDependencyInterface(ServletContextAware.class);
    // 
    registerWebApplicationScopes();
}

// 注册 Web 应用的作用域
private void registerWebApplicationScopes() {
    ExistingWebApplicationScopes existingScopes = new ExistingWebApplicationScopes(getBeanFactory());
    // 设置作用域
    WebApplicationContextUtils.registerWebApplicationScopes(getBeanFactory());
    existingScopes.restore();
}


```



#### 2-11-4-2 设置 `Web` 作用域

```java
public static void registerWebApplicationScopes(ConfigurableListableBeanFactory beanFactory,
                                                @Nullable ServletContext sc) {
	// 设置作用域
    beanFactory.registerScope(WebApplicationContext.SCOPE_REQUEST, new RequestScope());
    beanFactory.registerScope(WebApplicationContext.SCOPE_SESSION, new SessionScope());
    // Servlet 容器. 传入sc为null
    if (sc != null) {
        ServletContextScope appScope = new ServletContextScope(sc);
        beanFactory.registerScope(WebApplicationContext.SCOPE_APPLICATION, appScope);
        // 设置属性
        sc.setAttribute(ServletContextScope.class.getName(), appScope);
    }

    // 解析依赖
    beanFactory.registerResolvableDependency(ServletRequest.class, new RequestObjectFactory());
    beanFactory.registerResolvableDependency(ServletResponse.class, new ResponseObjectFactory());
    beanFactory.registerResolvableDependency(HttpSession.class, new SessionObjectFactory());
    beanFactory.registerResolvableDependency(WebRequest.class, new WebRequestObjectFactory());
    if (jsfPresent) {
        FacesDependencyRegistrar.registerFacesDependencies(beanFactory);
    }
}
```



#### 2-11-5-0 初始化BeanFactory

调用 `BeanDefinitionRegistryPostProcessor` 接口的实现, 向容器中添加 `Bean` 的定义.
调用 `BeanFactoryPostProcessor` 接口的实现., 向容器内的 `Bean` 添加/修改属性

```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // getBeanFactoryPostProcessors() 首先获得获得 BeanFactoryPostProcessors 的实现.
    // 1. 初始化Bean
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

    // 方法跳过, 不包含 loadTimeWeaver
    if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }
}
```



#### 2-11-5-1 调用 `invokeBeanFactoryPostProcessors` 生成 `FactoryBean`

主要有三个遍历

首先遍历 `beanFactoryPostProcessors` 中的所有类,是否实现了 `BeanDefinitionRegistry` 接口 (用以向容器中注册 `Bean` ) , 
如果未实现了, 则放入 `regularPostProcessors` 集合, 表示未注入
如果实现了,则调用接口中的`postProcessBeanDefinitionRegistry` 方法, 向容器中添加 `Bean`, 并将该类放入 `registryProcessors` 集合,表示已注入

随后再次遍历 `beanFactory` 中 `BeanDefinitionRegistryPostProcessor` 接口的实现(用以向容器中注册 `Bean`),
其中实现类如果实现 `PriorityOrdered` 接口排序, 则将其添加到 `currentRegistryProcessors` 集合中,稍后对其排序, 并调用 `invokeBeanDefinitionRegistryPostProcessors` 方法初始化. 同时加入 `processedBeans` 集合表示该类已经被初始化.
清空集合.
最后为 ` while` 循环,  当仍有依赖尚未实现的 `BeanDefinitionRegistryPostProcessor` 接口实现类, 则通过循环,完成排序,初始化,清空集合.. 直到所有的 `BeanDefinitionRegistryPostProcessor` 接口实现类完成注入.

前两个遍历结束后,调用 `BeanFactoryPostProcessor` 接口的 `postProcessBeanFactory` 对注入后的 `Bean` 进行更多操作

剩余循环逻辑相似....

```java
public static void invokeBeanFactoryPostProcessors(
	ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

    // Invoke BeanDefinitionRegistryPostProcessors first, if any.
    Set<String> processedBeans = new HashSet<>();

    if (beanFactory instanceof BeanDefinitionRegistry) {
        BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
        List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
        List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

        for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
            if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
                BeanDefinitionRegistryPostProcessor registryProcessor =
                    (BeanDefinitionRegistryPostProcessor) postProcessor;
                registryProcessor.postProcessBeanDefinitionRegistry(registry);
                registryProcessors.add(registryProcessor);
            }
            else {
                regularPostProcessors.add(postProcessor);
            }
        }

        // Do not initialize FactoryBeans here: We need to leave all regular beans
        // uninitialized to let the bean factory post-processors apply to them!
        // Separate between BeanDefinitionRegistryPostProcessors that implement
        // PriorityOrdered, Ordered, and the rest.
        List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

        // First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
        String[] postProcessorNames =
            beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
            }
        }
        sortPostProcessors(currentRegistryProcessors, beanFactory);
        registryProcessors.addAll(currentRegistryProcessors);
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
        currentRegistryProcessors.clear();

        // Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
        postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
            }
        }
        sortPostProcessors(currentRegistryProcessors, beanFactory);
        registryProcessors.addAll(currentRegistryProcessors);
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
        currentRegistryProcessors.clear();

        // Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
        boolean reiterate = true;
        while (reiterate) {
            reiterate = false;
            postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            for (String ppName : postProcessorNames) {
                if (!processedBeans.contains(ppName)) {
                    currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    processedBeans.add(ppName);
                    reiterate = true;
                }
            }
            sortPostProcessors(currentRegistryProcessors, beanFactory);
            registryProcessors.addAll(currentRegistryProcessors);
            invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
            currentRegistryProcessors.clear();
        }

        // Now, invoke the postProcessBeanFactory callback of all processors handled so far.
        invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
        invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
    }

    else {
        // Invoke factory processors registered with the context instance.
        invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
    }

    // Do not initialize FactoryBeans here: We need to leave all regular beans
    // uninitialized to let the bean factory post-processors apply to them!
    String[] postProcessorNames =
        beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

    // Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
    // Ordered, and the rest.
    List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    List<String> orderedPostProcessorNames = new ArrayList<>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
        if (processedBeans.contains(ppName)) {
            // skip - already processed in first phase above
        }
        else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
        }
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            orderedPostProcessorNames.add(ppName);
        }
        else {
            nonOrderedPostProcessorNames.add(ppName);
        }
    }

    // First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

    // Next, invoke the BeanFactoryPostProcessors that implement Ordered.
    List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String postProcessorName : orderedPostProcessorNames) {
        orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }
    sortPostProcessors(orderedPostProcessors, beanFactory);
    invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

    // Finally, invoke all other BeanFactoryPostProcessors.
    List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String postProcessorName : nonOrderedPostProcessorNames) {
        nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }
    invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

    // Clear cached merged bean definitions since the post-processors might have
    // modified the original metadata, e.g. replacing placeholders in values...
    beanFactory.clearMetadataCache();
}
```



#### 2-11-6-0 配置 `Bean` 的初始化方法

找到 `BeanPostProcessor` 的实现, 定义 `Bean` 在初始化前后的动作
根据 `Order` 对其实现进行排序

```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```

#### 2-11-6-1 具体配置方法

调用 `org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors` 具体实现

```java
public static void registerBeanPostProcessors(
    ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

    // Register BeanPostProcessorChecker that logs an info message when
    // a bean is created during BeanPostProcessor instantiation, i.e. when
    // a bean is not eligible for getting processed by all BeanPostProcessors.
    int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
    beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

    // Separate between BeanPostProcessors that implement PriorityOrdered,
    // Ordered, and the rest.
    List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
    List<String> orderedPostProcessorNames = new ArrayList<>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            priorityOrderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            orderedPostProcessorNames.add(ppName);
        }
        else {
            nonOrderedPostProcessorNames.add(ppName);
        }
    }

    // First, register the BeanPostProcessors that implement PriorityOrdered.
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

    // Next, register the BeanPostProcessors that implement Ordered.
    List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String ppName : orderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        orderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    sortPostProcessors(orderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, orderedPostProcessors);

    // Now, register all regular BeanPostProcessors.
    List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String ppName : nonOrderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        nonOrderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

    // Finally, re-register all internal BeanPostProcessors.
    sortPostProcessors(internalPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, internalPostProcessors);

    // Re-register post-processor for detecting inner beans as ApplicationListeners,
    // moving it to the end of the processor chain (for picking up proxies etc).
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
}
```



#### 2-11-7-0 初始化国际化资源

如果存在 `messageSource` 则进行配置, 否则自动生成一个放入容器.
一般没有,自动创建一个

```java
protected void initMessageSource() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
        this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);
        // Make MessageSource aware of parent MessageSource.
        if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
            HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
            if (hms.getParentMessageSource() == null) {
                // Only set parent context as parent MessageSource if no parent MessageSource
                // registered already.
                hms.setParentMessageSource(getInternalParentMessageSource());
            }
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Using MessageSource [" + this.messageSource + "]");
        }
    }
    else {
        // Use empty MessageSource to be able to accept getMessage calls.
        DelegatingMessageSource dms = new DelegatingMessageSource();
        dms.setParentMessageSource(getInternalParentMessageSource());
        this.messageSource = dms;
        beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + MESSAGE_SOURCE_BEAN_NAME + "' bean, using [" + this.messageSource + "]");
        }
    }
}

```



#### 2-11-8-0 初始化事件广播器

查看 `applicationEventMulticaster` 事件广播器是否存在,不存在则创建放入容器.

```java
protected void initApplicationEventMulticaster() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
        this.applicationEventMulticaster =
            beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
        if (logger.isTraceEnabled()) {
            logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
        }
    }
    else {
        this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
        beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + APPLICATION_EVENT_MULTICASTER_BEAN_NAME + "' bean, using " +
                         "[" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
        }
    }
}
```



#### 2-11-9-0 初始化特殊的 `Bean`

根据不同的环境, 交由具体的子类实现

`Web` 环境下,交由 `org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#onRefresh` 去实现, 具体为创建一个 `web` 容器

```java
protected void onRefresh() {
    super.onRefresh();
    try {
        createWebServer();
    }
    catch (Throwable ex) {
        throw new ApplicationContextException("Unable to start web server", ex);
    }
}
```



#### 2-11-10-0 注册监听器到广播器

注册从 `META-INF/spring.factories` 读取的系统监听器; 注册从 `BeanFactory` 中读取的系统监听器; 广播早期事件

`earlyApplicationEvents` 为早期事件, 当容器尚未创建完成但却已触发事件,则会保存其中

```java
protected void registerListeners() {
    // 注册系统监听器, 已经通过 SpringFactoriesLoader 加载
    for (ApplicationListener<?> listener : getApplicationListeners()) {
        getApplicationEventMulticaster().addApplicationListener(listener);
    }

    // 获得程序定义的监听器,而非 META-INF/spring.factories 中定义的
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    for (String listenerBeanName : listenerBeanNames) {
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    }

    // 如 earlyApplicationEvents 不为空, 则将其广播
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (!CollectionUtils.isEmpty(earlyEventsToProcess)) {
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
        }
    }
}
```



#### 2-11-11-0 初始化单例 `Bean`

初始化剩下的单实例 `Bean`

**主要为自定义的业务逻辑 `Bean`**

步骤为:
`getBean` -> `doGetBean` -> `getSingleton` -> `CreateBean` -> `resolveBeforeInstantiation` -> `doGreateBean` -> `createBeanInstance` -> `instantiateBean` -> `instantiate` -> `populateBean` -> `initializedBean` 

具体代码如下

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 判断是否含有 conversionService 的实现,如果有则存入.
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
        beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // 是否加载 解析器, 没有则注入
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // 加载 LoadTimeWeaverAware 进行 AOP 织入操作
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // 停止使用临时的 ClassLoader
    beanFactory.setTempClassLoader(null);

    // 1. 冻结 BeanFactory , 实例化期间不希望有 Bean 注入
    beanFactory.freezeConfiguration();

    // 2. 实例剩下的单实例 Bean
    beanFactory.preInstantiateSingletons();
}
```



#### 2-11-11-1 冻结 `BeanFactory`

标记当前 `BeanFactory` 状态为 `configurationFrozen`  , 停止其他 `Bean` 的注入
`frozenBeanDefinitionNames`  存放当前正在实例化中的 `Bean` 实例

```java
@Override
public void freezeConfiguration() {
    this.configurationFrozen = true;
    this.frozenBeanDefinitionNames = StringUtils.toStringArray(this.beanDefinitionNames);
}
```



#### 2-11-11-2 单例 `Bean` 的实例化

子类 `org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons` 方法调用, 进行实例化

```java
public void preInstantiateSingletons() throws BeansException {
    if (logger.isTraceEnabled()) {
        logger.trace("Pre-instantiating singletons in " + this);
    }

    // 获得所有的 Bean 定义的名字
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    // 构建单实例 Bean
    for (String beanName : beanNames) {
        // 获得 Bean 的 BeanDefinition 描述信息
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        // 非抽象类, 单例, 非延迟加载
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            // 当前 Bean 是否为 工厂类
            if (isFactoryBean(beanName)) {
                // 获得 FactoryBean 实例, Bean 的名字, 以  & 开头,表示工厂本身,而非工厂生产的Bean
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                // 属于 FactoryBean
                if (bean instanceof FactoryBean) {
                    final FactoryBean<?> factory = (FactoryBean<?>) bean;
                    boolean isEagerInit;
                    if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                        isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                                                    ((SmartFactoryBean<?>) factory)::isEagerInit,
                                                                    getAccessControlContext());
                    }
                    else {
                        isEagerInit = (factory instanceof SmartFactoryBean &&
                                       ((SmartFactoryBean<?>) factory).isEagerInit());
                    }
                    if (isEagerInit) {
                        getBean(beanName);
                    }
                }
            }
            else {
                // 3. 非工厂 FactoryBean , 实例化
                getBean(beanName);
            }
        }
    }

    // Trigger post-initialization callback for all applicable beans...
    for (String beanName : beanNames) {
        Object singletonInstance = getSingleton(beanName);
        if (singletonInstance instanceof SmartInitializingSingleton) {
            final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
            if (System.getSecurityManager() != null) {
                AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                    smartSingleton.afterSingletonsInstantiated();
                    return null;
                }, getAccessControlContext());
            }
            else {
                smartSingleton.afterSingletonsInstantiated();
            }
        }
    }
}
```

#### 2-11-11-3 实例化 `Bean`

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
	// 获得 Bean 的名字. 如果是工厂类,去&前缀, 如果是别名, 获得原Bean名字
    final String beanName = transformedBeanName(name);
    Object bean;

    // 尝试从缓存中.读取 Bean
    Object sharedInstance = getSingleton(beanName);
    // 缓存读取成功
    if (sharedInstance != null && args == null) {
        if (logger.isTraceEnabled()) {
            if (isSingletonCurrentlyInCreation(beanName)) {
                logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
                             "' that is not fully initialized yet - a consequence of a circular reference");
            }
            else {
                logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
            }
        }
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
	// 缓存读取失败
    else {
        // Fail if we're already creating this bean instance:
        // We're assumably within a circular reference.
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }

        // 获取 父类 BeanFactory
        BeanFactory parentBeanFactory = getParentBeanFactory();
        // 如果 父类已经加载过,则返回
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            // Not found -> check parent.
            String nameToLookup = originalBeanName(name);
            if (parentBeanFactory instanceof AbstractBeanFactory) {
                return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                    nameToLookup, requiredType, args, typeCheckOnly);
            }
            else if (args != null) {
                // Delegation to parent with explicit args.
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            }
            else if (requiredType != null) {
                // No args -> delegate to standard getBean method.
                return parentBeanFactory.getBean(nameToLookup, requiredType);
            }
            else {
                return (T) parentBeanFactory.getBean(nameToLookup);
            }
        }

        // 标记当前Bean正在被创建
        if (!typeCheckOnly) {
            markBeanAsCreated(beanName);
        }

        try {
            // 获得 Bean 的定义信息
            final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            // 检查
            checkMergedBeanDefinition(mbd, beanName, args);

            // 获得 Bean 的依赖对象
            String[] dependsOn = mbd.getDependsOn();
            // 如果存在依赖, 则优先创建依赖对象 @DependOn 注解标识需要的Bean
            if (dependsOn != null) {
                for (String dep : dependsOn) {
                    if (isDependent(beanName, dep)) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                        "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                    }
                    registerDependentBean(dep, beanName);
                    try {
                        getBean(dep);
                    }
                    catch (NoSuchBeanDefinitionException ex) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                        "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
                    }
                }
            }

            // 不存在依赖, 如果是单例,创建 Bean
            if (mbd.isSingleton()) {
                // 获得 Bean 工厂
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    catch (BeansException ex) {
                        // Explicitly remove instance from singleton cache: It might have been put there
                        // eagerly by the creation process, to allow for circular reference resolution.
                        // Also remove any beans that received a temporary reference to the bean.
                        destroySingleton(beanName);
                        throw ex;
                    }
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }

            else if (mbd.isPrototype()) {
                // It's a prototype -> create a new instance.
                Object prototypeInstance = null;
                try {
                    beforePrototypeCreation(beanName);
                    prototypeInstance = createBean(beanName, mbd, args);
                }
                finally {
                    afterPrototypeCreation(beanName);
                }
                bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
            }

            else {
                String scopeName = mbd.getScope();
                final Scope scope = this.scopes.get(scopeName);
                if (scope == null) {
                    throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                }
                try {
                    Object scopedInstance = scope.get(beanName, () -> {
                        beforePrototypeCreation(beanName);
                        try {
                            return createBean(beanName, mbd, args);
                        }
                        finally {
                            afterPrototypeCreation(beanName);
                        }
                    });
                    bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                }
                catch (IllegalStateException ex) {
                    throw new BeanCreationException(beanName,
                                                    "Scope '" + scopeName + "' is not active for the current thread; consider " +
                                                    "defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                                                    ex);
                }
            }
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }

    // Check if required type matches the type of the actual bean instance.
    if (requiredType != null && !requiredType.isInstance(bean)) {
        try {
            T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
            if (convertedBean == null) {
                throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
            }
            return convertedBean;
        }
        catch (TypeMismatchException ex) {
            if (logger.isTraceEnabled()) {
                logger.trace("Failed to convert bean '" + name + "' to required type '" +
                             ClassUtils.getQualifiedName(requiredType) + "'", ex);
            }
            throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
        }
    }
    return (T) bean;
}
```





#### 2-11-12-0 结束刷新

初始化生命周期处理器 `LifecycleProcessor` 实现类, 并调用 `onRefresh` 方法
发布 `ContextRefreshedEvent` 事件
`JMX` 相关处理

```java
protected void finishRefresh() {
    // 清空缓存
    clearResourceCaches();

    // 初始化生命周期
    initLifecycleProcessor();

    // 调用 LifecycleProcessor 接口实现类的 onRefresh 方法
    getLifecycleProcessor().onRefresh();

    // 发布事件 ContextRefreshedEvent
    publishEvent(new ContextRefreshedEvent(this));

    // JMX 相关处理
    LiveBeansView.registerApplicationContext(this);
}
```



#### 2-11-13-0 清理相关

清理相关 `Bean` 的元数据信息

```java
protected void resetCommonCaches() {
    ReflectionUtils.clearCache();
    AnnotationUtils.clearCache();
    ResolvableType.clearCache();
    CachedIntrospectionResults.clearClassLoader(getClassLoader());
}
```



## 单实例 `Bean` 注入





## `Banner` 打印

通过设置 `spring.banner.location` 设置文字简笔画图片
通过设置 `spring.banner.image.location` 设置图片(png,jpg,gif),系统会转为对应的字符简笔画
或者在 `SpringApplication` 中的 `setBanner()` 方法配置信息
系统默认位置在 `resources/banner.txt`  目录下



### 源码步骤

#### 2-7-0-0 打印 `Banner`

在 `run` 方法中, 调用 `Banner printedBanner = printBanner(environment)` 进行打印.

```java
private Banner printBanner(ConfigurableEnvironment environment) {
    // 是否启用 Banner 打印
    if (this.bannerMode == Banner.Mode.OFF) {
        return null;
    }
    // 1. 资源加载器,如果不存在则创建.打印默认的 Banner
    ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader
        : new DefaultResourceLoader(getClassLoader());
    SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
    // 2. 打印到日志
    if (this.bannerMode == Mode.LOG) {
        return bannerPrinter.print(environment, this.mainApplicationClass, logger);
    }
    // 打印到控制台
    return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}
```

#### 2-7-1-0 获得 `Banner` 并打印

```java
Banner print(Environment environment, Class<?> sourceClass, Log logger) {
    // 1. 获得 Banner 对象
    Banner banner = getBanner(environment);
    try {
        // 日志
        logger.info(createStringFromBanner(banner, environment, sourceClass));
    }
    catch (UnsupportedEncodingException ex) {
        logger.warn("Failed to create String for banner", ex);
    }
    // 2. 构建打印对象
    return new PrintedBanner(banner, sourceClass);
}
```

#### 2-7-1-1 获得 `Banner` 对象

```java
private Banner getBanner(Environment environment) {
    // 创建 Banner 列表, 可以有多个Banner
    Banners banners = new Banners();
    // 尝试添加图片打印
    banners.addIfNotNull(getImageBanner(environment));
    // 尝试添加文字打印
    banners.addIfNotNull(getTextBanner(environment));    
    if (banners.hasAtLeastOneBanner()) {
        return banners;
    }
    // 如果在 SpringApplication 中指定,则返回
    if (this.fallbackBanner != null) {
        return this.fallbackBanner;
    }
    // 以上均未匹配, 返回默认的配置
    return DEFAULT_BANNER;
}

```

#### 2-7-1-2 不同类型的打印

默认打印的 `SpringBanner` 类如下.

```java
class SpringBootBanner implements Banner {

	private static final String[] BANNER = { "", "  .   ____          _            __ _ _",
			" /\\\\ / ___'_ __ _ _(_)_ __  __ _ \\ \\ \\ \\", "( ( )\\___ | '_ | '_| | '_ \\/ _` | \\ \\ \\ \\",
			" \\\\/  ___)| |_)| | | | | || (_| |  ) ) ) )", "  '  |____| .__|_| |_|_| |_\\__, | / / / /",
			" =========|_|==============|___/=/_/_/_/" };

	private static final String SPRING_BOOT = " :: Spring Boot :: ";

	private static final int STRAP_LINE_SIZE = 42;

	@Override
	public void printBanner(Environment environment, Class<?> sourceClass, PrintStream printStream) {
        // 文字简笔画打印
		for (String line : BANNER) {
			printStream.println(line);
		}
        // 版本信息
		String version = SpringBootVersion.getVersion();
		version = (version != null) ? " (v" + version + ")" : "";
		StringBuilder padding = new StringBuilder();
        // 字符串补空格
		while (padding.length() < STRAP_LINE_SIZE - (version.length() + SPRING_BOOT.length())) {
			padding.append(" ");
		}
		// 文字设置颜色
		printStream.println(AnsiOutput.toString(AnsiColor.GREEN, SPRING_BOOT, AnsiColor.DEFAULT, padding.toString(),
				AnsiStyle.FAINT, version));
		printStream.println();
	}
}
```

文字打印如下
具体方法为 `org.springframework.boot.ResourceBanner#printBanner` 

```java
@Override
public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
    try {
        // 读取文本, 指定字符
        String banner = StreamUtils.copyToString(this.resource.getInputStream(),
                                                 environment.getProperty("spring.banner.charset", Charset.class, StandardCharsets.UTF_8));
		// 可以通过 ${} 为系统指定环境信息
        for (PropertyResolver resolver : getPropertyResolvers(environment, sourceClass)) {
            banner = resolver.resolvePlaceholders(banner);
        }
        // 打印文本内容
        out.println(banner);
    }
    catch (Exception ex) {
        logger.warn(LogMessage.format("Banner not printable: %s (%s: '%s')", this.resource, ex.getClass(),
                                      ex.getMessage()), ex);
    }
}
```

图案打印

具体方法为 `org.springframework.boot.ImageBanner#printBanner`

```java
public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
	// 是否在 headless 模式下
    String headless = System.getProperty("java.awt.headless");
    try {
        System.setProperty("java.awt.headless", "true");
        // 打印方法
        printBanner(environment, out);
    }
    catch (Throwable ex) {
        logger.warn(LogMessage.format("Image banner not printable: %s (%s: '%s')", this.image, ex.getClass(),
                                      ex.getMessage()));
        logger.debug("Image banner printing failure", ex);
    }
    finally {
        if (headless == null) {
            System.clearProperty("java.awt.headless");
        }
        else {
            System.setProperty("java.awt.headless", headless);
        }
    }
}
```

具体打印

```java
private void printBanner(Environment environment, PrintStream out) throws IOException {
    // 基本信息. 通过 spring.banner.image.* 设置属性
   	int width = getProperty(environment, "width", Integer.class, 76);
    int height = getProperty(environment, "height", Integer.class, 0);
    int margin = getProperty(environment, "margin", Integer.class, 2);
    boolean invert = getProperty(environment, "invert", Boolean.class, false);
 	// 打印属性
    BitDepth bitDepth = getBitDepthProperty(environment);
    PixelMode pixelMode = getPixelModeProperty(environment);
    // 读取
    Frame[] frames = readFrames(width, height);
    for (int i = 0; i < frames.length; i++) {
        if (i > 0) {
            resetCursor(frames[i - 1].getImage(), out);
        }
        // 打印输出流,图片内容
        printBanner(frames[i].getImage(), margin, invert, bitDepth, pixelMode, out);
        sleep(frames[i].getDelayTime());
    }
}
```



## 计时器 `StopWatch`

Spring提供的计时器,主要方法如下.
可以多次停止,纪录多个任务,并

```java
// 创建计时器
StopWatch stopWatch = new StopWatch();  
// 计时,指定任务-A
stopWatch.start("任务-A");		
stopWatch.stop();
// 计时,指定任务-B
stopWatch.start("任务-B");
stopWatch.stop();
// 格式打印输出
stopWatch.prettyPrint();
```



## 启动加载器

在框架启动后, 进行某些操作

### 示例 - 自定义启动加载器

通过实现 `CommandLineRunner` 和 `ApplicationRunner` 接口进行定义, 可以使用 `@Order` 注解进行排序,相同排序则 `ApplicationRunner` 接口优先.

#### 实现 `CommandLineRunner` 接口

实现 `org.springframework.boot.CommandLineRunner` 接口中的方法, 重写 `run` 方法, 在程序完成后进行某些操作. `args` 为启动时传入的命令参数

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

/**
 *  自定义容器启动器,在容器启动后执行
 *      Order 注解 为启动器进行排序
 * @author Jion
 */
@Order(1)
@Component
public class WebApplicationCommandLineRunnerFirst implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println("--- 方法一 ---");
        System.out.println("启动加载器: 容器启动成功...");
        System.out.println("------");
    }
}
```

#### 实现 `ApplicationRunner` 接口

实现 `org.springframework.boot.ApplicationRunner` 接口中的方法, 重写 `run` 方法, 在Spring框架完成后进行操作.  `context` 为当前容器, `args` 为框架封装的启动参数

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

/**
 *  自定义容器启动器,在容器启动后执行
 *      优先执行
 * @author Jion
 */
@Order(1)
@Component
public class WebApplicationCommandLineRunnerSecond implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("--- 方法二 ---");
        System.out.println("启动加载器: 容器启动成功...");
        System.out.println("------");
    }
}
```



### 源码步骤

#### 2-15-0-0 调用

在 `run` 方法中, 通过 `callRunners(context, applicationArguments)` 方法, 对启动扩展加载器进行调用.


```java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
    // 集合
    List<Object> runners = new ArrayList<>();
    // ApplicationRunner 接口实现
    runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
    // CommandLineRunner 接口实现
    runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
    // 对其排序
    AnnotationAwareOrderComparator.sort(runners);
    // 进行调用
    for (Object runner : new LinkedHashSet<>(runners)) {
        // ApplicationRunner 接口先, 传入 ApplicationArguments 参数对象
        if (runner instanceof ApplicationRunner) {
            callRunner((ApplicationRunner) runner, args);
        }
        // CommandLineRunner 接口后, 传入 args 参数数组
        if (runner instanceof CommandLineRunner) {
            callRunner((CommandLineRunner) runner, args);
        }
    }
}
```



## 环境属性

### 属性配置

#### 配置先后

| 顺序 | 方式                                              | 示例                                                         |
| ---- | ------------------------------------------------- | ------------------------------------------------------------ |
| 1    | `Devtools` 全局属性                               |                                                              |
| 2    | 测试环境 `@TestPropertySource` 注解               | `@TestPropertySource("classpath:属性文件.properties")`       |
| 3    | 测试环境 `properties` 属性                        | 通过在 `@SpringBootTest` 注解中, 为属性 `properties` 添加 `json` 格式的参数对象,完成配置 |
| 4    | 命令行参数                                        |                                                              |
| 5    | `SPRING_APPLICATION_JSON` 属性                    | 通过将 `json`  格式的参数以 `SPRING_APPLICATION_JSON` 为参数名传入, 完成配置 |
| 6    | `ServletConfig` 初始化参数                        |                                                              |
| 7    | `ServletContext` 初始化参数                       |                                                              |
| 8    | `JNDI` 属性                                       |                                                              |
| 9    | `JAVA` 系统虚拟机属性                             | 通过设置 `JVM` 虚拟机参数                                    |
| 10   | 操作系统环境变量                                  |                                                              |
| 11   | `RandomValuePropertySource` 随机属性              | `{random.int[20,30]}` 属性配置                               |
| 12   | jar包外的 `application-{profile}.properties` 文件 | 默认使用 `default` 配置文件                                  |
| 13   | jar包内的 `application-{profile}.properties` 文件 |                                                              |
| 14   | jar包外的 `application.properties` 文件           |                                                              |
| 15   | jar包外的 `application.properties` 文件           |                                                              |
| 16   | `@PropertySource` 绑定配置                        | `@PropertySource("classpath:属性文件.properties")`           |
| 17   | 默认属性                                          | ```java Properties properties = new Properties(); properties.setProperty("author","Jion17"); springApplication.setDefaultProperties(properties); ``` |




## `Aware` 接口

 在需要使用 `Spring` 容器中的相关功能时, 调用 `org.springframework.beans.factory.Aware` 接口及其相关实现

### 常用接口

| 接口                             | 作用                     | 作用位置    |
| -------------------------------- | ------------------------ | ----------- |
| `BeanNameAware`                  | 获取容器中 `bean` 的名称 |             |
| `BeanClassLoaderAware`           | 获得类加载器             |             |
| `BeanFactoryAware`               | 获得 `bean` 创建工厂     | `bean` 创建 |
| `EnvironmentAware`               | 获得环境变量             | `bean` 创建 |
| `EmbeddedValueResolverAware`     | 获得容器加载属性文件的值 | `bean` 创建 |
| `ResourceLoaderAware`            | 获得资源加载器           | `bean` 创建 |
| `ApplicationEventPublisherAware` | 获得应用事件发布器       | `bean` 创建 |
| `MessageSouceAware`              | 获得文本信息, 用以国际化 | `bean` 创建 |
| `ApplicationContextAware`        | 获得当前应用上下文       | `bean` 创建 |

### 调用位置

`doCreateBean` -> `initializeBean` -> `invokeAwareMethods` -> `applyBeanPostProcessorsBeforeInitialization` -> `ApplicationContextAwareProcessor`

初始化 `Bean` 在方法 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean` 中

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
            // 调用
            invokeAwareMethods(beanName, bean);
            return null;
        }, getAccessControlContext());
    }
    else {
        // 调用
        invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 2. 调用一些 aware 接口
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex);
    }
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}
```

在 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods` 中调用

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
    if (bean instanceof Aware) {
        if (bean instanceof BeanNameAware) {
            ((BeanNameAware) bean).setBeanName(beanName);
        }
        if (bean instanceof BeanClassLoaderAware) {
            ClassLoader bcl = getBeanClassLoader();
            if (bcl != null) {
                ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
            }
        }
        if (bean instanceof BeanFactoryAware) {
            ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
        }
    }
}
```



`org.springframework.context.support.ApplicationContextAwareProcessor#postProcessBeforeInitialization` 方法, 进行调用

```java
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    if (!(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
          bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
          bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)){
        return bean;
    }

    AccessControlContext acc = null;

    if (System.getSecurityManager() != null) {
        acc = this.applicationContext.getBeanFactory().getAccessControlContext();
    }

    if (acc != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
            invokeAwareInterfaces(bean);
            return null;
        }, acc);
    }
    else {
        // 调用 aware 接口
        invokeAwareInterfaces(bean);
    }
    return bean;
}
// 具体的调用
private void invokeAwareInterfaces(Object bean) {
    if (bean instanceof EnvironmentAware) {
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    if (bean instanceof MessageSourceAware) {
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
}
```



### 示例 - 自定义 `Aware` 接口

自定义一个  `aware` 接口, 被用来继承,以期实现传入指定封装类.

#### 自定义封装类

自定义封装类, 用 `@Component` 注入容器中, 以便可以直接通过容器获得

```java
import org.springframework.stereotype.Component;

/**
 *  自定义类, 使用 aware 获取
 * @author Jion
 */
@Component
public class WebApplicationFlag {

    /** 是否Web应用 */
    private Boolean isWebApplicationFlag = Boolean.FALSE;

    public Boolean getWebApplicationFlag() {
        return isWebApplicationFlag;
    }

    public void setWebApplicationFlag(Boolean webApplicationFlag) {
        isWebApplicationFlag = webApplicationFlag;
    }
}

```



#### 自定义 `aware` 接口

通过实现 `Aware` 接口, 并自定义方法

```java
import org.springframework.beans.factory.Aware;

/**
 *  自定义的 Aware 接口, 用来 从容器中获得类
 * @see WebApplicationAwareProcess 设置过程
 * @see WebApplicationFlag 设置对象
 * @author Jion
 */
public interface WebApplicationAware extends Aware {

    /** 自定义, 将当容器对象设置到实现方法中 */
    void setWebApplicationFlag(WebApplicationFlag webApplicationFlag);
}
```



#### 自定义后置处理器

通过实现 `org.springframework.beans.factory.config.BeanPostProcessor` 接口,判断类型后将其类型转换,调用相关方法

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.Aware;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.stereotype.Component;

/**
 *  对自定义 aware 接口进行后置处理
 * @author Jion
 */
@Component
public class WebApplicationAwareProcessor implements BeanPostProcessor {

    /** 上下文容器 */
    private final ConfigurableApplicationContext context;

    public WebApplicationAwareProcessor(ConfigurableApplicationContext context) {
        this.context = context;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(bean instanceof Aware){
            if(bean instanceof WebApplicationAware){
                // 从上下文容器中获得, 并封装为 自定义对象
                WebApplicationFlag flag = context.getBean(WebApplicationFlag.class);
                String property = context.getEnvironment().getProperty("web-application");
                flag.setWebApplicationFlag(Boolean.parseBoolean(property));
                // 调用接口实现的方法
                ((WebApplicationAware) bean).setWebApplicationFlag(flag);
            }
        }
        // 一定要返回 Bean
        return bean;
    }
}
```



#### 尝试获取

通过继承,完成 `set` 方法, 获得接口中的对象

```java
import org.springframework.stereotype.Component;

/**
 * @author Jion
 */
@Component
public class MyWebApplicationAware implements WebApplicationAware {

    private WebApplicationFlag webApplicationFlag;

    @Override
    public void setWebApplicationFlag(WebApplicationFlag webApplicationFlag) {
        this.webApplicationFlag = webApplicationFlag;
    }

    /** 通过 aware 接口获得 */
    public Boolean getFlag(){
        return webApplicationFlag.getWebApplicationFlag();
    }
}

```



#### 测试用例

在 `@SpringBootTest(properties = {"web-application:true"})` 中, 注入环境属性

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * @author Jion
 */
@RunWith(SpringRunner.class)
@SpringBootTest(properties = {"web-application:true"})
public class MyWebApplicationAwareTest{

    @Autowired
    MyWebApplicationAware myWebApplicationAware;

    @Test
    public void getFlag() {
        Boolean flag = myWebApplicationAware.getFlag();
        System.out.println("通过Aware接口获得:" + flag);
    }
}
```



## `Environment` 环境

在 `Environment` 对象中,有多个 `propertySources` , 每个 `propertySources` 封装了来自框架内外的参数信息.从中获得相关的属性.



![Environment类图](./Spring全家桶-SpringBoot源码篇/Environment类图-1.png)

### 源码步骤

#### 2-6-0-0 配置环境信息

在 `run` 方法中 , 通过 `ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments)` 方法构建上下环境信息.

``` java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
                                                   ApplicationArguments applicationArguments) {
    // 1. 创建一个环境信息
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    // 2. 配置环境信息
    configureEnvironment(environment, applicationArguments.getSourceArgs());
    // 3. 添加可以配置的源 configurationProperties
    ConfigurationPropertySources.attach(environment);
    // 4. 发送一个事件
    listeners.environmentPrepared(environment);
    // 5. 绑定上下文对象属性
    bindToSpringApplication(environment);
    // 6. 转换场景,不同场景下配置对象不同,转为不同的环境配置
    if (!this.isCustomEnvironment) {
        environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
                                                                                               deduceEnvironmentClass());
    }
    // 7. 配置属性源
    ConfigurationPropertySources.attach(environment);
    return environment;
}
```



#### 2-6-1-0 创建环境信息

```java
private ConfigurableEnvironment getOrCreateEnvironment() {
    // 一般为 null
    if (this.environment != null) {
        return this.environment;
    }
    // 根据环境信息
    switch (this.webApplicationType) {
        case SERVLET:
            // 一般是这个..
            return new StandardServletEnvironment();
        case REACTIVE:
            return new StandardReactiveWebEnvironment();
        default:
            return new StandardEnvironment();
    }
}
```

#### 2-6-1-1 配置环境信息源

`StandardServletEnvironment`  继承自 `StandardEnvironment` , 而 `StandardEnvironment` 再继承自 `AbstractEnvironment`  类, 在该类的构造方法中,  回调方法 `customizePropertySources` 其具体实现为对应子类的方法.(最终子类实现)

```java
public AbstractEnvironment() {
    // 自义定属性来源
    customizePropertySources(this.propertySources);
}

// 交由子类实现
protected void customizePropertySources(MutablePropertySources propertySources) {}
```

在子类 `org.springframework.web.context.support.StandardServletEnvironment` 中, 有具体实现.

```java
@Override
protected void customizePropertySources(MutablePropertySources propertySources) {
    // 添加源 servletConfigInitParams
    propertySources.addLast(new StubPropertySource(SERVLET_CONFIG_PROPERTY_SOURCE_NAME));
    // 添加源 servletContextInitParams
    propertySources.addLast(new StubPropertySource(SERVLET_CONTEXT_PROPERTY_SOURCE_NAME));
    //是否为JNDI环境
    if (JndiLocatorDelegate.isDefaultJndiEnvironmentAvailable()) {
        // 添加源 jndiProperties
        propertySources.addLast(new JndiPropertySource(JNDI_PROPERTY_SOURCE_NAME));
    }
    // 调用父类的方法
    super.customizePropertySources(propertySources);
}
```

在父类  `org.springframework.core.env.StandardEnvironment#customizePropertySources` 方法中.添加属性源

```java
@Override
protected void customizePropertySources(MutablePropertySources propertySources) {
    propertySources.addLast(
        // 添加源 systemProperties 
        new PropertiesPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
    propertySources.addLast(
        // 添加源 systemEnvironment
        new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
}
```

#### 2-6-2-0 配置参数

```java
protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
    // 属性转换
    if (this.addConversionService) {
        ConversionService conversionService = ApplicationConversionService.getSharedInstance();
        environment.setConversionService((ConfigurableConversionService) conversionService);
    }
    // 1. 配置启动参数
    configurePropertySources(environment, args);
    // 2. 
    configureProfiles(environment, args);
}
```

#### 2-6-2-1 配置启动参数属性

```java
protected void configurePropertySources(ConfigurableEnvironment environment, String[] args) {
    MutablePropertySources sources = environment.getPropertySources();
    // 添加默认的属性, 在 Application 中通过 setDefaultProperties 设置
    if (this.defaultProperties != null && !this.defaultProperties.isEmpty()) {
        sources.addLast(new MapPropertySource("defaultProperties", this.defaultProperties));
    }
    // 命令行参数属性赋值
    if (this.addCommandLineProperties && args.length > 0) {
        // commandLineArgs
        String name = CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME;
        if (sources.contains(name)) {
            PropertySource<?> source = sources.get(name);
            CompositePropertySource composite = new CompositePropertySource(name);
            composite.addPropertySource(
                new SimpleCommandLinePropertySource("springApplicationCommandLineArgs", args));
            composite.addPropertySource(source);
            sources.replace(name, composite);
        }
        else {
            // 添加命令行属性参数 --开头, =分割
            sources.addFirst(new SimpleCommandLinePropertySource(args));
        }
    }
}
```

#### 2-6-2-2 配置激活属性

```java
protected void configureProfiles(ConfigurableEnvironment environment, String[] args) {
    Set<String> profiles = new LinkedHashSet<>(this.additionalProfiles);
    profiles.addAll(Arrays.asList(environment.getActiveProfiles()));
    environment.setActiveProfiles(StringUtils.toStringArray(profiles));
}
```

#### 2-6-4-0 发布事件

在 `org.springframework.boot.SpringApplicationRunListeners#environmentPrepared`  方法中, 发布 `ApplicationEnvironmentPreparedEvent` 事件,该事件包含 `ConfigurableEnvironment` 配置对象,以便监听者可以对其进行修改.

```java
void environmentPrepared(ConfigurableEnvironment environment) {
    for (SpringApplicationRunListener listener : this.listeners) {
        // 获得 SpringApplicationRunListener 包装类, 并调用方法, 调用每一个监听器的 environmentPrepared() 方法
        listener.environmentPrepared(environment);
    }
}
```

在监听器中,通过 `org.springframework.context.event.SimpleApplicationEventMulticaster#multicastEvent` 方法,调用广播器进行广播,.

```java
@Override
public void environmentPrepared(ConfigurableEnvironment environment) {
    this.initialMulticaster
        .multicastEvent(new ApplicationEnvironmentPreparedEvent(this.application, this.args, environment));
}
@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
    Executor executor = getTaskExecutor();
    // 获得对该事件感兴趣的监听器, 并调用其方法
    for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
        if (executor != null) {
            executor.execute(() -> invokeListener(listener, event));
        }
        else {
            invokeListener(listener, event);
        }
    }
}
```

其中的广播, `org.springframework.boot.context.config.ConfigFileApplicationListener#onApplicationEvent` 方法在收到该事件后, 对环境配置进行修改.
具体通过`SpringFactoriesLoader` 加载 `META-INF/spring.factories`  中接口 `org.springframework.boot.env.EnvironmentPostProcessor` 的实现类(实现 `Order` 接口, 定义具体属性重载排序), 并调用从而实现对配置信息的修改
常用的有 `SpringApplicationJsonEnvironmentPostProcessor`  (添加 `SPRING_APPLICATION_JSON` 属性集), `CloudFoundryVcapEnvironmentPostProcessor` (添加 `vcap` 属性集), `

```java
// 监听事件
@Override
public void onApplicationEvent(ApplicationEvent event) {
    if (event instanceof ApplicationEnvironmentPreparedEvent) {
        // 容器环境准备
        onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent) event);
    }
    if (event instanceof ApplicationPreparedEvent) {
        onApplicationPreparedEvent(event);
    }
}

// 容器环境备注
private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
    // 通过 META-INF/spring.factories 加载 EnvironmentPostProcessor 实现类
    List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
    postProcessors.add(this);
    // 排序
    AnnotationAwareOrderComparator.sort(postProcessors);
    for (EnvironmentPostProcessor postProcessor : postProcessors) {
        // 修改环境变量
        postProcessor.postProcessEnvironment(event.getEnvironment(), event.getSpringApplication());
    }
}

// 通过 META-INF/spring.factories 加载 EnvironmentPostProcessor 实现类
List<EnvironmentPostProcessor> loadPostProcessors() {
    return SpringFactoriesLoader.loadFactories(EnvironmentPostProcessor.class, getClass().getClassLoader());
}
```

#### 2-6-5-0 绑定上下文对象属性

将 `spring.main.*` 的属性 绑定到 `SpringApplication` 对象的属性

```java
protected void bindToSpringApplication(ConfigurableEnvironment environment) {
    try {
        // spring.main.* 的属性 绑定到 SpringApplication 对象
        Binder.get(environment).bind("spring.main", Bindable.ofInstance(this));
    }
    catch (Exception ex) {
        throw new IllegalStateException("Cannot bind to SpringApplication", ex);
    }
}
```

#### 2-6-6-0 转换场景环境配置

```java
StandardEnvironment convertEnvironmentIfNecessary(ConfigurableEnvironment environment,
                                                  Class<? extends StandardEnvironment> type) {
    // 将 ConfigurableEnvironment 转为具体的环境配. 如 StandardEnvironment, StandardServletEnvironment,StandardReactiveWebEnvironment
    if (type.equals(environment.getClass())) {
        return (StandardEnvironment) environment;
    }
    // 不存在,则转为 标准的 StandardEnvironment
    return convertEnvironment(environment, type);
}

// 转为 StandardEnvironment , 拷贝属性,并返回
private StandardEnvironment convertEnvironment(ConfigurableEnvironment environment,
                                               Class<? extends StandardEnvironment> type) {
    StandardEnvironment result = createEnvironment(type);
    result.setActiveProfiles(environment.getActiveProfiles());
    result.setConversionService(environment.getConversionService());
    copyPropertySources(environment, result);
    return result;
}
```



#### 2-6-7-0 可配置的源

```java
public static void attach(Environment environment) {
    Assert.isInstanceOf(ConfigurableEnvironment.class, environment);
    MutablePropertySources sources = ((ConfigurableEnvironment) environment).getPropertySources();
    // 配置源 configurationProperties
    PropertySource<?> attached = sources.get(ATTACHED_PROPERTY_SOURCE_NAME);
    // 一般存在,默认删除
    if (attached != null && attached.getSource() != sources) {
        sources.remove(ATTACHED_PROPERTY_SOURCE_NAME);
        attached = null;
    }
    // 不存在, 添加默认的
    if (attached == null) {
        sources.addFirst(new ConfigurationPropertySourcesPropertySource(ATTACHED_PROPERTY_SOURCE_NAME,
                                                                        new SpringConfigurationPropertySources(sources)));
    }
}

```





## `profile` 解析





# 附录







