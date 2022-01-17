---
title: Spring全家桶-SpringBoot源码篇
abbrlink: 339b7cd
typora-root-url: ../../
date: 2020-06-23 22:43:34
categories:
  - Java
  - Spring
tags: [Java, Spring]
---

> SpringBoot源码梳理

<!--more-->



# 简介

介绍SpringBoot的启动原理, 场景启动器, Bean管理等...



## 环境信息

- JDK14
- spring-boot 2.3.X
- idea



# 启动流程

1. 框架初始化  (`SpringApplication` 对象初始化)
2. 框架启动 (`run` 方法执行)
3. 自动化装配 (`autoconfig` 的具体执行)



## 框架初始化

1. 配置资源加载器
2. 配置 `primarySources` , 一般为启动类
3. 应用环境监测. (标准环境; Web环境; 响应式Web环境)
4. 配置系统初始化器
5. 配置应用监听器.
6. 配置 `main` 方法所在类, 作为扫描入口



## 启动框架

1. 计时器启动
2. `Headless` 模式赋值  (没有显示器,键盘环境)
3. **发送 `ApplicationStartingEven` 事件**, 
4. 配置环境模块, 配置属性
5. **发送 `ApplicationEnvironmentPreparedEvent` 事件**
6. 打印 `banner` . 字符画图标
7. 创建应用上下文对象. (`ConfigurableApplicationContext` 对象)
8. 初始化失败分析器
9. 关联 `SpringBoot` 组件与应用上下文对象
10. **发送 `ApplicationContextInitializedEvent` 事件**
11. 加载 `source` 到 `context`
12. **发送 `ApplicationPreparedEvent` 事件**
13. 刷新上下文
14. 计时器停止计数
15. **发送 `ApplicationStartedEvent` 事件**
16. 调用框架启动扩展类
17. **发送 `ApplicationReadyEvent` 事件**
18. 如果期间抛出了异常,  **发出 `ApplicationFailedEvent` 事件**



## 框架自动化装配步骤

1. 收集配置文件中的配置工厂类
2. 加载组件工厂
3. 注册组件内定义的 `Bean`



# 框架初始化

## 工厂类加载器

### 作用

`org.springframework.core.io.support.SpringFactoriesLoader` 工厂类加载器

- 通用工厂加载机制.
- 在 `META-INF/spring.factories`  从 `classpath` 下多个 `jar` 内的特定位置读取文件并初始化类.
- 其配置文件必须为 `properties` 文件, `key` 为全限定名, `value` 为具体的实现,可以有多个,并用逗号分隔.



### 加载步骤

1. 判断缓存中是否存在,如果存在则返回.否则继续
2. 读取指定资源文件 `META-INF/spring.factories`
3. 构造 `properties` 对象
4. 获取指定 `key` 对应的 `value` 值
5. 逗号分隔到 `value`
6. 保存结果到缓存
7. 依次实例化到结果对象
8. 对结果进行排序
9. 调用反射创建实例
10. 返回实例结果



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

#### 1-2-4-0 加载容器初始化器

通过传入 `org.springframework.context.ApplicationContextInitializer` 的接口类型,加载其具体实现

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}
```

这里主要有三步

1. 使用加载器加载具体实现类的全限定名
2. 创建工厂具体实例对象
3. 对其进行排序. 如使用 `@Order` 注解

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

#### 1-2-4-2 获得工厂类及其实例

具体的加载原理如下.
首先尝试从缓存中加载,如果能获得到需要的接口及其实现类,则返回结束.
缓存在装填时默认会读取全部 `jar` 中的配置工厂信息解析并存入

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
        throw new IllegalArgumentException("Unable to load factories from location [" + FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}
```

#### 1-2-4-2 创建工厂类的实例对象

通过 `names` 传入 `Set<String>` 集合, 集合中包括了要创建的实例对象的全限定名. 调用 `BeanUtils.instantiateClas` 方法传入类的构造器及参数,创建实例对象.
**通过构造器反射创建实例, 建议工厂类必须含有无参数的构造类**

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



## 系统初始化器

实现接口 `org.springframework.context.ApplicationContextInitializer`
在容器刷新 `refresh` 方法前执行函数回调, 以便向容器中注入初始化器.
通过继承接口并各自实现.

### 作用

- 上下文刷新 `refresh` 方法前调用
- 编码设置一些属性变量.通常在 web 环境中
- 可以通过 `order` 接口进行自定义排序



### 示例 - 自定义初始化器

其中,  `@Order` 注解中的值越小,则越先执行



####  工厂文件注册

首先通过实现 `org.springframework.context.ApplicationContextInitializer` 接口中的初始化方法,随后在项目创建 `META-INF/spring.factories` 文件, 注册初始化器

**实现接口**

```java
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.annotation.Order;
import org.springframework.core.env.ConfigurableEnvironment;

/**
 * 自定义容器启动器.通过 META-INF/spring.factories 注册
 * 在容器注入时,进行操作
 *
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

实现 `org.springframework.context.ApplicationContextInitializer` 接口,并在启动类中 `springApplication.addInitializers` 方法中注册初始化器

**实现接口**

```java
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.annotation.Order;
import org.springframework.core.env.ConfigurableEnvironment;

/**
 * 自定义容器启动器. 通过启动类注册
 * 在容器注入时,进行操作
 *
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
 * 主启动类
 *
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

通过在 `application.yml` 文件中, 通过配置 `context.initializer.classes` 属性指定.

**实现接口**

```java
package top.jionjion.initializer;

import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.annotation.Order;
import org.springframework.core.env.ConfigurableEnvironment;

/**
 * 自定义容器启动器. 通过属性 context.initializer.classes 指定调用
 * 在容器注入时,进行操作
 * 
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

**配置文件 **

```yaml
context:
  initializer:
    classes: top.jionjion.initializer.WebApplicationInitializerThird
```



### 初始化器调用位置

#### 2-10-0-0 `prepareContext` 方法

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
        // 准备容器上下文,执行
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

这里, 断言进行判断. 只有在当前容器为泛型中定义容器子类的对象才有必要进行初始化
如: `implements ApplicationContextInitializer<ConfigurableApplicationContext>`

```java
protected void applyInitializers(ConfigurableApplicationContext context) {
    for (ApplicationContextInitializer initializer : getInitializers()) {
        // 断言,只有该容器对象类为其泛型的子类时,才能进行初始化操作
        Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(), ApplicationContextInitializer.class);
        Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
        initializer.initialize(context);
    }
}
```

#### 扩展

定义在 `spring.factories` 文件中的初始化器被 `SpringFactoriesLoader` 发现并注册

`SpringApplication` 初始化后再手动添加注册

通过备配置文件注册的初始化器,会由 `org.springframework.boot.context.config.DelegatingApplicationContextInitializer` 类进行注册调用, 该类由Spring加载 `spring-boot-2.4.0.jar!\META-INF\spring.factories` 文件获得, 其配置优先级最高(默认`order`为0),会使其属性文件 `context.initializer.classes` 定义的初始化器最先调用

**`DelegatingApplicationContextInitializer` 类**

```java
// 默认实现初始化器,由容器进行加载
public class DelegatingApplicationContextInitializer
		implements ApplicationContextInitializer<ConfigurableApplicationContext>, Ordered {
	// 定义属性
    private static final String PROPERTY_NAME = "context.initializer.classes";
	// 默认排序
	private int order = 0;
	
    // 初始化方法
	@Override
	public void initialize(ConfigurableApplicationContext context) {
        // 1.获得环境中 context.initializer.classes 属性的实现类
		ConfigurableEnvironment environment = context.getEnvironment();
		List<Class<?>> initializerClasses = getInitializerClasses(environment);
        // 2.调用其初始化方法
		if (!initializerClasses.isEmpty()) {
			applyInitializerClasses(context, initializerClasses);
		}
	}
    
    // 1.获得环境中 context.initializer.classes 属性的实现类
	private List<Class<?>> getInitializerClasses(ConfigurableEnvironment env) {
		String classNames = env.getProperty(PROPERTY_NAME);
		List<Class<?>> classes = new ArrayList<>();
		if (StringUtils.hasLength(classNames)) {
			for (String className : StringUtils.tokenizeToStringArray(classNames, ",")) {
				classes.add(getInitializerClass(className));
			}
		}
		return classes;
	}    
    
    // 2.调用其初始化方法
	private void applyInitializerClasses(ConfigurableApplicationContext context, List<Class<?>> initializerClasses) {
		Class<?> contextClass = context.getClass();
		List<ApplicationContextInitializer<?>> initializers = new ArrayList<>();
		for (Class<?> initializerClass : initializerClasses) {
			initializers.add(instantiateInitializer(contextClass, initializerClass));
		}
		applyInitializers(context, initializers);
	}  
}
```



## 监听/广播器

### 监听器/广播模式 

- 事件
- 监听器
-  广播器
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

### 监听器调用者

隔离监听器内部实现与容器调用的接口, 容器仅需调用该接口方法, 无需关注监听器内部.

```java
public interface SpringApplicationRunListener {

	// 框架启动
	default void starting(ConfigurableBootstrapContext bootstrapContext) {
		starting();
	}

	// 环境准备完成, 已经加载系统配置属性
	default void environmentPrepared(ConfigurableBootstrapContext bootstrapContext,
			ConfigurableEnvironment environment) {
	}

	// 容器准备完成事件, 上下文对象容器完成 ,尚未完全加载完 Bean 定义
	default void contextPrepared(ConfigurableApplicationContext context) {
	}

    // 容器加载完成, 尚未刷新容器
	default void contextLoaded(ConfigurableApplicationContext context) {
	}

	// 容器刷新完成, Bean 已经实例化完成,尚未调用 run 扩展方法
	default void started(ConfigurableApplicationContext context) {
	}

	// 容器正常运行, 已经调用 run 扩展方法
	default void running(ConfigurableApplicationContext context) {
	}

	// 容器失败
	default void failed(ConfigurableApplicationContext context, Throwable exception) {
	}

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
3.  `ApplicationContextInitializedEvent` 容器初始化完成事件,  `SpringBoot` 中已经准备好上下文容器, 尚未装载`Bean` 对象定义
4.  `ApplicationPreparedEvent` 容器准备完成事件, 上下文对象容器完成,尚未完全加载完 `Bean`
5.  `ApplicationStartedEvent` 容器启动完成事件,  `Bean` 已经装载完成,尚未调用 `run` 扩展方法
6.  `ApplicationReadyEvent` 容器准备完成事件
7.  `ApplicationFailedEvent` 容器失败事件

![系统事件](./images/2021-02/系统事件-1.png)



### 加载步骤

1. 获得监听器列表
2. 尝试从缓存中读取监听器列表,不存在则加载
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
    // 16. 异常处理
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    // 16. 异常处理
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

### 配置方式

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

- 使用 `@Component` 注解, 或者 `@Controller` ; `@Service` ; `@Repository` 注解
- 配置类中使用 `@Bean` , 配合 `@Configuration` 使用
- 实现 `FactoryBean` 接口 , 可能需要配合 `@Qualifier` 指定 `Bean` 的名字
- 实现 `BeanDefinitionRegistryPostProcessor` 接口, 在容器实例化 `Bean` 前向容器注入 `BeanDefinition` 定义
- 实现 `ImportBeanDefinitionRegistrar` 接口 , 配合 `@Import` 被扫描注入

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
 * 测试 @Component 注入Bean
 *
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
 * 注解配置文件
 *
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
 * 测试,通过 @Configuration 结合 @Bean 注入Bean
 *
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

    /** 获得动物名称 */
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
 * 使用 FactoryBean<T> 注入Bean, 通过实例工厂注入容器
 *
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
 * 测试 FactoryBean<T> 注入Bean
 *
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

`org.springframework.beans.factory.support.RootBeanDefinition` 类为 `org.springframework.beans.factory.config.BeanDefinition` 接口的默认实现类.类似的还有 `GenericBeanDefinition` 和 `ChildBeanDefinition` 类及其子类

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.stereotype.Component;

/**
 * 使用 BeanDefinitionRegistryPostProcessor 在实例化前,向容器注入Bean定义,完成注入 Bean
 *
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
 * 测试 BeanDefinitionRegistryPostProcessor 注入Bean
 *
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
 * 使用 ImportBeanDefinitionRegistrar 注入Bean.
 * 常用配置 @Import 注解, 在自定义注解中,将当注解注入.同时携带自定义注解中的元信息, 用来向容器中动态添加Bean
 *
 * @author Jion
 */
@Component
public class DuckImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    
    /**
     * 向容器注入Bean的定义
     * GenericBeanDefinition为常用的Bean定义
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        // importingClassMetadata 在 @Import 注解上的类的注解信息.

        // 注入Bean
        GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
        genericBeanDefinition.setBeanClass(Duck.class);
        // 注入Bean名和定义
        registry.registerBeanDefinition("duck", genericBeanDefinition);
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
 * 使用 ImportBeanDefinitionRegistrar 注入Bean
 *
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

![类图信息](./images/2021-02/BeanDefinition类图-1.png)



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

`org.springframework.context.support.AbstractApplicationContext#refresh` 容器刷新方法

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



#### 2-11-1-X 准备刷新

##### `prepareRefresh` 

1. 设置容器状态
2. 环境必备属性检查
3. 初始化 `Servlet` 相关信息
4. 储存早期监听器及其事件

#### 2-11-1-0 准备刷新

主要进行 容器状态设置,  必备属性检查 ,初始化属性设置(Web环境, 储存应用监听器及其事件)

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

    // 1. 初始化环境上下文信息,如果是Web环境,初始化 ServletConfig 和 ServletContext 信息
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

    // 初始化 属性 earlyApplicationEvents , 用来存放刷新前的广播事件
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
    // 当前环境=> StandardServletEnvironment    
    if (env instanceof ConfigurableWebEnvironment) {
        // 初始化 Servlet信息, 传入 null
        ((ConfigurableWebEnvironment) env).initPropertySources(this.servletContext, null);
    }
}
```

初始化 `ServletContext` 和 `ServletConfig` 信息, 传入 `null` 并不会实际调用. 

```java
@Override
public void initPropertySources(@Nullable ServletContext servletContext, @Nullable ServletConfig servletConfig) {
    WebApplicationContextUtils.initServletPropertySources(getPropertySources(), servletContext, servletConfig);
}
```

#### 2-11-2-X 获取 `BeanFactory`

##### `obtainFreshBeanFactory`

1. 设置容器刷新状态
2. 设置 `BeanFactory` 的序列化 ID
3. 获取 ``ConfigurableListableBeanFactory` 实例 

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



#### 2-11-2-1 刷新状态

设置容器刷新状态; 设置序列化ID

```java
protected final void refreshBeanFactory() throws IllegalStateException {
    // 设置状态,标识容器正在刷新. 乐观锁
    if (!this.refreshed.compareAndSet(false, true)) {
        throw new IllegalStateException(
            "GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
    }
    // 设置序列化ID 默认是application, 如果是实例化过,则为 org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@5e1fa5b1
    this.beanFactory.setSerializationId(getId());
}
```



#### 2-11-2-2 返回实例

调用子类的 `org.springframework.context.support.GenericApplicationContext#getBeanFactory` 方法,返回 `org.springframework.beans.factory.support.DefaultListableBeanFactory` 实例

```java
	private final DefaultListableBeanFactory beanFactory;	

	@Override
	public final ConfigurableListableBeanFactory getBeanFactory() {
		return this.beanFactory;
	}
```



#### 2-11-3-X 初始化 `BeanFactory`

##### `prepareBeanFactory`

1. 设置类加载器
2. 设置SpEL表达式的解析器
3. 设置后置处理器
4. 忽略 `Aware` 接口的依赖关系
5. 注册一些组件

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

    // 设置忽略Bean的后置处理器
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    // 忽略自动装配的接口的依赖关系
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // 添加解析依赖关系, 当用到下述类作为依赖对象时,指向自身.
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

    // 是否包含 environment 的Bean, 表示当前环境信息
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        // 如果不存在, 注册当前环境信息作为Bean 
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    // 是否包含 systemProperties 的Bean, 不存在注入
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    // 是否包含 systemEnvironment 的Bean, 不存在注入
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

#### 2-11-4-X 设置 `BeanFactory` 后置处理器

##### `postProcessBeanFactory`

1. 子类重写,以便对 `BeanFactory` 作更进一步设置
2. `Web` 环境中添加相关作用域设置, ` Web` 相关组件

#### 2-11-4-0 设置 `BeanFactory` 后置处理器

调用子类 `org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext#postProcessBeanFactory` 方法进行设置

一般由子类重写,在`BeanFactory` 完成创建后进一步作设置, 如 `Web` 相关的一些组件

```java
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 1. 设置 ServletContextAwareProcessor 后置处理器及忽略装配依赖
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



#### 2-11-4-1 设置 `Web` 相关后置处理器

`ServletContextAwareProcessor`  后置处理器及忽略装配依赖

```java
@Override
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 设置后置处理器
    beanFactory.addBeanPostProcessor(new WebApplicationContextServletContextAwareProcessor(this));
    // 忽略装配依赖的接口
    beanFactory.ignoreDependencyInterface(ServletContextAware.class);
    // 注册 Web 应用的作用域
    registerWebApplicationScopes();
}

// 注册 Web 应用的作用域
private void registerWebApplicationScopes() {
    ExistingWebApplicationScopes existingScopes = new ExistingWebApplicationScopes(getBeanFactory());
    // 2. 设置 Web 作用域及环境配置信息
    WebApplicationContextUtils.registerWebApplicationScopes(getBeanFactory());
    existingScopes.restore();
}


```



#### 2-11-4-2 设置 `Web` 作用域及环境配置信息

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

    // 设置解析依赖
    beanFactory.registerResolvableDependency(ServletRequest.class, new RequestObjectFactory());
    beanFactory.registerResolvableDependency(ServletResponse.class, new ResponseObjectFactory());
    beanFactory.registerResolvableDependency(HttpSession.class, new SessionObjectFactory());
    beanFactory.registerResolvableDependency(WebRequest.class, new WebRequestObjectFactory());
    if (jsfPresent) {
        FacesDependencyRegistrar.registerFacesDependencies(beanFactory);
    }
}
```



#### 2-11-5-X 初始化 `BeanFactory`

##### `invokeBeanFactoryPostProcessors`

1. 调用 `BeanDefinitionRegistryPostProcessor` 子接口的实现, 向容器中**添加** `BeanDefinition` 的定义信息
2. 调用 `BeanFactoryPostProcessor` 根接口的实现, 对容器内的已经有的 `BeanDefinition` 的定义信息进行**修改**

#### 2-11-5-0 获得 `BeanFactory` 的后置处理器

通过 `getBeanFactoryPostProcessors()`  方法获得上下文对象中的 `BeanFactory` 后置处理器, 在容器工厂创建后执行相关操作. 
上下文中的后置处理器在不同初始化器或者监听器启动运行阶段注入.

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



#### 2-11-5-1 调用 `BeanFactory` 后置处理器

主要有三个遍历

1. 首先遍历 `beanFactoryPostProcessors` 中的所有类,是否实现了 `BeanDefinitionRegistry` 接口 (用以向容器中注册 `Bean` ) , 
   如果未实现了, 则放入 `regularPostProcessors` 集合, 表示未注入
   如果实现了,则调用接口中的`postProcessBeanDefinitionRegistry` 方法, 向容器中添加 `Bean`, 并将该类放入 `registryProcessors` 集合,表示已注入

2. 随后再次遍历 `beanFactory` 中 `BeanDefinitionRegistryPostProcessor` 接口的实现(用以向容器中注册 `Bean`),
   其中实现类如果实现 `PriorityOrdered` 接口排序, 则将其添加到 `currentRegistryProcessors` 集合中,稍后对其排序, 并调用 `invokeBeanDefinitionRegistryPostProcessors` 方法初始化. 同时加入 `processedBeans` 集合表示该类已经被初始化.
   清空集合.
3. 再次遍历 `beanFactory` 中 `BeanDefinitionRegistryPostProcessor` 接口的实现(用以向容器中注册 `Bean`),
   其中实现类如果实现 `Ordered` 接口排序, 则将其添加到 `currentRegistryProcessors` 集合中,稍后对其排序, 并调用 `invokeBeanDefinitionRegistryPostProcessors` 方法初始化. 同时加入 `processedBeans` 集合表示该类已经被初始化.
   清空集合.
4. 最后为 ` while` 循环,  当仍有依赖尚未实现的 `BeanDefinitionRegistryPostProcessor` 接口实现类, 则通过循环,完成排序,初始化,清空集合.. 直到所有的 `BeanDefinitionRegistryPostProcessor` 接口实现类完成注入.

前两个遍历结束后,调用 `BeanFactoryPostProcessor` 接口的 `postProcessBeanFactory` 对注入后的 `Bean` 进行更多操作

剩余循环逻辑相似....

```java
/**
 * @param beanFactory 容器工厂
 * @param resobeanFactoryPostProcessorsurceLoader 上下文对象中,容器启动阶段由初始化器和监听器添加的容器工厂的后置处理器
 */
public static void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

    // 已经调用过后置处理器的Bean的名字的集合.无论 BeanDefinitionRegistryPostProcessors 子接口还是 BeanFactoryPostProcessor 父接口
    Set<String> processedBeans = new HashSet<>();
	    
    // 如果当前BeanFactory支持BeanDefinition注册及其注册后置处理 (先转为子类处理).
    if (beanFactory instanceof BeanDefinitionRegistry) {
        // 类型转换, BeanDefinitionRegistry 用来向容器中添加 BeanDefinition 定义信息及其后置处理
        BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
        // 普通的容器工厂后置处理器集合
        List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
        // 支持 BeanDefinition 定义的后置处理器集合
        List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();
		// 循环 容器启动阶段由初始化器和监听器添加的容器工厂的后置处理器
        for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
            // 如果支持处理 BeanDefinition ,则调用后置处理后加入集合
            if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
                // 1. 转为子类
                BeanDefinitionRegistryPostProcessor registryProcessor = (BeanDefinitionRegistryPostProcessor) postProcessor;
                // 2. 调用 BeanDefinition 后置处理
                registryProcessor.postProcessBeanDefinitionRegistry(registry);
                // 3. 加入支持 BeanDefinition 定义的后置处理器集合
                registryProcessors.add(registryProcessor);
            }
            // 普通的容器工厂后置处理器
            else {
                // 加入普通的容器工厂后置处理器集合
                regularPostProcessors.add(postProcessor);
            }
        }

        /* 
           不在这里初始化所有的工厂类(FactoryBeans)
           我们需要保留所有未初始化的常规类(Bean)，以使容器工厂后处理器对其应用!
           在实现 PriorityOrdered, Ordered 和其余优先级的 BeanDefinitionRegistryPostProcessor 之间分开。 
        */        
        // 当前,支持 BeanDefinition 定义的后置处理器集合
        List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

        // 首先, 从容器中获得BeanDefinition后置处理器(BeanDefinitionRegistryPostProcessor)接口的实现类
        String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            // 支持 PriorityOrdered 更高优先级的排序接口
            if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
                // 加入当前,支持 BeanDefinition 定义的后置处理器集合
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                // 加入 需要调用的后置处理器集合
                processedBeans.add(ppName);
            }
        }
        // 排序
        sortPostProcessors(currentRegistryProcessors, beanFactory);
        // 加入普通的容器工厂后置处理器集合
        registryProcessors.addAll(currentRegistryProcessors);
        // 调用 BeanDefinition 后置处理器
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
        // 清空当前,支持 BeanDefinition 定义的后置处理器集合
        currentRegistryProcessors.clear();

        // 然后, 从容器中获得BeanDefinition后置处理器(BeanDefinitionRegistryPostProcessors)接口的实现类
        postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            // 支持 Ordered 排序接口
            if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
                // 加入当前, 支持 BeanDefinition 定义的后置处理器集合
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                // 加入 需要调用的后置处理器集合
                processedBeans.add(ppName);
            }
        }
        // 排序
        sortPostProcessors(currentRegistryProcessors, beanFactory);
        // 加入普通的容器工厂后置处理器集合
        registryProcessors.addAll(currentRegistryProcessors);
        // 调用 BeanDefinition 后置处理器
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
        // 清空当前,支持 BeanDefinition 定义的后置处理器集合
        currentRegistryProcessors.clear();
        
        // 最后, 循环获取实现 BeanDefinitionRegistryPostProcessor 接口实现
        boolean reiterate = true;
        while (reiterate) {
            reiterate = false;
            // 从容器中获得BeanDefinition后置处理器(BeanDefinitionRegistryPostProcessors)接口的实现类
            postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            for (String ppName : postProcessorNames) {
                if (!processedBeans.contains(ppName)) {
                    // 加入当前, 支持 BeanDefinition 定义的后置处理器集合
                    currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                    // 加入 需要调用的后置处理器集合
                    processedBeans.add(ppName);
                    // 每次有新的加入, 则再次进行循环
                    reiterate = true;
                }
            }
            // 排序
            sortPostProcessors(currentRegistryProcessors, beanFactory);
            // 加入普通的容器工厂后置处理器集合
            registryProcessors.addAll(currentRegistryProcessors);
            // 调用 BeanDefinition 后置处理器
            invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
            // 清空当前,支持 BeanDefinition 定义的后置处理器集合
            currentRegistryProcessors.clear();
        }

        // 现在,调用 BeanFactoryPostProcessor 接口. 
        // 支持 BeanDefinition 定义的后置处理器集合(经过多次扩充)
        invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
        // 普通的容器工厂后置处理器集合(仅包含容器上下文启动阶段主动设置的后置处理)
        invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
    }
	
    // 当前BeanFactory仅为一般的后置处理器.
    else {
        // 调用 BeanFactoryPostProcessor 接口
        invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
    }

    // 不要在这里初始化FactoryBeans：我们需要保留所有未初始化的常规 Bean, 以使 Bean工厂后处理器对其应用!
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

    // 实现 PriorityOrdered 接口的 Bean工厂后置处理器集合
    List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    // 实现 Ordered 接口的 Bean工厂后置处理器类名集合
    List<String> orderedPostProcessorNames = new ArrayList<>();
    // 普通的Bean工厂后置处理器集合类名集合
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
        if (processedBeans.contains(ppName)) {
            // 如果已经处理过, 则跳过
        }
        else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            // 添加到实现 PriorityOrdered 接口的 Bean工厂后置处理器集合
            priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
        }
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            // 添加到实现 Ordered 接口的 Bean工厂后置处理器类名集合
            orderedPostProcessorNames.add(ppName);
        }
        else {
            // 添加到普通的Bean工厂后置处理器集合类名集合
            nonOrderedPostProcessorNames.add(ppName);
        }
    }

    // 首先, 排序并调用实现 PriorityOrdered 接口的 Bean工厂后置处理器集合
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

    // 然后, 排序并调用实现 PriorityOrdered 接口的 Bean工厂后置处理器类名的实现类
    List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String postProcessorName : orderedPostProcessorNames) {
        // 通过名称获得类.
        orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }
    sortPostProcessors(orderedPostProcessors, beanFactory);
    invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

    // 最后, 调用普通的Bean工厂后置处理器集合类名集合的实现类
    List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String postProcessorName : nonOrderedPostProcessorNames) {
        nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }
    invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

    // 清空,并合并元信息.(如经过SpEL表达式解析的占位符等元信息的合并)
    beanFactory.clearMetadataCache();
}
```



#### 2-11-6-X 注册 `Bean` 的后置处理器

##### `registerBeanPostProcessors`

1. 找到 `BeanPostProcessor` 的实现, 其定义 `Bean` 在初始化前后的动作
2. 根据是否实现 `PriorityOrdered` ; `Order` 对其实现进行排序



#### 2-11-6-0 注册 `Bean` 的后置处理器

入口方法 `org.springframework.context.support.AbstractApplicationContext#registerBeanPostProcessors`

```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) 
    // 调用 静态方法, 注册
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```

#### 2-11-6-1 注册 `Bean` 的后置处理器

调用 `org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors` 具体实现.
获得 `BeanPostProcessor` 接口实现, 并注册到容器中.
其中 `MergedBeanDefinitionPostProcessor` 接口为 `Bean` 定义在属性合并时调用

```java
/**
 * @param beanFactory 容器工厂,包含了所有Bean信息
 * @param applicationContext 上下文容器
 */
public static void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
	// 从容器工厂中获得 BeanPostProcessor 的实现类
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
    
    // 注册BeanPostProcessorChecker，当在BeanPostProcessor实例化期间创建Bean时，即当某个Bean不适合所有BeanPostProcessor处理时，记录一条信息消息。
    int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
    beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

    // 根据其是否实现 PriorityOrdered 接口, Ordered 接口, 进行区分
    
    // 实现 PriorityOrdered 接口的 BeanPostProcessor 的集合
    List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    // 实现 PriorityOrdered 接口和 MergedBeanDefinitionPostProcessor 接口的 BeanPostProcessor 的集合
    List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
    // 实现 Ordered 接口的 BeanPostProcessor 的类的名称的集合
    List<String> orderedPostProcessorNames = new ArrayList<>();
    // 未实现排序接口的 BeanPostProcessor 的类的名称的集合
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    // 循环添加
    for (String ppName : postProcessorNames) {
        // 实现 PriorityOrdered 接口
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            priorityOrderedPostProcessors.add(pp);
            // 实现 MergedBeanDefinitionPostProcessor 接口
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        // 实现 Ordered 接口
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            orderedPostProcessorNames.add(ppName);
        }
        // 未实现排序接口
        else {
            nonOrderedPostProcessorNames.add(ppName);
        }
    }

    // 首先, 排序并注册实现了 PriorityOrdered 接口的 Bean后置处理器类
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

    // 然后, 获取并注册实现了 Ordered 接口的Bean后置处理器类
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

    // 最后, 获取并注册未实现排序接口的Bean后置处理器
    List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String ppName : nonOrderedPostProcessorNames) {
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
        nonOrderedPostProcessors.add(pp);
        if (pp instanceof MergedBeanDefinitionPostProcessor) {
            internalPostProcessors.add(pp);
        }
    }
    registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

    // 对于 MergedBeanDefinitionPostProcessor 接口的实现, 最后注册.
    sortPostProcessors(internalPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, internalPostProcessors);

    // 重新注册 ApplicationListenerDetector 后置处理器
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
    
    
    // 批量注册给定的Bean后置处理器
	private static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanPostProcessor> postProcessors) {

		if (beanFactory instanceof AbstractBeanFactory) {
            // 批量注册
			((AbstractBeanFactory) beanFactory).addBeanPostProcessors(postProcessors);
		}
		else {            
			for (BeanPostProcessor postProcessor : postProcessors) {
                // 单条注册
				beanFactory.addBeanPostProcessor(postProcessor);
			}
		}
	}    
}
```

#### 2-11-6-2 注册

在 `org.springframework.beans.factory.support.AbstractBeanFactory#addBeanPostProcessor` 中定义了向 `BeanFactory` 注册 `Bean` 后置处理器的方法, 为避免重复注册, 采用先删除,后重新注入的方式

```java
public void addBeanPostProcessors(Collection<? extends BeanPostProcessor> beanPostProcessors) {
    this.beanPostProcessors.removeAll(beanPostProcessors);
    this.beanPostProcessors.addAll(beanPostProcessors);
}
```



#### 2-11-7-0 初始化国际化资源

##### `initMessageSource`

初始化国际化资源并注入到容器中

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

##### `initApplicationEventMulticaster` 

初始化时间广播器并注入到容器中

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

##### `onRefresh` 

根据不同的环境, 交由具体的子类实现

`Web` 环境下,交由 `org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#onRefresh` 去实现, 具体为创建一个 `web` 容器. 如 `tomcat`

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

##### `registerListeners`

向广播器添加监听器的具体实现并进行广播

1. 注册从 `META-INF/spring.factories` 读取的系统监听器; 
2. 注册从 `BeanFactory` 中读取的系统监听器; 
3. 广播早期事`earlyApplicationEvents`  (当容器尚未创建完成但却已触发事件,则会保存其中)

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

##### `finishBeanFactoryInitialization`

1. 初始化剩下的单实例 `Bean`



**主要为自定义的业务逻辑 `Bean`**

步骤为:

1. `getBean` 
2.  `doGetBean` 
3.  `getSingleton` 
4.  `CreateBean` 
5.  `resolveBeforeInstantiation` 
6.  `doGreateBean` 
7.  `createBeanInstance` 
8.  `instantiateBean` 
9.  `instantiate` 
10.  `populateBean` 
11.  `initializedBean` 

具体代码如下

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 判断是否含有 conversionService 的实现,如果有则存入.. 一般没有
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
        beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // 是否加载 内置值解析器, 没有则注入, 一般有
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // 加载 LoadTimeWeaverAware 进行 AOP 织入操作, 一般没有
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // 停止使用临时的 ClassLoader
    beanFactory.setTempClassLoader(null);

    // 1. 冻结 BeanFactory , 实例化期间不希望有 Bean 定义注册
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



#### 2-11-11-2  `Bean` 的实例化

子类 `org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons` 方法调用, 进行实例化单例 `Bean`

```java
public void preInstantiateSingletons() throws BeansException {
    // 日志记录
    if (logger.isTraceEnabled()) {
        logger.trace("Pre-instantiating singletons in " + this);
    }

    // 获得所有的 BeanDefinition 的名字
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    // 循环, 构建单实例 Bean
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

通过调用 `doGetBean` 获得实例

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
        // 再次检查, 能否通过缓存获取, 避免循环依赖
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }

        // 获取 父类 BeanFactory
        BeanFactory parentBeanFactory = getParentBeanFactory();
        // 如果 父类已经加载过,则返回; 否则子类BeanFactory继续加载
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

        // 标记当前Bean正在被创建, 加入到 alreadyCreated 集合中
        if (!typeCheckOnly) {
            markBeanAsCreated(beanName);
        }

        // 开始创建 Bean
        try {
            // 合并获得 BeanDefinition 的定义信息
            final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
            // 检查 BeanDefinition , 抽象类不能实例化
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
                    // 执行 getBean ,实例化依赖对象
                    try {
                        getBean(dep);
                    }
                    catch (NoSuchBeanDefinitionException ex) {
                        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                        "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
                    }
                }
            }

            // 如果是单例, 创建 Bean
            if (mbd.isSingleton()) {
                // 获得 Bean 工厂, 调用父类的方法
                sharedInstance = getSingleton(beanName, () -> {
                    // 匿名内部类, 创建Bean实例
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



#### 2-11-11-4 获得单实例 `Bean`

`org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton`  调用父类的获得单实例方法, 传入一个 `ObjectFactory` 接口的匿名函数类

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    // BeanName 不能为空
    Assert.notNull(beanName, "Bean name must not be null");
    // 线程锁
    synchronized (this.singletonObjects) {
        // 尝试从已实例化缓存中读取
        Object singletonObject = this.singletonObjects.get(beanName);
        // 如果仍未实例化, 则调用
        if (singletonObject == null) {
            // 判断, 当前Bean 是否处在销毁阶段
            if (this.singletonsCurrentlyInDestruction) {
                throw new BeanCreationNotAllowedException(beanName,
                                                          "Singleton bean creation not allowed while singletons of this factory are in destruction " +
                                                          "(Do not request a bean from a BeanFactory in a destroy method implementation!)");
            }
            // 日志..
            if (logger.isDebugEnabled()) {
                logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
            }
            // 检查, 如果出现错误则抛出
            beforeSingletonCreation(beanName);
            boolean newSingleton = false;
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
            if (recordSuppressedExceptions) {
                this.suppressedExceptions = new LinkedHashSet<>();
            }
            
            // 调用传入的 ObjectFactory 接口的匿名实现, 获得Bean实例
            try {
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            catch (IllegalStateException ex) {
                // Has the singleton object implicitly appeared in the meantime ->
                // if yes, proceed with it since the exception indicates that state.
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    throw ex;
                }
            }
            catch (BeanCreationException ex) {
                if (recordSuppressedExceptions) {
                    for (Exception suppressedException : this.suppressedExceptions) {
                        ex.addRelatedCause(suppressedException);
                    }
                }
                throw ex;
            }
            finally {
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = null;
                }
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}
```

#### 2-11-11-5 检查 `Bean` 状态

子类 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean` 执行具体的创建 Bean

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
	// 日志
    if (logger.isTraceEnabled()) {
        logger.trace("Creating instance of bean '" + beanName + "'");
    }
    // BeanDefinition 描述信息
    RootBeanDefinition mbdToUse = mbd;

    // 获得Bean的原生类
    Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }

    // 准备方法替换, 替换 lookup methods ... 不常用
    try {
        mbdToUse.prepareMethodOverrides();
    }
    catch (BeanDefinitionValidationException ex) {
        throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(), beanName, "Validation of method overrides failed", ex);
    }

    try {
        // 为 BeanPostProcessor接口, Bean的后置处理器提供一个返回代理类的机会, 如果返回代理类,则该代理类作为实例化结果
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
    catch (Throwable ex) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
                                        "BeanPostProcessor before instantiation of bean failed", ex);
    }

    try {
        // 未有代理类生成, 则进行正常的实例化过程.
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        if (logger.isTraceEnabled()) {
            logger.trace("Finished creating instance of bean '" + beanName + "'");
        }
        return beanInstance;
    }
    catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
        // A previously detected exception with proper bean creation context already,
        // or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
        throw ex;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
            mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
    }
}
```



#### 2-11-11-6  根据 `BeanDefinition`  创建代理类

`org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#resolveBeforeInstantiation` 方法中, 描述了对代理类的创建过程

```java
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        // 确保当前类已经被正确解析
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            // 要创建的目标类的真正class类型
            Class<?> targetType = determineTargetType(beanName, mbd);
            if (targetType != null) {
                // 执行 InstantiationAwareBeanPostProcessor 接口中的 postProcessBeforeInstantiation 方法, 将其结果作为代理对象
                bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                // 减少循环判断次数
                if (bean != null) {
                    // 代理对象不为空, 执行 BeanPostProcessor 接口中的 postProcessAfterInitialization 方法, 动态修改代理对象. 
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }
        mbd.beforeInstantiationResolved = (bean != null);
    }
    return bean;
}

//  执行 InstantiationAwareBeanPostProcessor 接口中的 postProcessBeforeInstantiation 方法, 将其结果作为代理对象
@Override
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
        Object result = bp.postProcessBeforeInstantiation(beanClass, beanName);
        if (result != null) {
            return result;
        }
    }
    return null;
}

// 代理对象不为空, 执行 BeanPostProcessor 接口中的 postProcessAfterInitialization 方法, 动态修改代理对象. 
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```

#### 2-11-11-7 根据 `BeanDefinition` 创建非代理类

`createBeanInstance` 创建实例并返回包装结果

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {

    BeanWrapper instanceWrapper = null;
    // 如果是单例Bean, 从单例缓存中移除
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    // 1. 创建实例的包装信息
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    // 包装类中的修饰的实例信息,真正的Bean对象.
    Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();
    if (beanType != NullBean.class) {
        // 修改其解析的代理类对应的类类型
        mbd.resolvedTargetType = beanType;
    }

    // 2. 执行 MergedBeanDefinitionPostProcessor 接口中定义的 postProcessMergedBeanDefinition 进行属性合并
    synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            try {
                applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            }
            catch (Throwable ex) {
                throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                                "Post-processing of merged bean definition failed", ex);
            }
            mbd.postProcessed = true;
        }
    }

    // Eagerly cache singletons to be able to resolve circular references
    // even when triggered by lifecycle interfaces like BeanFactoryAware.
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        if (logger.isTraceEnabled()) {
            logger.trace("Eagerly caching bean '" + beanName +
                         "' to allow for resolving potential circular references");
        }
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }

    // 3. 为 Bean 填充属性, 相关依赖
    Object exposedObject = bean;
    try {
        populateBean(beanName, mbd, instanceWrapper);
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    catch (Throwable ex) {
        if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
            throw (BeanCreationException) ex;
        }
        else {
            throw new BeanCreationException(
                mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
        }
    }

    if (earlySingletonExposure) {
        Object earlySingletonReference = getSingleton(beanName, false);
        if (earlySingletonReference != null) {
            if (exposedObject == bean) {
                exposedObject = earlySingletonReference;
            }
            else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
                String[] dependentBeans = getDependentBeans(beanName);
                Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
                for (String dependentBean : dependentBeans) {
                    if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                        actualDependentBeans.add(dependentBean);
                    }
                }
                if (!actualDependentBeans.isEmpty()) {
                    throw new BeanCurrentlyInCreationException(beanName,
                                                               "Bean with name '" + beanName + "' has been injected into other beans [" +
                                                               StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
                                                               "] in its raw version as part of a circular reference, but has eventually been " +
                                                               "wrapped. This means that said other beans do not use the final version of the " +
                                                               "bean. This is often the result of over-eager type matching - consider using " +
                                                               "'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");
                }
            }
        }
    }

    // Register bean as disposable.
    try {
        registerDisposableBeanIfNecessary(beanName, bean, mbd);
    }
    catch (BeanDefinitionValidationException ex) {
        throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
    }

    return exposedObject;
}

// 1. 获取 Bean 实例的包装信息
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
	// 获得 Bean 的 class 信息
    Class<?> beanClass = resolveBeanClass(mbd, beanName);

    if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
    }

    // 是否为从其他配置类中加载而来
    Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
    if (instanceSupplier != null) {
        // 调用其配置文件中的配置信息进行实例化
        return obtainFromSupplier(instanceSupplier, beanName);
    }

    // 是否为工厂Bean, 调用其对应存在的工厂方法返回实例化结果
    if (mbd.getFactoryMethodName() != null) {
        return instantiateUsingFactoryMethod(beanName, mbd, args);
    }

    // 
    boolean resolved = false;
    boolean autowireNecessary = false;
    if (args == null) {
        synchronized (mbd.constructorArgumentLock) {
            if (mbd.resolvedConstructorOrFactoryMethod != null) {
                resolved = true;
                autowireNecessary = mbd.constructorArgumentsResolved;
            }
        }
    }
    if (resolved) {
        if (autowireNecessary) {
            return autowireConstructor(beanName, mbd, null, null);
        }
        else {
            return instantiateBean(beanName, mbd);
        }
    }

    // 1-1. 通过掉用 SmartInstantiationAwareBeanPostProcessor 接口来选择构造器生成对应的Bean实例.
    Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
        mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
        return autowireConstructor(beanName, mbd, ctors, args);
    }

    // 使用首选的构造器生成对应实例.. 如声明的有参构造器
    ctors = mbd.getPreferredConstructors();
    if (ctors != null) {
        return autowireConstructor(beanName, mbd, ctors, null);
    }

    // 1-2. 什么没有, 使用默认的无参构造器创建实例
    return instantiateBean(beanName, mbd);
}

// 1-1. 通过掉用 SmartInstantiationAwareBeanPostProcessor 接口来选择构造器生成对应的Bean实例.
protected Constructor<?>[] determineConstructorsFromBeanPostProcessors(@Nullable Class<?> beanClass, String beanName)
    throws BeansException {

    if (beanClass != null && hasInstantiationAwareBeanPostProcessors()) {
        for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {
            Constructor<?>[] ctors = bp.determineCandidateConstructors(beanClass, beanName);
            if (ctors != null) {
                return ctors;
            }
        }
    }
    return null;
}

// 1-2. 使用默认的无参构造器创建实例
protected BeanWrapper instantiateBean(String beanName, RootBeanDefinition mbd) {
    try {
        Object beanInstance;
        if (System.getSecurityManager() != null) {
            beanInstance = AccessController.doPrivileged(
                (PrivilegedAction<Object>) () -> getInstantiationStrategy().instantiate(mbd, beanName, this),
                getAccessControlContext());
        }
        else {
            // 获取实例化策略并进行实例化, 默认 cglib 来进行实例化
            beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, this);
        }
        // 将实例化后的返回进行进行包装
        BeanWrapper bw = new BeanWrapperImpl(beanInstance);
        initBeanWrapper(bw);
        return bw;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
    }
}

// 2. 执行 MergedBeanDefinitionPostProcessor 接口中定义的 postProcessMergedBeanDefinition 进行属性合并
protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
    for (MergedBeanDefinitionPostProcessor processor : getBeanPostProcessorCache().mergedDefinition) {
        processor.postProcessMergedBeanDefinition(mbd, beanType, beanName);
    }
}

// 3. 为 Bean 填充属性, 相关依赖
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    if (bw == null) {
        if (mbd.hasPropertyValues()) {
            throw new BeanCreationException(
                mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
        }
        else {
            // Skip property population phase for null instance.
            return;
        }
    }

    // 调用 InstantiationAwareBeanPostProcessor 子接口的 postProcessAfterInstantiation 去修改Bean的属性
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                return;
            }
        }
    }

    // 属性列表
    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

    // 依赖注入方式
    int resolvedAutowireMode = mbd.getResolvedAutowireMode();
    if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
        // 根据名称自动注入
        if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }
        // 根据类型自动注入
        if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }
        pvs = newPvs;
    }

    boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
    boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);
	
    // 调用 InstantiationAwareBeanPostProcessor 子接口的 
    PropertyDescriptor[] filteredPds = null;
    if (hasInstAwareBpps) {
        if (pvs == null) {
            pvs = mbd.getPropertyValues();
        }
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                if (filteredPds == null) {
                    filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                }
                pvsToUse = bp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                if (pvsToUse == null) {
                    return;
                }
            }
            pvs = pvsToUse;
        }
    }
    if (needsDepCheck) {
        if (filteredPds == null) {
            filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
        }
        checkDependencies(beanName, mbd, filteredPds, pvs);
    }

    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}
```

#### 2-11-11-8  不同实例化策略实例化 `Bean`

`org.springframework.beans.factory.support.InstantiationStrategy` 接口定义了实例化 `Bean` 的策略. 

1. `org.springframework.beans.factory.support.SimpleInstantiationStrategy` 简单地通过反射实例化
2. `org.springframework.beans.factory.support.CglibSubclassingInstantiationStrategy` 通过 `cglib` 实例化

```java
// 实例化 Bean
@Override
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
    // 没有方法重写, 使用简单的反射创建
    if (!bd.hasMethodOverrides()) {
        Constructor<?> constructorToUse;
        synchronized (bd.constructorArgumentLock) {
            constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
            if (constructorToUse == null) {
                final Class<?> clazz = bd.getBeanClass();
                if (clazz.isInterface()) {
                    throw new BeanInstantiationException(clazz, "Specified class is an interface");
                }
                try {
                    if (System.getSecurityManager() != null) {
                        constructorToUse = AccessController.doPrivileged(
                            (PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
                    }
                    else {
                        constructorToUse = clazz.getDeclaredConstructor();
                    }
                    bd.resolvedConstructorOrFactoryMethod = constructorToUse;
                }
                catch (Throwable ex) {
                    throw new BeanInstantiationException(clazz, "No default constructor found", ex);
                }
            }
        }
        // 通过反射构造实例化对象
        return BeanUtils.instantiateClass(constructorToUse);
    }    
    // 否则使用 cglib 实例化
    else {
        return instantiateWithMethodInjection(bd, beanName, owner);
    }
}
```







#### 2-11-12-0 结束刷新

##### `finishRefresh`

1. 初始化生命周期处理器 `LifecycleProcessor` 实现类,并调用 `onRefresh` 方法
2. 发布 `ContextRefreshedEvent` 事件
3. `JMX` 相关处理

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



#### 2-11-13-0 清理相关缓存

##### `resetCommonCaches`

清理相关 `Bean` 的元数据信息

```java
protected void resetCommonCaches() {
    ReflectionUtils.clearCache();
    AnnotationUtils.clearCache();
    ResolvableType.clearCache();
    CachedIntrospectionResults.clearClassLoader(getClassLoader());
}
```



## `Banner` 打印

#### 实现方式

1. 通过设置 `spring.banner.location` 设置文字简笔画图片
2. 通过设置 `spring.banner.image.location` 设置图片 (`png` , `jpg` , `gif` ),系统会转为对应的字符简笔画
3. 在 `SpringApplication` 中的 `setBanner()` 方法配置信息
4. 系统默认位置在 `resources/banner.txt`  目录下
5. 通过 `spring.main.banner-mode=off` 关闭输出



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
    // 传入类加载器和兜底的Banner 
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
    // 以上均未匹配, 返回默认的Banner配置
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

在框架启动后, 执行某些操作

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

实现 `org.springframework.boot.ApplicationRunner` 接口中的方法, 重写 `run` 方法, 在Spring框架完成后进行操作.  `context` 为当前容器, `args` 为框架封装的启动参数的包装类(将参数转为 `key-value` 形式)

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


## `Aware` 接口

 在需要使用 `Spring` 容器中的相关功能时, 调用 `org.springframework.beans.factory.Aware` 接口及其相关实现
因为 `Spring` 的相关功能多为工厂类或者各种容器,因此不能通过容器获得相应的对象引用, 因此需要 `Aware` 接口实现

### 常用接口

| 接口                             | 作用                     | 赋值的调用位置                                               |
| -------------------------------- | ------------------------ | ------------------------------------------------------------ |
| `BeanNameAware`                  | 获取容器中 `bean` 的名称 | `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods` 初始化处理 |
| `BeanClassLoaderAware`           | 获得类加载器             | `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods` |
| `BeanFactoryAware`               | 获得 `bean` 创建工厂     | `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods` |
| `EnvironmentAware`               | 获得环境变量             | `org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces` 后置处理器处理 |
| `EmbeddedValueResolverAware`     | 获得容器加载属性文件的值 | `org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces` |
| `ResourceLoaderAware`            | 获得资源加载器           | `org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces` |
| `ApplicationEventPublisherAware` | 获得应用事件发布器       | `org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces` |
| `MessageSouceAware`              | 获得文本信息, 用以国际化 | `org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces` |
| `ApplicationContextAware`        | 获得当前应用上下文       | `org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces` |
| `ApplicationStartupAware`        | 获得容器运行状态标签     | `org.springframework.context.support.ApplicationContextAwareProcessor#invokeAwareInterfaces` |

### 调用位置

1. `doCreateBean` -> `initializeBean` -> `invokeAwareMethods` -> `applyBeanPostProcessorsBeforeInitialization` -> `ApplicationContextAwareProcessor`
2. 初始化 `Bean` 在方法 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean` 中

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
// 具体的赋值调用
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

1. 自定义框架类, 模拟框架组件
2. 自定义子接口, 继承自 `aware` 接口, 提供 `setter` 框架组件的方法
3. 自定义后置处理器 `BeanPostProcessor` 实现并注入到容器中, 以便对每一个自定义`aware` 接口实现类, 进行后置处理 `postProcessBeforeInitialization`, 调用 `setter` 方法传入框架组件对象
4. 目标类, 实现自定义的 `aware` 接口, 实现 `setter` 获得框架组件.

#### 自定义框架类

自定义框架类, 用 `@Component` 注入容器中, 以便可以直接通过容器获得. 
模拟那些不能够直接获得的框架组件对象

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



## 属性配置

### 先后顺序

| 顺序 | 方式                                              | 示例                                                         |
| :--: | ------------------------------------------------- | :----------------------------------------------------------- |
|  1   | `Devtools` 全局属性                               | 热部署时指定                                                 |
|  2   | 测试环境 `@TestPropertySource` 注解               | `@TestPropertySource("classpath:属性文件.properties")` ; 在测试类上添加 |
|  3   | 测试环境 `properties` 属性                        | 通过在 `@SpringBootTest` 注解中, 为属性 `properties` 添加 `json` 格式的参数对象,完成配置 |
|  4   | 命令行参数                                        | `--key=value`                                                |
|  5   | `SPRING_APPLICATION_JSON` 属性                    | 通过将 `json`  格式的参数以 `SPRING_APPLICATION_JSON` 为参数名传入, 完成配置 |
|  6   | `ServletConfig` 初始化参数                        |                                                              |
|  7   | `ServletContext` 初始化参数                       |                                                              |
|  8   | `JNDI` 属性                                       |                                                              |
|  9   | `JAVA` 系统虚拟机属性                             | 通过设置 `JVM` 虚拟机参数在程序启动时赋值                    |
|  10  | 操作系统环境变量                                  | `Linux` 或者 `Windows` 中的环境变量                          |
|  11  | `RandomValuePropertySource` 随机属性              | `${random.int[20,30]}` 属性配置随机`20-30`之内的整数         |
|  12  | jar包外的 `application-{profile}.properties` 文件 | 默认在 `classpath` 路径下和 `classpath:/config` 路径下       |
|  13  | jar包内的 `application-{profile}.properties` 文件 | 默认使用 `default` 配置文件, 即 `application-default-properties` |
|  14  | jar包外的 `application.properties` 文件           | 默认使用当前jar包外同级目录下, 同级目录下的 `config` 目录下  |
|  15  | jar包内的 `application.properties` 文件           |                                                              |
|  16  | `@PropertySource` 绑定配置                        | `@PropertySource("classpath:属性文件.properties")` ; 在启动类上添加 |
|  17  | 默认属性                                          | `springApplication.setDefaultProperties(new Properties())` ; 在创建Spring容器时添加 |



## `Environment` 环境

在 `Environment` 对象中,有多个 `propertySources` , 每个 `propertySources` 封装了来自框架内外的参数信息.从中获得相关的属性.

![Environment类图](./images/2021-02/Environment类图-1.png)

### 源码步骤

#### 2-6-0-0 配置环境信息

在 `run` 方法中 , 通过 `ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments)` 方法构建上下环境信息.

``` java
private ConfigurableEnvironment prepare Environment(SpringApplicationRunListeners listeners,
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
        environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment, deduceEnvironmentClass());
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
    // 2. 配置激活属性
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

#### 2-6-4-1 监听事件

其中的广播, `org.springframework.boot.context.config.ConfigFileApplicationListener#onApplicationEvent` 方法在收到该事件后, 对环境配置进行修改.
具体通过`SpringFactoriesLoader` 加载 `META-INF/spring.factories`  中接口 `org.springframework.boot.env.EnvironmentPostProcessor` 的实现类(实现 `Order` 接口, 定义具体属性重载排序), 并调用从而实现对配置信息的修改
常用的有 `SpringApplicationJsonEnvironmentPostProcessor`  (添加 `SPRING_APPLICATION_JSON` 属性集),  `CloudFoundryVcapEnvironmentPostProcessor` (添加 `vcap` 属性集), `

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
        // 修改环境变量, 解析 SPRING_APPLICATION_JSON属性为属性集
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



## `profile` 分散配置

默认激活的配置文件 `application-default.properties`, 如果存在其他配置文件,则失效

通过使用 `spring.profiles.active` 指定具体的配置文件环境前缀, 对应的 `spring.profiles.default` 指定默认的配置文件.两者互斥

通过使用 `spring.profiles.include` 指定包含多个配置文件

通过使用 `spring.profiles.name` 在命令行指定具体的配置文件



### 源码步骤

#### 2-6-4-1 监听事件

在框架环境准备完成后, 会发送 `ApplicationEnvironmentPreparedEvent` 事件,  `org.springframework.boot.context.config.ConfigFileApplicationListener#onApplicationEnvironmentPreparedEvent` 方法在监听到该事件后, 会调用接口`org.springframework.boot.env.EnvironmentPostProcessor` 的实现类中的`postProcessEnvironment` 方法, 其中包括 `ConfigFileApplicationListener` 类本身实现的方法.

```java
// 本身实现方法
@Override
public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
    addPropertySources(environment, application.getResourceLoader());
}

// 添加指定环境配置源
protected void addPropertySources(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
    // 添加 random 和 systemEnvironment 配置源
    RandomValuePropertySource.addToEnvironment(environment);
    // 加载配置
    new Loader(environment, resourceLoader).load();
}
```

其中的 `load()` 方法, 将系统具体指定的配置前缀进行添加

```java
void load() {
    // defaultProperties
    FilteredPropertySource.apply(this.environment, DEFAULT_PROPERTIES, LOAD_FILTERED_PROPERTY,
    (defaultProperties) -> {
        // 初始化集合
        this.profiles = new LinkedList<>();
        this.processedProfiles = new LinkedList<>();
        this.activatedProfiles = false;
        this.loaded = new LinkedHashMap<>();
        // 初始化默认的 default 集合
        initializeProfiles();
        // 遍历配置集合. 如果不为空, 表示有属性集合, 加载读取其中的配置文件
        while (!this.profiles.isEmpty()) {
            Profile profile = this.profiles.poll();
            if (isDefaultProfile(profile)) {
                // 获得 spring.profiles.active 中指定的前缀, 默认为 default
                addProfileToEnvironment(profile.getName());
            }
            // 2. 加载读取配置文件
            load(profile, this::getPositiveProfileFilter,
                    addToLoaded(MutablePropertySources::addLast, false));
            // 已读取集合 
            this.processedProfiles.add(profile);
        }
        // 如果为空, 调用读取
        load(null, this::getNegativeProfileFilter, addToLoaded(MutablePropertySources::addFirst, true));
        // 将读取到的配置文件, 写入到环境中
        addLoadedPropertySources();
        // 为环境设置激活的配置信息
        applyActiveProfiles(defaultProperties);
    });
}
```

初始化默认集合

```java
private void initializeProfiles() {
    // 添加空前缀. 如 applocation.  配置文件, 则没有环境前缀
    this.profiles.add(null);
    Binder binder = Binder.get(this.environment);
    // 获得激活的配置文件
    Set<Profile> activatedViaProperty = getProfiles(binder, ACTIVE_PROFILES_PROPERTY);
    // 获得扩展的配置文件
    Set<Profile> includedViaProperty = getProfiles(binder, INCLUDE_PROFILES_PROPERTY);
    List<Profile> otherActiveProfiles = getOtherActiveProfiles(activatedViaProperty, includedViaProperty);
    this.profiles.addAll(otherActiveProfiles);
    // 添加属性集
    this.profiles.addAll(includedViaProperty);
    addActiveProfiles(activatedViaProperty);
    // 如果为1, 表示只有 application. 配置文件, 添加默认的配置
    if (this.profiles.size() == 1) {
        // spring.profile.default 定义的配置文件, 如果定义了分散文件属性,则无法加载. 可以在命令行中加入
        for (String defaultProfileName : this.environment.getDefaultProfiles()) {
            Profile defaultProfile = new Profile(defaultProfileName, true);
            this.profiles.add(defaultProfile);
        }
    }
}
```

#### 2-6-4-2 加载配置

首先遍历,尝试获取配置文件的位置

```java
private void load(Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
    // 获得配置文件的位置
    getSearchLocations().forEach((location) -> {
        // 判断是否为目录
        boolean isDirectory = location.endsWith("/");
        // 如果是目录,获得文件名. 属性 spring.config.name 指定文件名 默认 application  , 如果定义了分散文件属性,则无法加载. 可以在命令行中加入
        Set<String> names = isDirectory ? getSearchNames() : NO_SEARCH_NAMES;
        // 加载配置文件 指定文件路径 前缀 文件名
        names.forEach((name) -> load(location, name, profile, filterFactory, consumer));
    });
}
// 获得配置文件的位置
private Set<String> getSearchLocations() {
    // spring.config.additional-location 获取额外文件位置
    Set<String> locations = getSearchLocations(CONFIG_ADDITIONAL_LOCATION_PROPERTY);
    // spring.config.location 配置位置,如果指定,则加载
    if (this.environment.containsProperty(CONFIG_LOCATION_PROPERTY)) {
        locations.addAll(getSearchLocations(CONFIG_LOCATION_PROPERTY));
    }
    else {
        // 否则, 添加默认的配置文件路径
        locations.addAll(
            // 默认路径 classpath:/,classpath:/config/,file:./,file:./config/*/,file:./config/
            asResolvedSet(ConfigFileApplicationListener.this.searchLocations, DEFAULT_SEARCH_LOCATIONS));
    }
    return locations;
}
```

#### 2-6-4-2 读取文件

通过  `org.springframework.boot.context.config.ConfigFileApplicationListener.Loader#load`  方法,最终将配置文件中的属性读入

其中文件读取器有两个 `org.springframework.boot.env.PropertiesPropertySourceLoader` 读取 `.properties` 和 `.xml` 文件以及 `org.springframework.boot.env.YamlPropertySourceLoader` 读取 `.yml` 和 `.yaml`

```java
private void load(String location, String name, Profile profile, DocumentFilterFactory filterFactory,
                  DocumentConsumer consumer) {
    // 校验. 指定路径,文件不存再, 尝试递归调用
    if (!StringUtils.hasText(name)) {
        for (PropertySourceLoader loader : this.propertySourceLoaders) {
            if (canLoadFileExtension(loader, location)) {
                load(loader, location, profile, filterFactory.getDocumentFilter(profile), consumer);
                return;
            }
        }
        throw new IllegalStateException("File extension of config file location '" + location
                                        + "' is not known to any PropertySourceLoader. If the location is meant to reference "
                                        + "a directory, it must end in '/'");
    }
    // 读取文件
    Set<String> processed = new HashSet<>();
    for (PropertySourceLoader loader : this.propertySourceLoaders) {
        // 不同的加载器去加载. 在 /META-INF/spring.factories 中指定 PropertySourceLoader 实现类
        for (String fileExtension : loader.getFileExtensions()) {
            if (processed.add(fileExtension)) {
                // 读取文件方法.
                loadForFileExtension(loader, location + name, "." + fileExtension, profile, filterFactory,
                                     consumer);
            }
        }
    }
}
```

在读取文件方法中,具体如下

```java
// 读取文件方法.
private void loadForFileExtension(PropertySourceLoader loader, String prefix, String fileExtension,
                                  Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
    DocumentFilter defaultFilter = filterFactory.getDocumentFilter(null);
    DocumentFilter profileFilter = filterFactory.getDocumentFilter(profile);
    if (profile != null) {
        // Try profile-specific file & profile section in profile file (gh-340)
        String profileSpecificFile = prefix + "-" + profile + fileExtension;
        load(loader, profileSpecificFile, profile, defaultFilter, consumer);
        load(loader, profileSpecificFile, profile, profileFilter, consumer);
        // Try profile specific sections in files we've already processed
        for (Profile processedProfile : this.processedProfiles) {
            if (processedProfile != null) {
                String previouslyLoaded = prefix + "-" + processedProfile + fileExtension;
                load(loader, previouslyLoaded, profile, profileFilter, consumer);
            }
        }
    }
    // Also try the profile-specific section (if any) of the normal file
    load(loader, prefix + fileExtension, profile, profileFilter, consumer);
}

// 具体的读取方式
private void load(PropertySourceLoader loader, String location, Profile profile, DocumentFilter filter,
				DocumentConsumer consumer) {
    // 获得资源路径
    Resource[] resources = getResources(location);
    for (Resource resource : resources) {
        try {
            // 是否存在
            if (resource == null || !resource.exists()) {
                if (this.logger.isTraceEnabled()) {
                    StringBuilder description = getDescription("Skipped missing config ", location, resource,
                                                               profile);
                    this.logger.trace(description);
                }
                continue;
            }
            // 文件扩展名是否正确
            if (!StringUtils.hasText(StringUtils.getFilenameExtension(resource.getFilename()))) {
                if (this.logger.isTraceEnabled()) {
                    StringBuilder description = getDescription("Skipped empty config extension ", location,
                                                               resource, profile);
                    this.logger.trace(description);
                }
                continue;
            }
            // 文件扩展名
            String name = "applicationConfig: [" + getLocationName(location, resource) + "]";
            // 加载配置文件, 包装为 Document 对象(属性集, 配置前缀, 激活的配置前缀, 包括的其他扩展配置集合). 
            List<Document> documents = loadDocuments(loader, name, resource);
            // 加载失败.说明配置文件有问题,不符合Spring结构. 结束
            if (CollectionUtils.isEmpty(documents)) {
                if (this.logger.isTraceEnabled()) {
                    StringBuilder description = getDescription("Skipped unloaded config ", location, resource,
                                                               profile);
                    this.logger.trace(description);
                }
                continue;
            }
            // 加载成功,遍历
            List<Document> loaded = new ArrayList<>();
            for (Document document : documents) {
                // 避免重复加载
                if (filter.match(document)) {
                    // 添加 激活的属性
                    addActiveProfiles(document.getActiveProfiles());
                    // 添加 扩展属性
                    addIncludedProfiles(document.getIncludeProfiles());
                    loaded.add(document);
                }
            }
            Collections.reverse(loaded);
            // 处理
            if (!loaded.isEmpty()) {
                loaded.forEach((document) -> consumer.accept(profile, document));
                if (this.logger.isDebugEnabled()) {
                    StringBuilder description = getDescription("Loaded config file ", location, resource,
                                                               profile);
                    this.logger.debug(description);
                }
            }
        }
        catch (Exception ex) {
            StringBuilder description = getDescription("Failed to load property source from ", location,
                                                       resource, profile);
            throw new IllegalStateException(description.toString(), ex);
        }
    }
}
```

#### 2-6-4-3 属性赋值环境

在获得 `environment` 环境中的 `MutablePropertySources` 集合,  随后

```java
private void addLoadedPropertySources() {
    // 环境信息中的属性
    MutablePropertySources destination = this.environment.getPropertySources();
    // 遍历已经加载的属性
    List<MutablePropertySources> loaded = new ArrayList<>(this.loaded.values());
    Collections.reverse(loaded);
    String lastAdded = null;
    Set<String> added = new HashSet<>();
    // 将已经加载的配置信息拷贝到环境信息变量中
    for (MutablePropertySources sources : loaded) {
        for (PropertySource<?> source : sources) {
            if (added.add(source.getName())) {
                addLoadedPropertySource(destination, lastAdded, source);
                lastAdded = source.getName();
            }
        }
    }
}
```



## 异常报告

通过接口 `org.springframework.boot.SpringBootExceptionReporter` 中定义的类进行实现

```java
@FunctionalInterface
public interface SpringBootExceptionReporter {
	boolean reportException(Throwable failure);
}
```

### 自定义异常报告

通过实现接口,并注册

#### 实现类

通过实现 `org.springframework.boot.SpringBootExceptionReporter` 接口,  并声明一个有参的构造器,传入 `ConfigurableApplicationContext` 容器上下文对象.

```java
public class WebApplicationExceptionReporter implements SpringBootExceptionReporter {
    private ConfigurableApplicationContext context;
    // 有参构造器, 传入容器对象
    WebApplicationExceptionReporter(ConfigurableApplicationContext context) {
        this.context = context;
    }

    @Override
    public boolean reportException(Throwable failure) {
        if(failure instanceof RuntimeException){
            System.out.println("容器启动失败...");
            System.out.println(context.toString());
        }
        // 如果返回 true 则改异常不会再被后续异常报告处理
        return false;
    }
}
```

#### 注册异常报告

在 `spring.factories` 中注册, 目前 `SpringBoot` 仅有唯一实现类

```
org.springframework.boot.SpringBootExceptionReporter=top.jionjion.except.WebApplicationExceptionReporter
```



### 源码步骤

1. `run` 方法中调用, 创建填充 `Collection<SpringBootExceptionReporter>` 异常报告. 其泛型中接口的实现完成异常的捕捉处理.
  1. 调用 `analyze` 方法的具体实现, 找到对当前异常感兴趣的处理对象, 调用 `report` 方法.
  2. `report` 方法由 `FailureAnalysisReporter` 接口的实现类完成.

#### 2-9-0-0 异常报告器

在 `run` 方法中,  一共两次异常捕获, 一次是在容器创建, 另一次是在监听程序出现异常.

```java
	public ConfigurableApplicationContext run(String... args) {
		// ...
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		// ...
		try {
            // .. 
			// 1. 获得容器初始化的异常报告器, 通过 ClassLoader 从 spring.factories 文件中读取. 
            // 传入 参数及其类型ConfigurableApplicationContext.class. 调用构造器
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			
		}
        // 获得运行时异常信息
		catch (Throwable ex) {
            // 2. 并做处理, 交由不同的 监听器后续处理
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
            // 容器运行
			listeners.running(context);
		}
        // 出现异常
		catch (Throwable ex) {
            // 3. 并做处理
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

#### 2-9-1-0 初始化异常报告器

加载具体实现, `org.springframework.boot.diagnostics.FailureAnalyzers` 类实现了框架的异常报告捕获处理

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    // 类加载器
    ClassLoader classLoader = getClassLoader();
    // 通过类加载器, 加载 spring.factories 文件中的实现类的名字
    Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 创建实例,   传入 参数及其类型ConfigurableApplicationContext.class. 调用构造器
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
    // 排序, 通过 Order 接口指定
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
```

具体实现类为 `org.springframework.boot.diagnostics.FailureAnalyzers`
其构造方法如下, 

```java
// 初始化时,传入容器对象
FailureAnalyzers(ConfigurableApplicationContext context) {
    this(context, null);
}

// 传入容器对象, 及其类加载器
FailureAnalyzers(ConfigurableApplicationContext context, ClassLoader classLoader) {
    Assert.notNull(context, "Context must not be null");
    this.classLoader = (classLoader != null) ? classLoader : context.getClassLoader();
    // 1. 初始化 FailureAnalyzer 接口实例
    this.analyzers = loadFailureAnalyzers(this.classLoader);
    // 2. aware 接口调用
    prepareFailureAnalyzers(this.analyzers, context);
}
```

#### 2-9-1-1 初始化类 `FailureAnalyzer` 接口实例

```java
// 加载类
private List<FailureAnalyzer> loadFailureAnalyzers(ClassLoader classLoader) {
    // 类名
    List<String> analyzerNames = SpringFactoriesLoader.loadFactoryNames(FailureAnalyzer.class, classLoader);
    List<FailureAnalyzer> analyzers = new ArrayList<>();
    for (String analyzerName : analyzerNames) {
        try {
            // 获得类信息
            Constructor<?> constructor = ClassUtils.forName(analyzerName, classLoader).getDeclaredConstructor();
            // 类访问权限修改
            ReflectionUtils.makeAccessible(constructor);
            // 创建实例, 并添加
            analyzers.add((FailureAnalyzer) constructor.newInstance());
        }
        catch (Throwable ex) {
            logger.trace(LogMessage.format("Failed to load %s", analyzerName), ex);
        }
    }
    // 排序后返回
    AnnotationAwareOrderComparator.sort(analyzers);
    return analyzers;
}
```

#### 2-9-1-2 调用 `aware` 接口

调用 `BeanFactoryAware` 和 `EnvironmentAware` 中的设置方法. 设置属性

```java

private void prepareFailureAnalyzers(List<FailureAnalyzer> analyzers, ConfigurableApplicationContext context) {
    for (FailureAnalyzer analyzer : analyzers) {
        prepareAnalyzer(context, analyzer);
    }
}

private void prepareAnalyzer(ConfigurableApplicationContext context, FailureAnalyzer analyzer) {
    if (analyzer instanceof BeanFactoryAware) {
        ((BeanFactoryAware) analyzer).setBeanFactory(context.getBeanFactory());
    }
    if (analyzer instanceof EnvironmentAware) {
        ((EnvironmentAware) analyzer).setEnvironment(context.getEnvironment());
    }
}

```

#### 2-9-1-X `FailureAnalyzer`  接口

定义接口, 对抛出异常进行分析处理.
 在 `SpringBootExceptionReporter` 接口实现中调用, 完成异常分析, 并返回分析结果 `FailureAnalysis`

```java
package org.springframework.boot.diagnostics;

@FunctionalInterface
public interface FailureAnalyzer {
	FailureAnalysis analyze(Throwable failure);
}
```

比如 `org.springframework.boot.diagnostics.AbstractFailureAnalyzer` 抽象类.定义类大多数异常分析器的方法. 其中泛型 `<T extends Throwable>` 传入感兴趣异常的类型
将异常信息封装为 `FailureAnalysis` 类

```java
public abstract class AbstractFailureAnalyzer<T extends Throwable> implements FailureAnalyzer {

	@Override
	public FailureAnalysis analyze(Throwable failure) {
        // getCauseType() 获得泛型中的异常类
        // findCause() 如果当前抛出异常类属于泛型中该兴趣的异常,则调用分析
		T cause = findCause(failure, getCauseType());
		if (cause != null) {
			return analyze(failure, cause);
		}
		return null;
	}
    
    // 子类具体实现, 封装为 FailureAnalysis 对象
    protected abstract FailureAnalysis analyze(Throwable rootFailure, T cause);

    // 获得泛型中定义的感兴趣的异常类
    protected Class<? extends T> getCauseType() {
        return (Class<? extends T>) ResolvableType.forClass(AbstractFailureAnalyzer.class, getClass()).resolveGeneric();
    }
    
    // 如果当前抛出异常类属于泛型中该兴趣的异常,则调用分析
    protected final <E extends Throwable> E findCause(Throwable failure, Class<E> type) {
        // 循环读取异常堆栈,获得其中感兴趣的异常.
        while (failure != null) {
            if (type.isInstance(failure)) {
                return (E) failure;
            }
            // 获得抛出的异常中的下一层.
            failure = failure.getCause();
        }
        return null;
    }
}
```

常见的具体实现类有以下等, 常用 `异常名` + `Analyzer` 命令, 表示这是对该类进行异常报告

- `BeanCurrentlyInCreationFailureAnalyzer.class` 
- `DataSourceBeanCreationFailureAnalyzer.class`
- `NoSuchMethodFailureAnalyzer.class`
- `NoUniqueBeanDefinitionFailureAnalyzer.class`
- `PortInUseFailureAnalyzer.class`

其类图如下

![AbstractFailureAnalyzer类图](./images/2021-02/AbstractFailureAnalyzer类图-1.png)

#### 2-9-2-0 异常分析器

`org.springframework.boot.diagnostics.FailureAnalyzers`  实现 `org.springframework.boot.diagnostics.FailureAnalyzer` 接口中的 `analyze` 方法, 将当前异常信息封装为 `org.springframework.boot.diagnostics.FailureAnalysis` 对象, 并返回.
再由不同的异常报告类 `org.springframework.boot.diagnostics.FailureAnalysisReporter` 进行报告

```java
// 实现方法
@Override
public boolean reportException(Throwable failure) {
    // 获得封装的异常信息
    FailureAnalysis analysis = analyze(failure, this.analyzers);
    // 报告异常
    return report(analysis, this.classLoader);
}
```

#### 2-9-2-X 异常分析结果类 `FailureAnalysis`

```java
public class FailureAnalysis {
	// 描述
	private final String description;
	// 动作
	private final String action;
	// 异常
	private final Throwable cause;
}
```

#### 2-9-2-1 返回异常分析结果信息

```java
// 获得封装的异常信息
private FailureAnalysis analyze(Throwable failure, List<FailureAnalyzer> analyzers) {
    // 遍历,获得最先抛出的一个异常报告,并对其返回
    for (FailureAnalyzer analyzer : analyzers) {
        try {           
            FailureAnalysis analysis = analyzer.analyze(failure);
            if (analysis != null) {
                return analysis;
            }
        }
        catch (Throwable ex) {
            logger.debug(LogMessage.format("FailureAnalyzer %s failed", analyzer), ex);
        }
    }
    return null;
}
```

#### 2-9-2-2 报告异常

通过 `spring.factories` 中的 `org.springframework.boot.diagnostics.FailureAnalysisReporter` 接口实现类,去完成报告的创建

```java
private boolean report(FailureAnalysis analysis, ClassLoader classLoader) {
    List<FailureAnalysisReporter> reporters = SpringFactoriesLoader.loadFactories(FailureAnalysisReporter.class,
                                                                                  classLoader);
    if (analysis == null || reporters.isEmpty()) {
        return false;
    }
    for (FailureAnalysisReporter reporter : reporters) {
        reporter.report(analysis);
    }
    return true;
}
```

#### 2-9-3-0 `FailureAnalysisReporter`  异常分析报告接口
通过读取 `spring.factories` 中的 `org.springframework.boot.diagnostics.FailureAnalysisReporter` 接口实现类, 将当前封装的异常信息进行处理. 其具体实现为 `org.springframework.boot.diagnostics.LoggingFailureAnalysisReporter` 目前只有一个.

```java
package org.springframework.boot.diagnostics;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import org.springframework.util.StringUtils;

public final class LoggingFailureAnalysisReporter implements FailureAnalysisReporter {

	private static final Log logger = LogFactory.getLog(LoggingFailureAnalysisReporter.class);
	// 报告异常
	@Override
	public void report(FailureAnalysis failureAnalysis) {
        // 打印异常信息
		if (logger.isDebugEnabled()) {
			logger.debug("Application failed to start due to an exception", failureAnalysis.getCause());
		}
		if (logger.isErrorEnabled()) {
			logger.error(buildMessage(failureAnalysis));
		}
	}
	// 具体的异常信息
	private String buildMessage(FailureAnalysis failureAnalysis) {
		StringBuilder builder = new StringBuilder();
		builder.append(String.format("%n%n"));
		builder.append(String.format("***************************%n"));
		builder.append(String.format("APPLICATION FAILED TO START%n"));
		builder.append(String.format("***************************%n%n"));
		builder.append(String.format("Description:%n%n"));
		builder.append(String.format("%s%n", failureAnalysis.getDescription()));
		if (StringUtils.hasText(failureAnalysis.getAction())) {
			builder.append(String.format("%nAction:%n%n"));
			builder.append(String.format("%s%n", failureAnalysis.getAction()));
		}
		return builder.toString();
	}

}
```



#### 2-16-0-0 异常处理

1. 在 `try - catch` 中捕获到异常,调用 `handleRunFailure` 方法
2. 获得系统预设的退出状态码. 并将其进行广播
3. 调用 `SpringApplicationRunListener` 的 `failed` 方法, 广播 `ApplicationFailedEvent` 事件
4. 调用 `SpringBootExceptionReporter` 接口的 `reportException` 方法, 报告异常信息
5. 容器关闭
6. 重新抛出异常,告知外围系统

#### 2-16-0-1 执行流程
在初始化异常报告器后, 如果在后续过程中发生了异常,则调用 `handleRunFailure` 方法, 将处理具体异常

```java
private void handleRunFailure(ConfigurableApplicationContext context, Throwable exception,
                              Collection<SpringBootExceptionReporter> exceptionReporters, SpringApplicationRunListeners listeners) {
    try {
        try {
            // 1. 处理异常,获取退出代码
            handleExitCode(context, exception);
            if (listeners != null) {
                // 2. 发送启动失败事件 ApplicationFailedEvent 
                listeners.failed(context, exception);
            }
        }
        finally {
            // 3. 生成异常报告
            reportFailure(exceptionReporters, exception);
            if (context != null) {
                // 4. 容器关闭
                context.close();
            }
        }
    }
    catch (Exception ex) {
        logger.warn("Unable to close ApplicationContext", ex);
    }
    // 5. 重新抛出异常
    ReflectionUtils.rethrowRuntimeException(exception);
}
```

#### 2-16-1-0 获取退出代码
通过 `getExitCodeFromException` 方法, 获得退出代码. 
如果不为 `0` , 表示异常退出, 发布异常时间.
```java
private void handleExitCode(ConfigurableApplicationContext context, Throwable exception) {
    // 获得退出代码
    int exitCode = getExitCodeFromException(context, exception);
    if (exitCode != 0) {
        if (context != null) {
            // 发布失败事件 ExitCodeEvent
            context.publishEvent(new ExitCodeEvent(context, exitCode));
        }
        // 异常处理类,将当前异常退出代码存入
        SpringBootExceptionHandler handler = getSpringBootExceptionHandler();
        if (handler != null) {
             // 异常退出代码存入 
            handler.registerExitCode(exitCode);
        }
    }
}
```

#### 2-16-1-1 退出代码

获取当前的系统异常, 主要通过 `org.springframework.boot.ExitCodeExceptionMapper` 接口实现和 `org.springframework.boot.ExitCodeGenerator` 接口实现确认具体的异常及其对应的退出代码.

** 如果自定义异常状态码及其捕获异常, 那么在容器抛出相关异常后,则判定容器退出,调用失败事件发布/容器关闭等... **

```java
// 获取当前退出系统代码
private int getExitCodeFromException(ConfigurableApplicationContext context, Throwable exception) {
    // ExitCodeExceptionMapper 接口实现获得
    int exitCode = getExitCodeFromMappedException(context, exception);
    if (exitCode == 0) {
        // ExitCodeGenerator 接口实现获得
        exitCode = getExitCodeFromExitCodeGeneratorException(exception);
    }
    return exitCode;
}

private int getExitCodeFromMappedException(ConfigurableApplicationContext context, Throwable exception) {
    // 容器存在, 且不为已停止的
    if (context == null || !context.isActive()) {
        return 0;
    }
    ExitCodeGenerators generators = new ExitCodeGenerators();
    Collection<ExitCodeExceptionMapper> beans = context.getBeansOfType(ExitCodeExceptionMapper.class).values();
    generators.addAll(exception, beans);
    return generators.getExitCode();
}

private int getExitCodeFromExitCodeGeneratorException(Throwable exception) {
    if (exception == null) {
        return 0;
    }
    if (exception instanceof ExitCodeGenerator) {
        return ((ExitCodeGenerator) exception).getExitCode();
    }
    return getExitCodeFromExitCodeGeneratorException(exception.getCause());
}
```

#### 2-16-2-0 发送监听事件
通过 `SpringApplicationRunListeners` 封装类, 封装了所有的 `SpringApplicationRunListener` 运行监听器接口的实现,作为工具类,调用其中的 `failed` 方法. 遍历每一个监听器,广播一个事件

```java
void failed(ConfigurableApplicationContext context, Throwable exception) {
    // 调用加载的监听器
    for (SpringApplicationRunListener listener : this.listeners) {
        callFailedListener(listener, context, exception);
    }
}

private void callFailedListener(SpringApplicationRunListener listener, ConfigurableApplicationContext context,
                                Throwable exception) {
    try {
        // 1. 调用运行监听器工具对象中的方法
        listener.failed(context, exception);
    }
    catch (Throwable ex) {
        if (exception == null) {
            ReflectionUtils.rethrowRuntimeException(ex);
        }
        if (this.log.isDebugEnabled()) {
            this.log.error("Error handling failed", ex);
        }
        else {
            String message = ex.getMessage();
            message = (message != null) ? message : "no error message";
            this.log.warn("Error handling failed (" + message + ")");
        }
    }
}
```
#### 2-16-2-1 发送事件
`org.springframework.boot.SpringApplicationRunListener`接口定义了一系列发布容器事件,其的具体实现为 `org.springframework.boot.context.event.EventPublishingRunListener` , 在容器失败后,调用 `failed` 方法.
创建一个 `ApplicationFailedEvent` 事件, 并对其进行广播

```java
@Override
public void failed(ConfigurableApplicationContext context, Throwable exception) {
    ApplicationFailedEvent event = new ApplicationFailedEvent(this.application, this.args, context, exception);
    // 如果容器不为空, 且处于活动状态 -> 严重的异常导致容器退出
    if (context != null && context.isActive()) {        
        // 容器发布事件
        context.publishEvent(event);
    }
    else {
        // 容器初始化中, 使用 SimpleApplicationEventMulticaster 容器初始化广播器进行发布事件
        if (context instanceof AbstractApplicationContext) {
            for (ApplicationListener<?> listener : ((AbstractApplicationContext) context)
                 .getApplicationListeners()) {
                this.initialMulticaster.addApplicationListener(listener);
            }
        }
        this.initialMulticaster.setErrorHandler(new LoggingErrorHandler());
        this.initialMulticaster.multicastEvent(event);
    }
}
```



#### 2-16-3-0 生成异常报告

调用 `SpringBootExceptionReporter` 接口实现类中的  `reportException` 方法, 完成异常信息的日志输出

```java
private void reportFailure(Collection<SpringBootExceptionReporter> exceptionReporters, Throwable failure) {
    try {
        for (SpringBootExceptionReporter reporter : exceptionReporters) {
            // 调用异常报告
            if (reporter.reportException(failure)) {
                // 异常处理注册,已经处理过改异常
                registerLoggedException(failure);
                return;
            }
        }
    }
    catch (Throwable ex) {
        // 空处理
    }
    if (logger.isErrorEnabled()) {
        logger.error("Application run failed", failure);
        registerLoggedException(failure);
    }
}
```

#### 2-16-4-0 容器关闭

主要进行容器的关闭和是否移除`JVM` 退出时的关闭事件钩子函数.

```java
@Override
public void close() {
    synchronized (this.startupShutdownMonitor) {
        // 关闭容器
        doClose();
        // 事件钩子, 在JVM退出时执行
        if (this.shutdownHook != null) {
            try {
                Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
            }
            catch (IllegalStateException ex) {
                // ignore - VM is already shutting down
            }
        }
    }
}
```
#### 2-16-4-1 具体的关闭
```java
protected void doClose() {
    // 检查容器是否在激活, 
    if (this.active.get() && this.closed.compareAndSet(false, true)) {
        // 关闭日志
        if (logger.isDebugEnabled()) {
            logger.debug("Closing " + this);
        }
        // 注销应用上下文
        LiveBeansView.unregisterApplicationContext(this);

        try {
            // 发布事件 ContextClosedEvent
            publishEvent(new ContextClosedEvent(this));
        }
        catch (Throwable ex) {
            logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
        }

        // 调用 Bean的生命周期的销毁方法 
        if (this.lifecycleProcessor != null) {
            try {
                this.lifecycleProcessor.onClose();
            }
            catch (Throwable ex) {
                logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
            }
        }

        // 销毁单例的Bean
        destroyBeans();

        // 关闭 Bean 工厂, 将其置空,便于垃圾回收
        closeBeanFactory();

        // 清理工作, 空实现
        onClose();

        // 监听器重置到容器启动前的状态
        if (this.earlyApplicationListeners != null) {
            // 清空
            this.applicationListeners.clear();
            // 重置 
            this.applicationListeners.addAll(this.earlyApplicationListeners);
        }

        // 切换容器状态.
        this.active.set(false);
    }
}
```

#### 2-16-5-0 重新抛出异常

重新抛出异常,交由 `SpringBoot` 框架外的调用者感知, 并作更进一步处理

```java
public static void rethrowRuntimeException(Throwable ex) {
    if (ex instanceof RuntimeException) {
        throw (RuntimeException) ex;
    }
    if (ex instanceof Error) {
        throw (Error) ex;
    }
    // 未被 Spring 处理的异常
    throw new UndeclaredThrowableException(ex);
}
```



# 运行框架

## 配置类解析
在容器创建后, 将当前配置类引入.

### 自定义 `Import`



### 解析入口

 1. `run` 方法中的 `refresh` 方法,刷新当前容器
  2. 其中 `invokeBeanFactoryPostProcessors` 方法会调用所有`BeanDefinitionRegistryPostProcessor` ( 向容器注册 `BeanDefinition` 信息 )接口的所有实现
     其中包括 `ConfigurationClassPostProcessor` 类.
 3. 实现类  `ConfigurationClassPostProcessor` 中的 `postProcessBeanDefinitionRegistry` 方法,进行配置类的解析
   1. 首先获得 `BeanDefinitionRegistry` 的唯一ID, 判断是否已经注册过. 已注册, 抛出异常
   2. 未注册, 添加到已处理集合中, 调用 `processConfigBeanDefinitions` 方法
 4. 在 `processConfigBeanDefinitions` 中. 具体判断如下:
   1. 首先获得注册的 `BeanDefinitionRegistry` 中的 Bean 名称, 判断是否处理过. 处理过则跳过.
   2. 未注册处理,检查其 `configurationClass` 属性判断配置类型,并设置
   3. 处理完成后, 将其加入到 `configCandidates` 集合中.
   4. `configCandidates` 集合根据 `order` 值排序
   5. 遍历集合进行解析处理
   6. 注册 `importRegistry` 并清空



### 源码步骤

首先在 `org.springframework.context.support.AbstractApplicationContext#refresh` 重新刷新容器方法中,通过 `invokeBeanFactoryPostProcessors(beanFactory)` 方法调用 `BeanFactoryPostProcessor` 接口中定义的 `postProcessBeanFactory` 方法, 将配置信息注册.

```java
// org.springframework.context.support.AbstractApplicationContext#refresh 中使用
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 向容器中添加配置类的BeanDefinition信息
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

    if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }
}
```

#### 2-11-5-X 入口方法

在 `PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors` 方法中
让所有 `BeanDefinitionRegistryPostProcessor` 接口实现中, 有 `org.springframework.context.annotation.ConfigurationClassPostProcessor` 实现类, 其中完成了对配置类的后置处理.

```java
public static void invokeBeanFactoryPostProcessors(
    ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
       // ...
    
       // 遍历所有 BeanDefinitionRegistryPostProcessor 接口的实现,调用其中的方法
       invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
    
       // ...
    }
```

其类图如下

![ConfigurationClassPostProcessor类图](./images/2021-02/ConfigurationClassPostProcessor类图-1.png)

#### X-1-0-0 实现类入口

通过 `org.springframework.context.annotation.ConfigurationClassPostProcessor#postProcessBeanFactory` 方法,在容器 `refresh` 方法中, 加载 `Bean`  后对其进行后续处理

```java
@Override
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
    // 首先获得 BeanDefinitionRegistry 用以注册 BeanDefinition 信息
    int registryId = System.identityHashCode(registry);
    // 如果 BeanDefinitionRegistry 已经注册过,则抛出异常
    if (this.registriesPostProcessed.contains(registryId)) {
        throw new IllegalStateException("postProcessBeanDefinitionRegistry already called on this post-processor against " + registry);
    }
    // 如果 BeanFactory 已经进行过后置处理 Bean 后, 则抛出异常
    if (this.factoriesPostProcessed.contains(registryId)) {
        throw new IllegalStateException("postProcessBeanFactory already called on this post-processor against " + registry);
    }
    // BeanDefinitionRegistry 注册 +1
    this.registriesPostProcessed.add(registryId);
	// 调用注册, 对Bean信息进行处理
    processConfigBeanDefinitions(registry);
}
```



#### X-2-0-0 注册配置类

在 `Bean` 创建后,通过类信息注册 `Configuration` 对象, 用以描述类的信息

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    // 待处理 BeanDefinition 对象集合,后续以该集合内元素为根节点,进行扫描; 通过 BeanDefinitionHolder 工具类持有修饰.
    List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
    // 当前上下文中的已经主动添加过的类的 BeanDefinition ; 有多个,其中包括 主启动类 xxxApplication
    String[] candidateNames = registry.getBeanDefinitionNames();
	// 遍历所有的配置类信息
    for (String beanName : candidateNames) {
        // 从 BeanDefinitionRegistry 注册中获取改配置类的信息
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        // 判断 BeanDefinition 中属性 org.springframework.context.annotation.ConfigurationClassPostProcessor.configurationClass
        // 存在,跳过
        if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
            if (logger.isDebugEnabled()) {
                logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
            }
        }
        // 不存在, 检查当前类的定义信息中是否包含 @Configuration 注解, 标记为全配置类
        // 或者 @Import @Component @ImportResource @ComponentScan 注解 或者含有@Bean注解方法, 标记为处理配置
        else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
            // 添加到 待处理 BeanDefinition 对象集合
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    // 当前集合为空, 未有任何配置信息, 返回.
    if (configCandidates.isEmpty()) {
        return;
    }

    // 根据 order 值排序
    configCandidates.sort((bd1, bd2) -> {
        int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
        int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
        return Integer.compare(i1, i2);
    });

    // 是否为 SingletonBeanRegistry 接口子类, 默认为 DefaultListableBeanFactory 为其子类, 尝试添加一些扩展(一般扩展为null).
    SingletonBeanRegistry sbr = null;
    if (registry instanceof SingletonBeanRegistry) {
        sbr = (SingletonBeanRegistry) registry;
        if (!this.localBeanNameGeneratorSet) {
            BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
                AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
            if (generator != null) {
                this.componentScanBeanNameGenerator = generator;
                this.importBeanNameGenerator = generator;
            }
        }
    }
    
	// 环境信息, 一般存在
    if (this.environment == null) {
        this.environment = new StandardEnvironment();
    }

    // 解析 @Configuration 类, 创建解析器
    ConfigurationClassParser parser = new ConfigurationClassParser(
        this.metadataReaderFactory, this.problemReporter, this.environment,
        this.resourceLoader, this.componentScanBeanNameGenerator, registry);
	// 待解析,配置类集合
    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    // 已解析,配置类集合
    Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
    
    // 循环解析
    do {
        // 1. 解析当前配置类
        parser.parse(candidates);
        // 2. 验证. 是否为 final, 不能被复写, 等
        parser.validate();
		// 3. 从parser中获得配置文件的集合赋值给临时集合, 移除已经解析
        Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
        configClasses.removeAll(alreadyParsed);

        // 构建工具类,读取配置
        if (this.reader == null) {
            this.reader = new ConfigurationClassBeanDefinitionReader(
                registry, this.sourceExtractor, this.resourceLoader, this.environment,
                this.importBeanNameGenerator, parser.getImportRegistry());
        }
        // 读取配置类, 如果有相关 BeanDefinition 定义信息, 则进行读取. 包括后续在循环内通过 @ImportResource 导入的
        this.reader.loadBeanDefinitions(configClasses);
        // 已解析,配置类集合添加
        alreadyParsed.addAll(configClasses);
        // 待解析,配置类集合清空
        candidates.clear();
        
        // 如果在解析时, 引入了新的配置类, 则进行添加. 通过判断解析前后, 容器内新添加的BeanDefinition类数量是否大于约定解析时的数量
        if (registry.getBeanDefinitionCount() > candidateNames.length) {
            // 容器内新加载的 BeanDefinition 名字集合
            String[] newCandidateNames = registry.getBeanDefinitionNames();
            // 一开始, 解析前 BeanDefinition 名字集合
            Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
            // 已处理过的集合
            Set<String> alreadyParsedClasses = new HashSet<>();
            for (ConfigurationClass configurationClass : alreadyParsed) {
                alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
            }
            // 循环新添加的
            for (String candidateName : newCandidateNames) {
                if (!oldCandidateNames.contains(candidateName)) {
                    // 新添加的 BeanDefinition 定义信息
                    BeanDefinition bd = registry.getBeanDefinition(candidateName);
                    // 如果是配置类, 且未被解析
                    if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                        !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                        // 待解析, 配置集合添加新的
                        candidates.add(new BeanDefinitionHolder(bd, candidateName));
                    }
                }
            }
            // 本次循环, 要解析 
            candidateNames = newCandidateNames;
        }
    }
    // do - while 结束条件
    while (!candidates.isEmpty());

    // 将 ImportRegistry 注册为Bean，以支持ImportAware @Configuration类
    if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
        sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
    }

    if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
        // 清除共享缓存
        ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
    }
}
```

#### X-2-1-0 解析入口

`do-while` 循环解析中,传入待处理的配置类集合,进行处理调用.

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
    // 遍历 BeanDefinition 集合
    for (BeanDefinitionHolder holder : configCandidates) {
        // 获得配置类的定义 BeanDefinition
        BeanDefinition bd = holder.getBeanDefinition();
        try {
            // 根据配置类的不同, 调用不同的解析器, 解析具体的信息
            if (bd instanceof AnnotatedBeanDefinition) {
                // 注解配置类
                parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
            }
            else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
                // 抽象配置类
                parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
            }
            else {
                // 其他类解析
                parse(bd.getBeanClassName(), holder.getBeanName());
            }
        }
        catch (BeanDefinitionStoreException ex) {
            throw ex;
        }
        catch (Throwable ex) {
            throw new BeanDefinitionStoreException(
                "Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
        }
    }

    this.deferredImportSelectorHandler.process();
}

```

#### X-2-1-1 解析执行

解析方法, 根据不同的配置类,  调用不同的解析方法. 

```java
// 其他类解析
protected final void parse(@Nullable String className, String beanName) throws IOException {
    Assert.notNull(className, "No bean class name for configuration class bean definition");
    MetadataReader reader = this.metadataReaderFactory.getMetadataReader(className);
    processConfigurationClass(new ConfigurationClass(reader, beanName), DEFAULT_EXCLUSION_FILTER);
}

// 抽象配置类
protected final void parse(Class<?> clazz, String beanName) throws IOException {
    processConfigurationClass(new ConfigurationClass(clazz, beanName), DEFAULT_EXCLUSION_FILTER);
}

// 解析注解配置类
protected final void parse(AnnotationMetadata metadata, String beanName) throws IOException {
    // 流程配置类
    processConfigurationClass(new ConfigurationClass(metadata, beanName), DEFAULT_EXCLUSION_FILTER);
}
```

随后构造 `ConfigurationClass` 封装类, 包含了配置类的类描述信息和类资源路径.

```java
public ConfigurationClass(Class<?> clazz, String beanName) {
    Assert.notNull(beanName, "Bean name must not be null");
    this.metadata = AnnotationMetadata.introspect(clazz);
    this.resource = new DescriptiveResource(clazz.getName());
    this.beanName = beanName;
}
```

 具体的配置类解析

```java
protected void processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter) throws IOException {
    // 1. 通过 Condition 接口判断是否可以跳过配置
    if (this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
        return;
    }
    // 2. 获得配置类的配置信息,第一次进入为空. 跳过
    ConfigurationClass existingClass = this.configurationClasses.get(configClass);
    if (existingClass != null) {
        if (configClass.isImported()) {
            if (existingClass.isImported()) {
                existingClass.mergeImportedBy(configClass);
            }
            // Otherwise ignore new imported config class; existing non-imported class overrides it.
            return;
        }
        else {
            // Explicit bean definition found, probably replacing an import.
            // Let's remove the old one and go with the new one.
            this.configurationClasses.remove(configClass);
            this.knownSuperclasses.values().removeIf(configClass::equals);
        }
    }

    // 3. 递归解析配置类及其父类
    SourceClass sourceClass = asSourceClass(configClass, filter);
    do {
        // 4. 实际的核心解析方法, 返回解析入口类的父类,如果父类为配置类则返回, 否则为空结束解析
        sourceClass = doProcessConfigurationClass(configClass, sourceClass, filter);
    }
    while (sourceClass != null);

    this.configurationClasses.put(configClass, configClass);
}
```

#### X-2-1-2 核心解析方法

解析以下情况

- 内部类处理, 如声明内部类.并使用 `@Configuration` 注解
- `PropertySource` 处理, 替换资源占位符,加载资源,添加到环境中.
- `ComponentScan` 处理, 指定扫描路径或类, 或者默认配置类所在路径. 
  - 可指定过滤类规则
  - 默认先执行排除规则,再执行引入规则(优先级更高)
- `Import` 处理. 
  - 处理 `ImportSelector` 和 `DeferredImportSelector` 接口实现中的返回类全路径数组
  - 处理 `ImportBeanDefinitionRegistrar` 中注册的Bean
  - 处理 `@Import` 注解中导入的类, 当做配置类处理
- `ImportResource` 处理, 导入 `spring` 的 `xml` 配置文件
- `BeanMethod` 处理, 如在配置类中使用 `@Bean` 的方法
- 接口默认方法实现, 如在 `default` 方法中使用 `@Bean` 注解
- 父类处理, 对非 `java.*` 的未处理类进行获取其父类递归调用解析. 直至 `java.lang.Object` 完成解析

```java
protected final SourceClass doProcessConfigurationClass(
    ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
    throws IOException {
    // 1. 是否含有 @Component 注解, 在内部类中,是否引入新的配置类
    if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
        // 递归处理内部类.
        processMemberClasses(configClass, sourceClass, filter);
    }

    // 2. 处理 @PropertySource 注解, 引入资源文件 替换SpEL表达式, 将资源引入到环境中
    for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
        sourceClass.getMetadata(), PropertySources.class,
        org.springframework.context.annotation.PropertySource.class)) {
        if (this.environment instanceof ConfigurableEnvironment) {
            processPropertySource(propertySource);
        }
        else {
            logger.info("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
                        "]. Reason: Environment must implement ConfigurableEnvironment");
        }
    }

    // 3. 处理 @ComponentScan 注解, 进行扫描注入
    Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
        sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
    if (!componentScans.isEmpty() &&
        !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
        for (AnnotationAttributes componentScan : componentScans) {
            // The config class is annotated with @ComponentScan -> perform the scan immediately
            Set<BeanDefinitionHolder> scannedBeanDefinitions =
                this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
            // Check the set of scanned definitions for any further config classes and parse recursively if needed
            for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
                BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
                if (bdCand == null) {
                    bdCand = holder.getBeanDefinition();
                }
                // 如果 @ComponentScan 注解的类同时为配置类, 递归调用解析
                if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
                    parse(bdCand.getBeanClassName(), holder.getBeanName());
                }
            }
        }
    }

    // 4. 处理 @Import 注解类. @Configuration 配置类 , 或者 Import 接口实现类
    processImports(configClass, sourceClass, getImports(sourceClass), filter, true);

    // 5. 处理 @ImportResource 注解, 进而读取配置Bean的 xml 文件
    AnnotationAttributes importResource =
        AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
    if (importResource != null) {
        // 读取 locations 属性指定的资源文件
        String[] resources = importResource.getStringArray("locations");
        Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
        for (String resource : resources) {
            // 遍历, 处理资源文件中的占位符
            String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
            // 添加读取资源
            configClass.addImportedResource(resolvedResource, readerClass);
        }
    }

    // 6. @Bean 标注的方法
    Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
    for (MethodMetadata methodMetadata : beanMethods) {
        // 包装为 BeanMethod 类并添加
        configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
    }

    // 7. 处理 接口 的默认方法. 如在默认实现方法中添加 @Bean 注解
    processInterfaces(configClass, sourceClass);

    // 8. 处理父类方法.
    if (sourceClass.getMetadata().hasSuperClass()) {
        String superclass = sourceClass.getMetadata().getSuperClassName();
        // 不为 null , 且不以 java开头, 且尚未处理
        if (superclass != null && !superclass.startsWith("java") &&
            !this.knownSuperclasses.containsKey(superclass)) {
            this.knownSuperclasses.put(superclass, configClass);
            // 找到父类,递归返回.
            return sourceClass.getSuperClass();
        }
    }

    // 没有父类, 递归处理完成
    return null;
}
```

#### X-2-2-0 不同配置解析

根据解析时配置类的类型不同. 调用不同的解析逻辑.

 #### X-2-2-1 处理内置类

```java
private void processMemberClasses(ConfigurationClass configClass, SourceClass sourceClass,
			Predicate<String> filter) throws IOException {
		// 内部类
		Collection<SourceClass> memberClasses = sourceClass.getMemberClasses();
        // 仅处理含有内部类的情况
		if (!memberClasses.isEmpty()) {
            // 内部类中,符合配置类要求的 class 集合
			List<SourceClass> candidates = new ArrayList<>(memberClasses.size());
			for (SourceClass memberClass : memberClasses) {
                // 判断内部类是否包含 @Component, @ComponentScan, @Import, @ImportResource, @Bean方法
				if (ConfigurationClassUtils.isConfigurationCandidate(memberClass.getMetadata()) &&
						!memberClass.getMetadata().getClassName().equals(configClass.getMetadata().getClassName())) {
                    // 添加到配置类集合
					candidates.add(memberClass);
				}
			}
            // 排序
			OrderComparator.sort(candidates);
			for (SourceClass candidate : candidates) {
                // 当前处理栈中是否包含本配置内部类
				if (this.importStack.contains(configClass)) {
                    // 存在抛出异常
					this.problemReporter.error(new CircularImportProblem(configClass, this.importStack));
				}
				else {
                    // 通过栈结构, 解决配置文件之间相互引入问题. 如一个类中有两个内部类,且均使用 @Configuration注解,相互引入.
                    // 不存在, 压入栈中.
					this.importStack.push(configClass);
					try {
                        // 递归调用并处理
						processConfigurationClass(candidate.asConfigClass(configClass), filter);
					}
					finally {
                        // 处理完成, 弹出栈中
						this.importStack.pop();
					}
				}
			}
		}
	}
```

#### X-2-2-2 处理 `@PropertySource` 

在 `@PropertySource` 注解中,可以指定配置文件的路径, 向容器引入新的属性配置信息

```java
private void processPropertySource(AnnotationAttributes propertySource) throws IOException {
    // 注解标识
    String name = propertySource.getString("name");
    if (!StringUtils.hasLength(name)) {
        name = null;
    }
    // 文件编码
    String encoding = propertySource.getString("encoding");
    if (!StringUtils.hasLength(encoding)) {
        encoding = null;
    }
    // 文件路径
    String[] locations = propertySource.getStringArray("value");
    Assert.isTrue(locations.length > 0, "At least one @PropertySource(value) location is required");
    boolean ignoreResourceNotFound = propertySource.getBoolean("ignoreResourceNotFound");
	// 解析工具类
    Class<? extends PropertySourceFactory> factoryClass = propertySource.getClass("factory");
    PropertySourceFactory factory = (factoryClass == PropertySourceFactory.class ?
                                     DEFAULT_PROPERTY_SOURCE_FACTORY : BeanUtils.instantiateClass(factoryClass));
	
    // 遍历文件路径
    for (String location : locations) {
        try {
            // 读取并替换占位符
            String resolvedLocation = this.environment.resolveRequiredPlaceholders(location);
            // 交由 ResourceLoader 读取
            Resource resource = this.resourceLoader.getResource(resolvedLocation);
            // 添加资源集合到环境中
            addPropertySource(factory.createPropertySource(name, new EncodedResource(resource, encoding)));
        }
        catch (IllegalArgumentException | FileNotFoundException | UnknownHostException | SocketException ex) {
            // Placeholders not resolvable or resource not found when trying to open it
            if (ignoreResourceNotFound) {
                if (logger.isInfoEnabled()) {
                    logger.info("Properties location [" + location + "] not resolvable: " + ex.getMessage());
                }
            }
            else {
                throw ex;
            }
        }
    }
}

// 添加属性文件资源集合
private void addPropertySource(PropertySource<?> propertySource) {
    // 属性文件名
    String name = propertySource.getName();
    // 当前环境中已有的属性文件集合
    MutablePropertySources propertySources = ((ConfigurableEnvironment) this.environment).getPropertySources();
	// 存在这个新添加的属性文件, 扩展属性
    if (this.propertySourceNames.contains(name)) {
        // 获得已经存在的同名属性文件
        PropertySource<?> existing = propertySources.get(name);
        if (existing != null) {
            // 新的属性文件
            PropertySource<?> newSource = (propertySource instanceof ResourcePropertySource ?
                                           ((ResourcePropertySource) propertySource).withResourceName() : propertySource);
            //  CompositePropertySource 优先级更高
            if (existing instanceof CompositePropertySource) {
                ((CompositePropertySource) existing).addFirstPropertySource(newSource);
            }
            // 否则合并两个同名属性文件, 并替换前一个
            else {
                if (existing instanceof ResourcePropertySource) {
                    existing = ((ResourcePropertySource) existing).withResourceName();
                }
                CompositePropertySource composite = new CompositePropertySource(name);
                composite.addPropertySource(newSource);
                composite.addPropertySource(existing);
                propertySources.replace(name, composite);
            }
            return;
        }
    }

    // 空. 添加到最后
    if (this.propertySourceNames.isEmpty()) {
        propertySources.addLast(propertySource);
    }
    else {
        // 否则, 添加到集合 List 队尾
        String firstProcessed = this.propertySourceNames.get(this.propertySourceNames.size() - 1);
        propertySources.addBefore(firstProcessed, propertySource);
    }
    this.propertySourceNames.add(name);
}
```



#### X-2-2-3 处理 `@ComponentScan`

通过 `@ComponentScan` 注解, 配置扫描的路径或者类, 将该路径或者类下的同包和子包注入到容器中..默认为被 `@ComponentScan` 注解类作为根扫描入口

`@SpringBootApplication` 注解包含 `@ComponentScan` 注解, 因此 `SpringBoot` 以当前启动类为根入口类, 进行扫描注入.

```java
protected final SourceClass doProcessConfigurationClass(
    // ...
    
    // 处理 @ComponentScans, @ComponentScan 注解
    // 获得注解的属性集合
    Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
        sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
    // 扫描集合
    if (!componentScans.isEmpty() &&
        // Condition 接口, 判断是否跳过该扫描
        !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
        for (AnnotationAttributes componentScan : componentScans) {
            // 需要扫描的入口
            Set<BeanDefinitionHolder> scannedBeanDefinitions =
                this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
            // 对扫描结果进行遍历
            for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
                // BeanDefinition 信息
                BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
                if (bdCand == null) {
                    bdCand = holder.getBeanDefinition();
                }
                // 如果 BeanDefinition 为配置类, 则再次执行 parse 解析逻辑
                if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
                    parse(bdCand.getBeanClassName(), holder.getBeanName());
                }
            }
        }
    }
    
    // ...
}
```

解析需要扫描的入口, 并执行扫描,返回 `BeanDefinition` 信息

```java
public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
    ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry,
                                                                                componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);
	// 指定扫描器
    Class<? extends BeanNameGenerator> generatorClass = componentScan.getClass("nameGenerator");
    boolean useInheritedGenerator = (BeanNameGenerator.class == generatorClass);
    scanner.setBeanNameGenerator(useInheritedGenerator ? this.beanNameGenerator :
                                 BeanUtils.instantiateClass(generatorClass));
	// 扫描方式
    ScopedProxyMode scopedProxyMode = componentScan.getEnum("scopedProxy");
    if (scopedProxyMode != ScopedProxyMode.DEFAULT) {
        scanner.setScopedProxyMode(scopedProxyMode);
    }
    else {
        Class<? extends ScopeMetadataResolver> resolverClass = componentScan.getClass("scopeResolver");
        scanner.setScopeMetadataResolver(BeanUtils.instantiateClass(resolverClass));
    }
	// 指定扫描的表达式
    scanner.setResourcePattern(componentScan.getString("resourcePattern"));

    // 指定需要被扫描的类
    for (AnnotationAttributes filter : componentScan.getAnnotationArray("includeFilters")) {
        for (TypeFilter typeFilter : typeFiltersFor(filter)) {
            scanner.addIncludeFilter(typeFilter);
        }
    }
    // 排序需要被扫描的类
    for (AnnotationAttributes filter : componentScan.getAnnotationArray("excludeFilters")) {
        for (TypeFilter typeFilter : typeFiltersFor(filter)) {
            scanner.addExcludeFilter(typeFilter);
        }
    }

    // 是否需要被延迟加载
    boolean lazyInit = componentScan.getBoolean("lazyInit");
    if (lazyInit) {
        scanner.getBeanDefinitionDefaults().setLazyInit(true);
    }

    // 需要被扫描的基础包集合
    Set<String> basePackages = new LinkedHashSet<>();
    String[] basePackagesArray = componentScan.getStringArray("basePackages");
    for (String pkg : basePackagesArray) {
        String[] tokenized = StringUtils.tokenizeToStringArray(this.environment.resolvePlaceholders(pkg),
                                                               ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
        Collections.addAll(basePackages, tokenized);
    }
    // 需要被扫描的基础包的类,将其同级包或子包纳入扫描范围
    for (Class<?> clazz : componentScan.getClassArray("basePackageClasses")) {
        basePackages.add(ClassUtils.getPackageName(clazz));
    }
    // 如果注解上未有任何声明扫描包范围, 则将当前类所在包作为扫描入口
    if (basePackages.isEmpty()) {
        basePackages.add(ClassUtils.getPackageName(declaringClass));
    }

    // 添加过滤器
    scanner.addExcludeFilter(new AbstractTypeHierarchyTraversingFilter(false, false) {
        @Override
        protected boolean matchClassName(String className) {
            return declaringClass.equals(className);
        }
    });
    // 确定范围后, 执行扫描. 并过滤判断是否需要被注入.
    return scanner.doScan(StringUtils.toStringArray(basePackages));
}
```

`org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan` 执行扫描

```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Assert.notEmpty(basePackages, "At least one base package must be specified");
    // 扫描结果 BeanDefinition
    Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
    // 需要被扫描的包路径
    for (String basePackage : basePackages) {
        Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
        for (BeanDefinition candidate : candidates) {
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
            candidate.setScope(scopeMetadata.getScopeName());
            String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
            if (candidate instanceof AbstractBeanDefinition) {
                postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
            }
            if (candidate instanceof AnnotatedBeanDefinition) {
                AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
            }
            if (checkCandidate(beanName, candidate)) {
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                definitionHolder =
                    AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                beanDefinitions.add(definitionHolder);
                registerBeanDefinition(definitionHolder, this.registry);
            }
        }
    }
    return beanDefinitions;
}
```



#### X-2-2-4 处理 `@Import` 

`@Import` 注解, 指定需要导入的类.

1. `ImportSelector` 接口及子接口实现 (指定导入类的全路径) 
2. `ImportBeanDefinitionRegistrar` 接口实现(主动注入容器)
3. 普通类, 作为 `@Configuration` 标注处理

其中 `ImportSelector` 接口实现会被自动导入容器中, 这是 `SpringBoot` 自动装配的入口.

```java
private void processImports(ConfigurationClass configClass, SourceClass currentSourceClass,
                            Collection<SourceClass> importCandidates, Predicate<String> exclusionFilter,
                            boolean checkForCircularImports) {
	// 不包含要导入的信息
    if (importCandidates.isEmpty()) {
        return;
    }

    // 如果需要检查循环导入, 且发生循环导入
    if (checkForCircularImports && isChainedImportOnStack(configClass)) {
        this.problemReporter.error(new CircularImportProblem(configClass, this.importStack));
    }
    else {
        // 通过栈导入, 避免循环导入问题
        this.importStack.push(configClass);
        // 执行导入逻辑
        try {
            for (SourceClass candidate : importCandidates) {
                // 如果为 ImportSelector 接口实现, 则调用, 获得需要导入的类全路径并递归调用
                if (candidate.isAssignable(ImportSelector.class)) {
                    // Candidate class is an ImportSelector -> delegate to it to determine imports
                    Class<?> candidateClass = candidate.loadClass();
                    ImportSelector selector = ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class,
                                                                                   this.environment, this.resourceLoader, this.registry);
                    Predicate<String> selectorFilter = selector.getExclusionFilter();
                    if (selectorFilter != null) {
                        exclusionFilter = exclusionFilter.or(selectorFilter);
                    }
                    if (selector instanceof DeferredImportSelector) {
                        this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);
                    }
                    else {
                        // 获得需要被导入的类的类名
                        String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
                        // 获得其中的配置类
                        Collection<SourceClass> importSourceClasses = asSourceClasses(importClassNames, exclusionFilter);
                        // 递归调用本身, 再次尝试从中获得需要导入的类
                        processImports(configClass, currentSourceClass, importSourceClasses, exclusionFilter, false);
                    }
                }
                // 如果为 ImportBeanDefinitionRegistrar 接口实现, 在接口实现中主动注入BeanDefinition到容器
                else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
                   Class<?> candidateClass = candidate.loadClass();
                    ImportBeanDefinitionRegistrar registrar =
                        ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class,
                                                             this.environment, this.resourceLoader, this.registry);
                    configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
                }
                // 其他情况, 当前类仅作为被 @Configuration 标注处理
                else {
                    this.importStack.registerImport(
                        currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
                    processConfigurationClass(candidate.asConfigClass(configClass), exclusionFilter);
                }
            }
        }
        catch (BeanDefinitionStoreException ex) {
            throw ex;
        }
        catch (Throwable ex) {
            throw new BeanDefinitionStoreException(
                "Failed to process import candidates for configuration class [" +
                configClass.getMetadata().getClassName() + "]", ex);
        }
        finally {
            this.importStack.pop();
        }
    }
}
```



#### X-2-2-5 处理 `@ImportResource`

将 `Spring` 的配置 `xml` 文件导入容器
在 `do-while` 循环中, 通过 `org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader#loadBeanDefinitions` 读取添加到容器

```java
protected final SourceClass doProcessConfigurationClass(
    ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
    throws IOException {
	// ...
    
    // 处理 @ImportResource 注解
	AnnotationAttributes importResource =
        AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
    if (importResource != null) {
        // xml 路径
        String[] resources = importResource.getStringArray("locations");
        // 获得 BeanDefinition 读取器
        Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
        for (String resource : resources) {
            // 占位符替换
            String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
            // 读取配置文件, 并放入待处理集合
            configClass.addImportedResource(resolvedResource, readerClass);
        }
    }
    
    // ...
}    
```

在该方法中, 先后读取

```java
private void loadBeanDefinitionsForConfigurationClass(
    ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {
	// 判断是否应该跳过
    if (trackedConditionEvaluator.shouldSkip(configClass)) {
        String beanName = configClass.getBeanName();
        if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
            this.registry.removeBeanDefinition(beanName);
        }
        this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
        return;
    }

    // 处理 @Import 注解导入
    if (configClass.isImported()) {
        registerBeanDefinitionForImportedConfigurationClass(configClass);
    }
    // 处理 @Bean 注解导入
    for (BeanMethod beanMethod : configClass.getBeanMethods()) {
        loadBeanDefinitionsForBeanMethod(beanMethod);
    }

    // 处理 @ImportResource 注解导入
    loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
    // 通过 ImportBeanDefinitionRegistrar 接口导入
    loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
}
```

其中 `loadBeanDefinitionsFromImportedResources` 方法为处理 `xml` 配置文件的方法入口

```java
private void loadBeanDefinitionsFromImportedResources(
    Map<String, Class<? extends BeanDefinitionReader>> importedResources) {

    Map<Class<?>, BeanDefinitionReader> readerInstanceCache = new HashMap<>();

    importedResources.forEach((resource, readerClass) -> {
        // Default reader selection necessary?
        if (BeanDefinitionReader.class == readerClass) {
            if (StringUtils.endsWithIgnoreCase(resource, ".groovy")) {
                // When clearly asking for Groovy, that's what they'll get...
                readerClass = GroovyBeanDefinitionReader.class;
            }
            else if (shouldIgnoreXml) {
                throw new UnsupportedOperationException("XML support disabled");
            }
            else {
                // Primarily ".xml" files but for any other extension as well
                readerClass = XmlBeanDefinitionReader.class;
            }
        }

        BeanDefinitionReader reader = readerInstanceCache.get(readerClass);
        if (reader == null) {
            try {
                // Instantiate the specified BeanDefinitionReader
                reader = readerClass.getConstructor(BeanDefinitionRegistry.class).newInstance(this.registry);
                // Delegate the current ResourceLoader to it if possible
                if (reader instanceof AbstractBeanDefinitionReader) {
                    AbstractBeanDefinitionReader abdr = ((AbstractBeanDefinitionReader) reader);
                    abdr.setResourceLoader(this.resourceLoader);
                    abdr.setEnvironment(this.environment);
                }
                readerInstanceCache.put(readerClass, reader);
            }
            catch (Throwable ex) {
                throw new IllegalStateException(
                    "Could not instantiate BeanDefinitionReader class [" + readerClass.getName() + "]");
            }
        }

        // TODO SPR-6310: qualify relative path locations as done in AbstractContextLoader.modifyLocations
        reader.loadBeanDefinitions(resource);
    });
}
```



#### X-2-2-6 处理 `@Bean`

将 `@Bean` 注解标注的方法纳入容器.

```java
protected final SourceClass doProcessConfigurationClass(
    ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
    throws IOException {
    
    // ...
    
    // 处理 @Bean
    Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
    for (MethodMetadata methodMetadata : beanMethods) {
        configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
    }
    
    // ...
}
```

在 `org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions` 方法中, 通过 `this.reader.loadBeanDefinitions(configClasses)` 读取配置文件 `configClass` 中的信息. 最终交给子类 `org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader#loadBeanDefinitionsForConfigurationClass` 方法去读取. 其中包括 `loadBeanDefinitionsForBeanMethod` 处理 `@Bean` 注解

```java
private void loadBeanDefinitionsForConfigurationClass(
    ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {

    if (trackedConditionEvaluator.shouldSkip(configClass)) {
        String beanName = configClass.getBeanName();
        if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
            this.registry.removeBeanDefinition(beanName);
        }
        this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
        return;
    }

    if (configClass.isImported()) {
        registerBeanDefinitionForImportedConfigurationClass(configClass);
    }
    // 处理 @Bean 注解
    for (BeanMethod beanMethod : configClass.getBeanMethods()) {
        loadBeanDefinitionsForBeanMethod(beanMethod);
    }

    loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
    loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
}
```

处理 `@Bean` 注解方法

```java
private void loadBeanDefinitionsForBeanMethod(BeanMethod beanMethod) {
    ConfigurationClass configClass = beanMethod.getConfigurationClass();
    MethodMetadata metadata = beanMethod.getMetadata();
    String methodName = metadata.getMethodName();

    // Condition 判断是否应该被忽略
    if (this.conditionEvaluator.shouldSkip(metadata, ConfigurationPhase.REGISTER_BEAN)) {
        configClass.skippedBeanMethods.add(methodName);
        return;
    }
    if (configClass.skippedBeanMethods.contains(methodName)) {
        return;
    }

    // @Bean 属性
    AnnotationAttributes bean = AnnotationConfigUtils.attributesFor(metadata, Bean.class);
    Assert.state(bean != null, "No @Bean annotation attributes");
    
    // Bean 的名字
    List<String> names = new ArrayList<>(Arrays.asList(bean.getStringArray("name")));
    String beanName = (!names.isEmpty() ? names.remove(0) : methodName);

    // Bean 的别名
    for (String alias : names) {
        this.registry.registerAlias(beanName, alias);
    }

    // 之前是存在其他定义, 覆盖定义
    if (isOverriddenByExistingDefinition(beanMethod, beanName)) {
        if (beanName.equals(beanMethod.getConfigurationClass().getBeanName())) {
            throw new BeanDefinitionStoreException(beanMethod.getConfigurationClass().getResource().getDescription(),
                                                   beanName, "Bean name derived from @Bean method '" + beanMethod.getMetadata().getMethodName() +
                                                   "' clashes with bean name for containing configuration class; please make those names unique!");
        }
        return;
    }

    // 构建 BeanDefinition 描述信息
    ConfigurationClassBeanDefinition beanDef = new ConfigurationClassBeanDefinition(configClass, metadata, beanName);
    beanDef.setSource(this.sourceExtractor.extractSource(metadata, configClass.getResource()));

    if (metadata.isStatic()) {
        // static @Bean method
        if (configClass.getMetadata() instanceof StandardAnnotationMetadata) {
            beanDef.setBeanClass(((StandardAnnotationMetadata) configClass.getMetadata()).getIntrospectedClass());
        }
        else {
            beanDef.setBeanClassName(configClass.getMetadata().getClassName());
        }
        beanDef.setUniqueFactoryMethodName(methodName);
    }
    else {
        // instance @Bean method
        beanDef.setFactoryBeanName(configClass.getBeanName());
        beanDef.setUniqueFactoryMethodName(methodName);
    }

    if (metadata instanceof StandardMethodMetadata) {
        beanDef.setResolvedFactoryMethod(((StandardMethodMetadata) metadata).getIntrospectedMethod());
    }

    // 默认使用,构造器自动注入
    beanDef.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR);
    // 跳过依赖检查
    beanDef.setAttribute(org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor.
                         SKIP_REQUIRED_CHECK_ATTRIBUTE, Boolean.TRUE);

    AnnotationConfigUtils.processCommonDefinitionAnnotations(beanDef, metadata);

    // 是否自动注入
    Autowire autowire = bean.getEnum("autowire");
    if (autowire.isAutowire()) {
        beanDef.setAutowireMode(autowire.value());
    }

    // 自动判断类型
    boolean autowireCandidate = bean.getBoolean("autowireCandidate");
    if (!autowireCandidate) {
        beanDef.setAutowireCandidate(false);
    }

    // bean 初始化方法
    String initMethodName = bean.getString("initMethod");
    if (StringUtils.hasText(initMethodName)) {
        beanDef.setInitMethodName(initMethodName);
    }

    // bean 销毁方法
    String destroyMethodName = bean.getString("destroyMethod");
    beanDef.setDestroyMethodName(destroyMethodName);

    // bean 的 scope. 是否开启代理
    ScopedProxyMode proxyMode = ScopedProxyMode.NO;
    AnnotationAttributes attributes = AnnotationConfigUtils.attributesFor(metadata, Scope.class);
    if (attributes != null) {
        beanDef.setScope(attributes.getString("value"));
        proxyMode = attributes.getEnum("proxyMode");
        if (proxyMode == ScopedProxyMode.DEFAULT) {
            proxyMode = ScopedProxyMode.NO;
        }
    }

    // 代理处理
    BeanDefinition beanDefToRegister = beanDef;
    if (proxyMode != ScopedProxyMode.NO) {
        BeanDefinitionHolder proxyDef = ScopedProxyCreator.createScopedProxy(
            new BeanDefinitionHolder(beanDef, beanName), this.registry,
            proxyMode == ScopedProxyMode.TARGET_CLASS);
        beanDefToRegister = new ConfigurationClassBeanDefinition(
            (RootBeanDefinition) proxyDef.getBeanDefinition(), configClass, metadata, beanName);
    }

    // 日志....
    if (logger.isTraceEnabled()) {
        logger.trace(String.format("Registering bean definition for @Bean method %s.%s()",
                                   configClass.getMetadata().getClassName(), beanName));
    }
    // 注册 BeanDefinition信息到工厂中
    this.registry.registerBeanDefinition(beanName, beanDefToRegister);
}
```

#### X-2-2-7 处理接口默认方法

在 `org.springframework.context.annotation.ConfigurationClassParser#processInterfaces` 中, 对 `interface` 接口类的 `default` 默认实现进行处理. 
一般在其默认实现上添加 `@Bean` 注解

```java
private void processInterfaces(ConfigurationClass configClass, SourceClass sourceClass) throws IOException {
    // 循环处理接口类
    for (SourceClass ifc : sourceClass.getInterfaces()) {
        // 获得 @Bean 注解信息
        Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(ifc);
        for (MethodMetadata methodMetadata : beanMethods) {
            if (!methodMetadata.isAbstract()) {
                // 处理 @Bean 注解
                configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
            }
        }
        // 递归调用本身
        processInterfaces(configClass, ifc);
    }
}
```

#### X-2-2-8 处理父类

在 `org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass` 中, 对父类进行处理

```java
protected final SourceClass doProcessConfigurationClass(
    ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
    throws IOException {
    
    // ...
        
    // 处理父类
    if (sourceClass.getMetadata().hasSuperClass()) {
        // 获得父类的全路径
        String superclass = sourceClass.getMetadata().getSuperClassName();
        // 如果不为 java.* 开头, 且未被处理
        if (superclass != null && !superclass.startsWith("java") &&
            !this.knownSuperclasses.containsKey(superclass)) {
            // 放入未处理的集合当中
            this.knownSuperclasses.put(superclass, configClass);
            // 返回当前类的父类
            return sourceClass.getSuperClass();
        }
    }
    
    // ...
}
```





## Servlet-Tomcat 容器启动

在 `SpringBoot` 中, 如果你是 `Servlet` 容器, 则会启动默认 `Tomcat` 容器

### 启动流程

#### 1-3-0-0 判断容器类型

在容器初始化时, 会判断容器类型.

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    // 判断容器类型
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

#### 1-3-1-0 根据类判断

根据当前类路径下是否含有某些特定类,判断容器类型.

- `reactive` 响应式容器
- `Servlet` 容器
- 非 `web` 容器

```java
static WebApplicationType deduceFromClasspath() {
    // org.springframework.web.reactive.DispatcherHandler
    if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) 
        // org.springframework.web.servlet.DispatcherServlet
        && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
        // org.glassfish.jersey.servlet.ServletContainer
        && !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
        // 响应式容器
        return WebApplicationType.REACTIVE;
    }
    // { "javax.servlet.Servlet", "org.springframework.web.context.ConfigurableWebApplicationContext" }
    for (String className : SERVLET_INDICATOR_CLASSES) {
        if (!ClassUtils.isPresent(className, null)) {
            // 非 Web 环境
            return WebApplicationType.NONE;
        }
    }
    // Servlet 容器
    return WebApplicationType.SERVLET;
}
```

#### 2-8-0-0 创建应用上下文
在容器启动 `run` 方法时, 根据容器类型的不同,创建不同的应用上下文
`context = createApplicationContext();`  方法调用
具体为判断当前容器类型,并调用不同的创建过程

```java
protected ConfigurableApplicationContext createApplicationContext() {
    Class<?> contextClass = this.applicationContextClass;
    if (contextClass == null) {
        try {
            // 根据容器类型获得不同的容器类全路径
            switch (this.webApplicationType) {
                case SERVLET:
                    // org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext
                    contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
                    break;
                case REACTIVE:
                    // org.springframework.boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext
                    contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
                    break;
                default:
                    // org.springframework.context.annotation.AnnotationConfigApplicationContext
                    contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
            }
        }
        catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                "Unable create a default ApplicationContext, please specify an ApplicationContextClass", ex);
        }
    }
    // 使用类加载机制, 获得指定类的实例
    return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```

#### 2-11-9-0 容器刷新
在容器刷新时. 会对Web容器进行初始化
`org.springframework.context.support.AbstractApplicationContext#refresh` 方法中, 调用 `org.springframework.context.support.AbstractApplicationContext#onRefresh` 方法, 将

```java
protected void onRefresh() throws BeansException {
    // 交由子类方法实现
    
}
```
#### 2-11-9-1 容器创建
`org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#onRefresh` 子类方法具体实现了容器的创建

```java
@Override
protected void onRefresh() {
    // 调用父类 AbstractApplicationContext 的方法
    super.onRefresh();
    try {
        // 创建 Servlet 容器
        createWebServer();
    }
    catch (Throwable ex) {
        throw new ApplicationContextException("Unable to start web server", ex);
    }
}
```
#### 2-11-9-2 获得容器工厂

其中的 `WebServerStartStopLifecycle` 在容器刷新完成后调用, 并启动 `Tomcat`.

```java
private void createWebServer() {
    // 一般为 null
    WebServer webServer = this.webServer;
    // 一般为 null
    ServletContext servletContext = getServletContext();
    if (webServer == null && servletContext == null) {
        // 3. 获得 ServletWebServer 的工厂类, 有且只有一个
        ServletWebServerFactory factory = getWebServerFactory();
        // getSelfInitializer() web容器初始化方法. 默认空
        
        // 4. getWebServer() 创建 servlet 容器 
        this.webServer = factory.getWebServer(getSelfInitializer());
        // 注册关闭函数
        getBeanFactory().registerSingleton("webServerGracefulShutdown",
                                           new WebServerGracefulShutdownLifecycle(this.webServer));
        // 注册启动/关闭程序
        getBeanFactory().registerSingleton("webServerStartStop",
                                           new WebServerStartStopLifecycle(this, this.webServer));
    }
    // 一般容器存在.
    else if (servletContext != null) {
        try {
            getSelfInitializer().onStartup(servletContext);
        }
        catch (ServletException ex) {
            throw new ApplicationContextException("Cannot initialize servlet context", ex);
        }
    }
    // 5. 属性赋值, 将 servletContextInitParams 和 servletConfigInitParams 属性集进行赋值.
    initPropertySources();
}

// 3. 获得 ServletWebServer 的工厂类, 有且只有一个
protected ServletWebServerFactory getWebServerFactory() {
    // 默认Tomcat tomcatServletWebServerFactory
    String[] beanNames = getBeanFactory().getBeanNamesForType(ServletWebServerFactory.class);
    if (beanNames.length == 0) {
        throw new ApplicationContextException("Unable to start ServletWebServerApplicationContext due to missing "
                                              + "ServletWebServerFactory bean.");
    }
    if (beanNames.length > 1) {
        throw new ApplicationContextException("Unable to start ServletWebServerApplicationContext due to multiple "
                                              + "ServletWebServerFactory beans : " + StringUtils.arrayToCommaDelimitedString(beanNames));
    }
    // 实例化工厂类
    return getBeanFactory().getBean(beanNames[0], ServletWebServerFactory.class);
}
```

#### 2-11-9-4 创建 `Tomcat` 容器

在不同容器中,有不同的创建方式,其中 `Tomcat` 为 
`org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory#getWebServer` 方法. 创建对应 `Tomcat`

```java
@Override
public WebServer getWebServer(ServletContextInitializer... initializers) {
    if (this.disableMBeanRegistry) {
        Registry.disableRegistry();
    }
    // 创建
    Tomcat tomcat = new Tomcat();
    // 设置属性 ... 已经读取到配置文件中
    File baseDir = (this.baseDirectory != null) ? this.baseDirectory : createTempDir("tomcat");
    tomcat.setBaseDir(baseDir.getAbsolutePath());
    Connector connector = new Connector(this.protocol);
    connector.setThrowOnFailure(true);
    tomcat.getService().addConnector(connector);
    customizeConnector(connector);
    tomcat.setConnector(connector);
    tomcat.getHost().setAutoDeploy(false);
    configureEngine(tomcat.getEngine());
    for (Connector additionalConnector : this.additionalTomcatConnectors) {
        tomcat.getService().addConnector(additionalConnector);
    }
    prepareContext(tomcat.getHost(), initializers);
    // 4. 返回Tomcat 容器,并启动
    return getTomcatWebServer(tomcat);
}
```

#### 2-11-9-5 属性赋值

属性赋值  最终调用`org.springframework.web.context.support.WebApplicationContextUtils#initServletPropertySources(org.springframework.core.env.MutablePropertySources, javax.servlet.ServletContext, javax.servlet.ServletConfig)` 方法进行环境属性的赋值, 将当前属性进行替换

```java
public static void initServletPropertySources(MutablePropertySources sources,
                                              @Nullable ServletContext servletContext, @Nullable ServletConfig servletConfig) {

    Assert.notNull(sources, "'propertySources' must not be null");
    // servletContextInitParams 参数属性
    String name = StandardServletEnvironment.SERVLET_CONTEXT_PROPERTY_SOURCE_NAME;
    if (servletContext != null && sources.contains(name) && sources.get(name) instanceof StubPropertySource) {
        sources.replace(name, new ServletContextPropertySource(name, servletContext));
    }
    // servletConfigInitParams 参数属性
    name = StandardServletEnvironment.SERVLET_CONFIG_PROPERTY_SOURCE_NAME;
    if (servletConfig != null && sources.contains(name) && sources.get(name) instanceof StubPropertySource) {
        sources.replace(name, new ServletConfigPropertySource(name, servletConfig));
    }
}
```



#### 2-11-12-0 `Tomcat` 容器启动

在 `org.springframework.context.support.AbstractApplicationContext#finishRefresh` 容器刷新完成方法中. 
调用 `getLifecycleProcessor().onRefresh();` 

```java
// 容器刷新
protected void finishRefresh() {
    // Clear context-level resource caches (such as ASM metadata from scanning).
    clearResourceCaches();

    // Initialize lifecycle processor for this context.
    initLifecycleProcessor();

    // 获得带有生命周期的程序,执行其 onRefresh方法
    getLifecycleProcessor().onRefresh();

    // Publish the final event.
    publishEvent(new ContextRefreshedEvent(this));

    // Participate in LiveBeansView MBean, if active.
    if (!IN_NATIVE_IMAGE) {
        LiveBeansView.registerApplicationContext(this);
    }
}
```

默认实现为 `org.springframework.context.support.DefaultLifecycleProcessor` 其中调用 `startBeans` 方法, 将调用所有的带有生命周期的Bean.
其中之一为 `org.springframework.boot.web.servlet.context.WebServerStartStopLifecycle` 将启动

```java
@Override
public void onRefresh() {
    startBeans(true);
    this.running = true;
}

// 调用所有的带有生命周期的Bean
private void startBeans(boolean autoStartupOnly) {
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();
    Map<Integer, LifecycleGroup> phases = new TreeMap<>();
	
    // 调用列表 phase 排序赋值
    lifecycleBeans.forEach((beanName, bean) -> {
        if (!autoStartupOnly || (bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())) {
            int phase = getPhase(bean);
            phases.computeIfAbsent(
                phase,
                // 转为包装类 LifecycleGroup, 为 启动-关闭 两个过程进行排序.
                p -> new LifecycleGroup(phase, this.timeoutPerShutdownPhase, lifecycleBeans, autoStartupOnly)
            ).add(beanName, bean);
        }
    });
    // 集合不为空, 调用其启动方法. 其中包括 org.springframework.boot.web.servlet.context.WebServerStartStopLifecycle 类
    if (!phases.isEmpty()) {
        phases.values().forEach(LifecycleGroup::start);
    }
}
```

#### 2-11-12-1 `Tomcat` 容器启动

`org.springframework.boot.web.servlet.context.WebServerStartStopLifecycle` 类中, 定义容器启动方法. 在 `Tomcat` 容器创建阶段已经被创建, 并注入到 `SpringICO` 上下文中
在此阶段被调用, 启动 `Tomcat` 容器

```java
WebServerStartStopLifecycle(ServletWebServerApplicationContext applicationContext, WebServer webServer) {
    this.applicationContext = applicationContext;
    this.webServer = webServer;
}

@Override
public void start() {
    this.webServer.start();
    this.running = true;
    this.applicationContext
        .publishEvent(new ServletWebServerInitializedEvent(this.webServer, this.applicationContext));
}
```

以下为 `Tomcat` 容器启动类, `org.springframework.boot.web.embedded.tomcat.TomcatWebServer`

```java
@Override
public void start() throws WebServerException {
    synchronized (this.monitor) {
        if (this.started) {
            return;
        }
        try {
            addPreviouslyRemovedConnectors();
            Connector connector = this.tomcat.getConnector();
            if (connector != null && this.autoStart) {
                performDeferredLoadOnStartup();
            }
            checkThatConnectorsHaveStarted();
            this.started = true;
            logger.info("Tomcat started on port(s): " + getPortsDescription(true) + " with context path '"
                        + getContextPath() + "'");
        }
        catch (ConnectorStartFailedException ex) {
            stopSilently();
            throw ex;
        }
        catch (Exception ex) {
            PortInUseException.throwIfPortBindingException(ex, () -> this.tomcat.getConnector().getPort());
            throw new WebServerException("Unable to start embedded Tomcat server", ex);
        }
        finally {
            Context context = findContext();
            ContextBindings.unbindClassLoader(context, context.getNamingToken(), getClass().getClassLoader());
        }
    }
}
```



### Tomcat 配置流程

1. 配置 `web` 属性文件 `application.properties` 
2. 注入到 `ServerProperties` 类中
3. 自动配置类导入 `WebServerFactoryCustomizer` 实现类, 如 `TomcatServletWebServerFactoryCustomizer` 引入 `Tomcat`
4. 将 `ServerProperties` 成为实现类的属性, 包含配置信息
5. 在 `ServletFactory` 工厂类初始化时, `getWebServiceFactory` 获得具体 `web` 服务工厂类
6. `IOC` 容器对具体实现类调用 `doGetBean` 进行初始化, 创建实例
   其中, 遍历 `BeanPostProcessor` 实现类, 对 `Bean` 进行处理. 
7. 与之相关的后置处理器为, `WebServerFactoryCustomizerBeanPostProcessor` 中的后置处理方法
8. `postProcessBeforeInitialization` 方法调用 `getCustomizers` 方法获得 `WebServerFactoryCustomizer` 实现类
   依次调用实现类的 `customize` 方法进行定制处理

### 加载 `Web` 组件

在 `Tomcat` 启动时, 将 `Spring` 扫描的 `Bean` 自动注入到容器中, 为 `Web` 容器加载 `Servlet` , `FIlter` , `Listener` ...

上古版本, 在外置 `tomcat` 启动时, 为 `Web` 容器添加什么....比如过滤器,拦截器什么.
`servlet3.0` 规范中,约定,在 `META-INF/services` 目录下文件 `javax.servlet.ServletContainerInitializer` 中写入其同名接口具体实现.
以便在Web容器启动时,通过java配置容器对象,实现注入组件.
`Spring` 中默认使用 `org.springframework.web.SpringServletContainerInitializer` 类来约定容器启动时的加载动作.
在实现类中, 具体是通过调用 `WebApplicationInitializer` 接口的具体实现类完成调用.

不过现在都是使用内置 `Tomcat` , 则调用
`org.springframework.boot.web.embedded.tomcat.TomcatStarter#onStartup` 具体实现.
从而注入 `ServletContextInitializer` 接口的具体实现类


实现 `ServletContextInitializer` 接口, 在容器启动执行相关方法. 常用于注册 `Web` 中的 `servlet`, `filter`, `listener` 等

`Spring` 提供相关子类有

*  `DispatcherServletRegistrationBean` 注册一个新的 `DispatcherServlet` ,处理请求
*  `ServletRegistrationBean` 注册新的 `Servlet` 请求处理器,对某个请求进行响应.
*   `FilterRegistrationBean`  注册新的 `Filter` 请求过滤器,对请求进行过滤
*  `DelegatingFilterProxyRegistrationBean` 注册新的 `Filter` 请求过滤器的代理对象
*  `ServletListenerRegistrationBean` 注册新的 `Listener` 监听器,对容器进行监听

#### 自定义 `Web` 组件

通过实现 `org.springframework.boot.web.servlet.ServletContextInitializer` 接口, 在容器启动时配置 `Web` 容器上下文

```java
@Component
public class MyServletContextInitializer implements ServletContextIni  tializer {

    /**
     * 为 Web容器添加什么..
     *
     * @param servletContext Web 容器
     */
    @Override
    public void onStartup(ServletContext servletContext) {
        System.out.println("Web容器,启动内置Tomcat启动....." + servletContext);
    }
}
```

自定义 `Servlet` 服务

```java
public class MyHttpServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        System.out.println("Web容器,处理请求...." + request.getRequestURI() + "  " + response.getStatus());
        response.getWriter().print("hello");
    }
}

```

自定义 `Filter` 过滤器

```java
public class MyHttpFilter extends HttpFilter {

    @Override
    public void init(FilterConfig filterConfig) {
        System.out.println("Web容器,自定义过滤器初始化..." + filterConfig);
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("Web容器,自定义过滤器执行..." + request.getRemoteAddr() + "  " + response.getContentType());
        super.doFilter(request, response, chain);
    }

    @Override
    public void destroy() {
        System.out.println("Web容器,自定义过滤器销毁...");
    }
}
```

自定义 `Listener` 监听器

```java
public class MyServletContextListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("Web容器,监听到容器初始化事件..." + sce.getServletContext());
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("Web容器,监听到容器销毁事件..." + sce.getServletContext());
    }
}
```

通过配置向容器中注册...

```java
/**
 * 向 Web 容器中添加三大组件..
 * 自定义请求处理器, 过滤器, 监听器..
 *
 * @author Jion
 */
@Configuration
public class WebContextConfig {

    /***
     *  注册自定义 servlet
     * @return 自定义 servlet
     */
    @Bean
    public ServletRegistrationBean<? extends Servlet> myHttpServlet() {
        ServletRegistrationBean<MyHttpServlet> registrationBean = new ServletRegistrationBean<>();
        MyHttpServlet myHttpServlet = new MyHttpServlet();
        registrationBean.setServlet(myHttpServlet);
        registrationBean.setName("myHttpServlet");
        registrationBean.addUrlMappings("/my-servlet");
        registrationBean.setLoadOnStartup(100);
        return registrationBean;
    }

    /**
     * 注册自定义 Filter
     * @return 自定义 Filter
     */
    @Bean
    public FilterRegistrationBean<? extends Filter> myHttpFilter() {
        FilterRegistrationBean<MyHttpFilter> registrationBean = new FilterRegistrationBean<>();
        MyHttpFilter myHttpFilter = new MyHttpFilter();
        registrationBean.setFilter(myHttpFilter);
        registrationBean.setUrlPatterns(Collections.singleton("/my-servlet"));
        registrationBean.setOrder(100);
        return registrationBean;
    }

    /**
     * 注册自定义 Listener
     * @return 自定义 Listener
     */
    @Bean
    public ServletListenerRegistrationBean<? extends EventListener> myHttpListen() {
        ServletListenerRegistrationBean<MyServletContextListener> registrationBean = new ServletListenerRegistrationBean<>();
        MyServletContextListener myServletContextListener = new MyServletContextListener();
        registrationBean.setListener(myServletContextListener);
        registrationBean.setOrder(100);
        return registrationBean;
    }
}
```

### 加载 `Web ` 组件流程解

#### X-1-0-0 入口方法

在获得 `TomcatWebServer` 时, 会去执行方法 `initialize`. 其中的 `Tomcat` 会被调用执行.

```java
public TomcatWebServer(Tomcat tomcat, boolean autoStart, Shutdown shutdown) {
    Assert.notNull(tomcat, "Tomcat Server must not be null");
    this.tomcat = tomcat;
    this.autoStart = autoStart;
    this.gracefulShutdown = (shutdown == Shutdown.GRACEFUL) ? new GracefulShutdown(tomcat) : null;
    initialize();
}
// org.springframework.boot.web.embedded.tomcat.TomcatWebServer#initialize
private void initialize() throws WebServerException {
    logger.info("Tomcat initialized with port(s): " + getPortsDescription(false));
    synchronized (this.monitor) {
        try {
            addInstanceIdToEngineName();

            Context context = findContext();
            context.addLifecycleListener((event) -> {
                if (context.equals(event.getSource()) && Lifecycle.START_EVENT.equals(event.getType())) {
                    // Remove service connectors so that protocol binding doesn't
                    // happen when the service is started.
                    removeServiceConnectors();
                }
            });

            // Tomcat 执行...
            this.tomcat.start();

            // We can re-throw failure exception directly in the main thread
            rethrowDeferredStartupExceptions();

            try {
                ContextBindings.bindClassLoader(context, context.getNamingToken(), getClass().getClassLoader());
            }
            catch (NamingException ex) {
                // Naming is not enabled. Continue
            }

            // Unlike Jetty, all Tomcat threads are daemon threads. We create a
            // blocking non-daemon to stop immediate shutdown
            startDaemonAwaitThread();
        }
        catch (Exception ex) {
            stopSilently();
            destroySilently();
            throw new WebServerException("Unable to start embedded Tomcat", ex);
        }
    }
}
```

#### X-1-1-0 内置 `Tomcat` 

在内置 `Tomcat` 中, `tomcat-embed-core-9.0.36.jar` 文件, 扩展了 `javax.servlet` 包, 并对其中的 `servlet` 容器规范进行实现. 

该接口会在容器启动时,进行调用. 具体实现类为 `org.springframework.boot.web.embedded.tomcat.TomcatStarter` 

```java
public interface ServletContainerInitializer {
    void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException;
}
```

#### X-1-2-0 启动拦截

在 `Tomcat` 启动时的 `onStartup()` 方法, 会去获得 `org.springframework.boot.web.servlet.ServletContextInitializer` 接口的实现类, 调用其中的 `onStartup` 方法, 并传入 `Servlet` 容器.

```java
class TomcatStarter implements ServletContainerInitializer {

	private static final Log logger = LogFactory.getLog(TomcatStarter.class);

	private final ServletContextInitializer[] initializers;

	private volatile Exception startUpException;

	TomcatStarter(ServletContextInitializer[] initializers) {
		this.initializers = initializers;
	}

	@Override
	public void onStartup(Set<Class<?>> classes, ServletContext servletContext) throws ServletException {
		try {
            // org.springframework.boot.web.servlet.ServletContextInitializer 接口实现类, 调用 onstart方法
			for (ServletContextInitializer initializer : this.initializers) {
				initializer.onStartup(servletContext);
			}
		}
		catch (Exception ex) {
			this.startUpException = ex;
			// Prevent Tomcat from logging and re-throwing when we know we can
			// deal with it in the main thread, but log for information here.
			if (logger.isErrorEnabled()) {
				logger.error("Error starting Tomcat context. Exception: " + ex.getClass().getName() + ". Message: "
						+ ex.getMessage());
			}
		}
    }
}
```

`Spring` 提工的拦截扩展 `Tomcat` 容器中组件的入口

```java
package org.springframework.boot.web.servlet;

@FunctionalInterface
public interface ServletContextInitializer {
	void onStartup(ServletContext servletContext) throws ServletException;

}
```

#### X-1-3-0 加载具体

在 `org.springframework.boot.web.servlet.ServletContextInitializer` 接口实现中,其中最多的是 `org.springframework.boot.web.servlet.RegistrationBean` 抽象实现类, 该抽象实现类约定了在容器启动时进行 `Bean` 的注册, 主要是为注册Web三大组件.

```java
public abstract class RegistrationBean implements ServletContextInitializer, Ordered {
	// Tomcat 启动时拦截扩展执行.
	@Override
	public final void onStartup(ServletContext servletContext) throws ServletException {
		String description = getDescription();
		if (!isEnabled()) {
			logger.info(StringUtils.capitalize(description) + " was not registered (disabled)");
			return;
		}
        // 注册Bean
		register(description, servletContext);
	}
	
    // 子类实现, 向容器中注册Bean
	protected abstract void register(String description, ServletContext servletContext);
}
```

其中的  `register()` 方法为子类具体实现. 向容器注入..具体子类有

1. `org.springframework.boot.web.servlet.ServletListenerRegistrationBean` 具体实现类注册`Linsener` 监听器

2. `org.springframework.boot.web.servlet.DynamicRegistrationBean` 抽象类, 注册 `Servlet` 和 `Filter`

   1. `org.springframework.boot.web.servlet.ServletRegistrationBean` 实现类, 注册 `servlet`

   2. `org.springframework.boot.web.servlet.FilterRegistrationBean` 实现类, 注册 `filter`
   3. `org.springframework.boot.web.servlet.DelegatingFilterProxyRegistrationBean` 实现类, 注册 `filter` 的增强代理
   4. `org.springframework.boot.autoconfigure.web.servlet.DispatcherServletRegistrationBean` 实现类, 注册 `dispathServlet`..自定义请求拦截.



## 场景加载

`SpringBoot` 提供个的各种 `starter-xxx` 

### `Conditional` 注解

在某个条件下将当前 `Bean` 注入

**常用注解**

- `Conditional` 通用注解, 指定具体实现类依据
- `ConditionalOnBean`  在某个 `Bean` 存在下生效
- `ConditionalOnClass` 在某个类存在下生效
- `ConditionalOnMissingBean` 在某个 `Bean` 存在下不生效
- `ConditionalOnMissingClass` 在某个类存在下不生效
- `ConditionalOnProperty` 在某个环境属性下生效
- `ConditionalOnWebApplication` 在 `Web` 环境下生效
- `ConditionalOnNotWebApplication` 在 `Web` 环境下不生效
- `ConditionalOnCloudPlatform`  在 `Cloud` 平台下生效
- `ConditionalOnExpression` 在某个`SpEL` 表达成立时生效
- `ConditionalOnJava` 在指定`JDK`版本下生效
- `ConditionalOnJndi` 在 `JNDI` 环境下生效
- `ConditionalOnResource` 在类路径下有指定值时生效
- `ConditionalOnSingleCandidate` 在容器中当前 `Bean` 只有一个时生效
- `ConditionalOnWarDeployment`在 `War`包部署下生效

### 示例 - 自定义 `ConditionOn` 注解

1. 实现 `org.springframework.context.annotation.Condition` 接口
   或者其子接口 `org.springframework.context.annotation.ConfigurationCondition` 中的 `matches` 方法.
2. 自定义注解, 并组合使用 `@Conditional` 注解, 指定具体实现类
3. 使用.

自定义 `Condition` 接口方法

```java
public class OnWebApplicationCondition implements Condition {
    /**
     *  自定义匹配规则
     *
     * @param context Spring提供的上下文信息, 包含Spring中的核心类及上下文信息
     * @param metadata Spring提供的注解信息, 包裹注解属性.. 等
     * @return 是否生效
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        System.out.println("注解信息" + metadata);
        System.out.println("上下文信息" + context);
        return true;
    }
}

```

自定义 `@Conditional` 的子注解, 指定自定义的类

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnWebApplicationCondition.class)
public @interface ConditionalOnWebApplication {

}
```

### 示例 - 自定义 `starter` 场景启动器

创建一个可以插板的插件

1. 新建 `SpringBoot` 项目
2. 引入 `spring-boot-autoconfigure` 依赖
3. 编写属性源与自动配置类, 确定合适的时机引入
4. 在 `spring.factories` 添加 `org.springframework.boot.autoconfigure.EnableAutoConfiguration` 的具体实现.
5. `Maven` 打包后, 对外可以使用.

## 自动配置原理

### 场景启动器加载流程

1. 启动类, 注解  `@SpringBootApplication`  暗含有 `@EnableAutoConfiguration` 注解
2. 通过 `@import`  方式, 引入 `org.springframework.boot.autoconfigure.AutoConfigurationImportSelector` 类.
3. 在 `org.springframework.context.annotation.ConfigurationClassParser.DeferredImportSelectorGroupingHandler#processGroupImports` 方法中被处理, 调用 `AutoConfigurationImportSelector` 类方法
4. 在 `process` 方法中, 通过 `spring.factories` 配置文件, 获得`org.springframework.boot.autoconfigure.EnableAutoConfiguration` 的具体实现.
5. 过滤. 去重, 将各种 `...AutoConfiguration` 类注入容器.



#### 2-11-5-X 入口方法

在重新刷新容器时, 会执行方法. 进入到 `AutoConfigurationImportSelector` 中.

```java
@Override
public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
    // 断言. 是否有不期望的类出现
    Assert.stprocessate(deferredImportSelector instanceof AutoConfigurationImportSelector,
                 () -> String.format("Only %s implementations are supported, got %s",
                                     AutoConfigurationImportSelector.class.getSimpleName(),
                                     deferredImportSelector.getClass().getName()));
    // 获得 自动配置类.
    AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector) deferredImportSelector)
        .getAutoConfigurationEntry(annotationMetadata);
    // 添加导入
    this.autoConfigurationEntries.add(autoConfigurationEntry);
    for (String importClassName : autoConfigurationEntry.getConfigurations()) {
        this.entries.putIfAbsent(importClassName, annotationMetadata);
    }
}
```

#### X-1-0-0 启动方法

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    // 注解属性
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 1. 声明需要自动配置的类
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    // 2. 去重. 
    configurations = removeDuplicates(configurations);
    // 排除某些不期望注入的类
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    // 3. 获得要排除的具体类
    checkExcludedClasses(configurations, exclusions);
    // 执行排除逻辑.
    configurations.removeAll(exclusions);
    // 4. 通过条件注入, 过滤不需要的配置类.
    configurations = getConfigurationClassFilter().filter(configurations);
    // 5. 发布事件
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}
```



#### X-1-1-0 获得声明实现

读取项目依赖下的 `spring.factories` 文件中的依赖自动注入. 其中包括自定义的.

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
                                                                         getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
                    + "are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

#### X-1-2-0 去重

通过新建 `LinkedHashSet` 去去重

```java
protected final <T> List<T> removeDuplicates(List<T> list) {
    return new ArrayList<>(new LinkedHashSet<>(list));
}
```

#### X-1-3-0 排除

```java
private void checkExcludedClasses(List<String> configurations, Set<String> exclusions) {
    List<String> invalidExcludes = new ArrayList<>(exclusions.size());
    for (String exclusion : exclusions) {
        if (ClassUtils.isPresent(exclusion, getClass().getClassLoader()) && !configurations.contains(exclusion)) {
            invalidExcludes.add(exclusion);
        }
    }
    if (!invalidExcludes.isEmpty()) {
        handleInvalidExcludes(invalidExcludes);
    }
}
```

#### X-1-4-0 去除不需要的配置类

通过类加载器, 加载 `spring.factories`文件中定义的 `org.springframework.boot.autoconfigure.AutoConfigurationImportFilter` 接口的具体实现, 进行不要的类过滤

```java
private ConfigurationClassFilter getConfigurationClassFilter() {
    if (this.configurationClassFilter == null) {
        // 过滤器.
        List<AutoConfigurationImportFilter> filters = getAutoConfigurationImportFilters();
        for (AutoConfigurationImportFilter filter : filters) {
            invokeAwareMethods(filter);
        }
        this.configurationClassFilter = new ConfigurationClassFilter(this.beanClassLoader, filters);
    }
    return this.configurationClassFilter;
}
```

#### X-1-5-0 发布事件

完成后, 发布 `AutoConfigurationImportEvent` 事件, 通知感兴趣的人..

```java
private void fireAutoConfigurationImportEvents(List<String> configurations, Set<String> exclusions) {
    List<AutoConfigurationImportListener> listeners = getAutoConfigurationImportListeners();
    if (!listeners.isEmpty()) {
        AutoConfigurationImportEvent event = new AutoConfigurationImportEvent(this, configurations, exclusions);
        for (AutoConfigurationImportListener listener : listeners) {
            invokeAwareMethods(listener);
            listener.onAutoConfigurationImportEvent(event);
        }
    }
}
```



## 日志管理

### 日志接口与实现

SpringBoot 推荐使用 `Slf4j` 与`Logback` 组合的方式, 进行日志管理

接口: `org.slf4j.Logger`
实现: `ch.qos.logback.classic.Logger`

### 日志配置

通过引入 `spring-boot-starter-logging` 启动器配置日志. 
自带引入三个依赖, 日志门面使用 `slf4j`

- `logback-classic`   具体日志实现
- `log4j-to-slf4j`  适配 `log4j`
- `jul-to-slf4j`  适配 `jul` (`java.util.Log`)



配置日志文件.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!--
  logback 配置文件.
    scan:当设置为true时配置文件若发生改变 将会重新加载;
    scanPeriod:扫描时间间隔, 若没给出时间单位默认为毫秒;
    debug:若设置 为true,将打印出logback内部日志信息
-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 上下文名称, 用来区分不同应用程序的记录默认为 default -->
    <contextName>Logger Demo</contextName>
    <!-- 属性配置 -->
    <property resource="application.properties"/>
    <!-- name:变量名称; value:变量值. 通过 :- 表示默认值 -->
    <!-- 文件路径 C:\Users\14345\springboot\application\logs -->
    <property name="LOG_PATH" value="${logging.path:-${user.home}/springboot/${spring.application.name:-application}/logs}"/>
    <!-- 文件名  C:\Users\14345\springboot\application\logs\application.log -->
    <property name="LOG_FILE" value="${logging.file:-${LOG_PATH}/application.log}"/>
    <!--
        logger{length}      输出日志的logger名,可有一个整形参数,可能能是缩短logger名
        contextName|cn      上下文名称
        date{pattern}       输出日志的打印时间,模式语法与java.text.SimpleDateFormat 兼容
        p/le/level          日志级别
        M/method            输出日志的方法名
        r/relative          从程序启动到创建日志记录的时间
        m/msg/message       程序提供的信息
        n                   换行符
     -->
    <!-- 格式化日志输出 -->
    <appender name="APPLICATION" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 输出文件名 -->
        <file>${LOG_FILE}</file>
        <!-- 格式 -->
        <encoder>
            <pattern>%date{HH:mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
        </encoder>
        <!-- 文件策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 文件名 -->
            <fileNamePattern>${LOG_FILE}.%d{yyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 最大文件保存天数 -->
            <maxHistory>7</maxHistory>
            <!-- 单个文件最大大小 -->
            <maxFileSize>50MB</maxFileSize>
            <!-- 总使用大小 -->
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
    </appender>
    <!-- 格式化日志输出, 标准输出到控制台 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%date{HH: mm:ss} %contextName [%t] %p %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- root: 全局,日志输出设置 -->
    <root level="info">
        <!-- 输出到: 控制台 -->
        <appender-ref ref="STDOUT"/>
        <!-- 输出到: 日志文件 -->
        <appender-ref ref="APPLICATION"/>
    </root>

    <!-- logger 具体包或子类输出设置 -->
    <logger name="top.jionjion.logging.service.SomeService" level="error" additivity="true">
        <appender-ref ref="STDOUT"/>
    </logger>
</configuration>
```



### 启动日志流程

#### X-1-0-0 入口方法

一般通过 `Logger logger = LoggerFactory.getLogger(getClass());` 获得当前类的日志管理

1. `LoggerFactory.getLogger` 入口
2. 调用 `findPossibleStaticLoggerBinderPathSet` 方法
3. 获取 `StaticLoggerBinder` 所在jar包路径
4. 若存在多个日志实现框架打印提示及选择
5. 使用 `StaticLoggerBinder` 获得日志工厂, 继而获得日志实现



#### X-1-1-0 获取日志

进行日志的初始化, 并返回日志对象. 

```java
public static Logger getLogger(Class<?> clazz) {
    // 进行日志寻址
    Logger logger = getLogger(clazz.getName());
    if (DETECT_LOGGER_NAME_MISMATCH) {
        Class<?> autoComputedCallingClass = Util.getCallingClass();
        if (autoComputedCallingClass != null && nonMatchingClasses(clazz, autoComputedCallingClass)) {
            Util.report(String.format("Detected logger name mismatch. Given name: \"%s\"; computed name: \"%s\".", logger.getName(),
                                      autoComputedCallingClass.getName()));
            Util.report("See " + LOGGER_NAME_MISMATCH_URL + " for an explanation");
        }
    }
    return logger;
}

// 进行日志寻址, 返回 ILoggerFactory 接口实现
public static Logger getLogger(String name) {
    // 获得日志工厂
    ILoggerFactory iLoggerFactory = getILoggerFactory();
    // 2. 获得日志实现类
    return iLoggerFactory.getLogger(name);
}
```

#### X-1-2-0 日志工厂初始化流程

日志工厂初始化流程. 通过状态进行区分, 执行. 

```java
// 获得日志工厂
public static ILoggerFactory getILoggerFactory() {
    // 如果当前状态为未初始化, 执行以下
    if (INITIALIZATION_STATE == UNINITIALIZED) {
        synchronized (LoggerFactory.class) {
            // 再次检查
            if (INITIALIZATION_STATE == UNINITIALIZED) {
                INITIALIZATION_STATE = ONGOING_INITIALIZATION;
                // 3. 准备初始化日志, 并确定具体实现
                performInitialization();
            }
        }
    }
    // 判断当前启动状态. 
    switch (INITIALIZATION_STATE) {
        // 如果启动成功
        case SUCCESSFUL_INITIALIZATION:
            // 获得实现类中的日志工厂单例对象
            return StaticLoggerBinder.getSingleton().getLoggerFactory();
        // ...
        case NOP_FALLBACK_INITIALIZATION:
            // 返回空对象, 默认空实现所有日志方法
            return NOP_FALLBACK_FACTORY;
        // ...
        case FAILED_INITIALIZATION:
            throw new IllegalStateException(UNSUCCESSFUL_INIT_MSG);
		// ...       
        case ONGOING_INITIALIZATION:
            return SUBST_FACTORY;
    }
    throw new IllegalStateException("Unreachable code");
}
```

#### X-1-3-0 日志工厂初始化

```java
private final static void performInitialization() {
    // - 日志初始化, 具体日志实现类的选择
    bind();
    // - 随后, 日志初始化状态为成功状态.
    if (INITIALIZATION_STATE == SUCCESSFUL_INITIALIZATION) {
        // 版本检查. 要求当日志实现版本必须在日志接口约定的版本中
        versionSanityCheck();
    }
}
```

#### X-1-3-1日志具体实现初始化

日志初始化. 在这里将具体实现类加载到系统... 

如果存在多个日志实现, 则由程序判断后引入并报告...

```java
private final static void bind() {
    try {
        Set<URL> staticLoggerBinderPathSet = null;
		// 非安卓环境, 执行
        if (!isAndroid()) {
            // 2. 加载, 日志的具体实现类, 并返回具体类的地址.
            staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
            // 3. 是否存在多个日志实现类, 如果存在则控制台打印报错.
            reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
        }
        // 具体的日志实现类中调用.
        StaticLoggerBinder.getSingleton();
        // 当前初始化状态改为成功
        INITIALIZATION_STATE = SUCCESSFUL_INITIALIZATION;
        // 4. 报告最终绑定状态. 如果有多个实现, 告知最终选中的日志实现
        reportActualBinding(staticLoggerBinderPathSet);
    } catch (NoClassDefFoundError ncde) {
        // 如果日志加载失败. 打印报错.
        String msg = ncde.getMessage();
        if (messageContainsOrgSlf4jImplStaticLoggerBinder(msg)) {
            INITIALIZATION_STATE = NOP_FALLBACK_INITIALIZATION;
            Util.report("Failed to load class \"org.slf4j.impl.StaticLoggerBinder\".");
            Util.report("Defaulting to no-operation (NOP) logger implementation");
            Util.report("See " + NO_STATICLOGGERBINDER_URL + " for further details.");
        } else {
            failedBinding(ncde);
            throw ncde;
        }
    } catch (java.lang.NoSuchMethodError nsme) {
        String msg = nsme.getMessage();
        if (msg != null && msg.contains("org.slf4j.impl.StaticLoggerBinder.getSingleton()")) {
            INITIALIZATION_STATE = FAILED_INITIALIZATION;
            Util.report("slf4j-api 1.6.x (or later) is incompatible with this binding.");
            Util.report("Your binding is version 1.5.5 or earlier.");
            Util.report("Upgrade your binding to version 1.6.x.");
        }
        throw nsme;
    } catch (Exception e) {
        failedBinding(e);
        throw new IllegalStateException("Unexpected initialization failure", e);
    } finally {
        // 5.清理.
        postBindCleanUp();
    }
}
```

#### X-1-3-2 日志实现类加载

主要是加载 `org/slf4j/impl/StaticLoggerBinder.class` 类, 约定日志接口的实现类的类名, 尝试加载具体的实现. 这里为 `Slf4j` 日志实现.

```java
static Set<URL> findPossibleStaticLoggerBinderPathSet() {
   	// 日志实现类的具体地址
    Set<URL> staticLoggerBinderPathSet = new LinkedHashSet<URL>();
    try {
        // 获得类加载器 ClassLoaders$AppClassLoader
        ClassLoader loggerFactoryClassLoader = LoggerFactory.class.getClassLoader();
        Enumeration<URL> paths;        
        if (loggerFactoryClassLoader == null) {        
            // 加载 org/slf4j/impl/StaticLoggerBinder.class 类,并使用其类加载器
            paths = ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH);
        } else {
            // 加载 org/slf4j/impl/StaticLoggerBinder.class 类
            paths = loggerFactoryClassLoader.getResources(STATIC_LOGGER_BINDER_PATH);
        }
        // 如果找到具体的日志实现类, 加载到集合中
        while (paths.hasMoreElements()) {
            URL path = paths.nextElement();
            staticLoggerBinderPathSet.add(path);
        }
    } catch (IOException ioe) {
        Util.report("Error getting resources from path", ioe);
    }
    // 返回日志的实现类地址结合
    return staticLoggerBinderPathSet;
}
```

#### X-1-3-3 多日志实现判断

```java
// 判断是否存在多个日志实现
private static boolean isAmbiguousStaticLoggerBinderPathSet(Set<URL> binderPathSet) {
    return binderPathSet.size() > 1;
}

private static void reportMultipleBindingAmbiguity(Set<URL> binderPathSet) {
    // 存在多个日志实现
    if (isAmbiguousStaticLoggerBinderPathSet(binderPathSet)) {
        // Util.report 实际是调用 System.err.println ..打印标准输出
        Util.report("Class path contains multiple SLF4J bindings.");
        for (URL path : binderPathSet) {
            Util.report("Found binding in [" + path + "]");
        }
        Util.report("See " + MULTIPLE_BINDINGS_URL + " for an explanation.");
    }
}
```

#### X-1-3-4 报告最终绑定状态

```java
private static void reportActualBinding(Set<URL> binderPathSet) {
    // 如果日志的实现不为空, 且存在多个日志框架实现.
    if (binderPathSet != null && isAmbiguousStaticLoggerBinderPathSet(binderPathSet)) {
        // 报告最终选择的日志实现.
        Util.report("Actual binding is of type [" + StaticLoggerBinder.getSingleton().getLoggerFactoryClassStr() + "]");
    }
}
```

#### X-2-1-0 日志对象 Logback

在初始后, 选择具体的日志

```java
public static Logger getLogger(String name) {
    // 获得实现类的日志工厂. 
    ILoggerFactory iLoggerFactory = getILoggerFactory();
    // 获得日志对象. 为具体子类实现... 如 Logback
    return iLoggerFactory.getLogger(name);
}
```

