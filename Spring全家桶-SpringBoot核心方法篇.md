---
title: Spring全家桶-SpringBoot核心方法篇
abbrlink: 27a11828
date: 2020-11-29 13:30:04
categories:
  - Java
  - Spring
tags:
  - Java
  - Spring
---

# 摘要

介绍 `Spring` 常用功能

<!--more-->

# 接口目录

## `Bean` 核心类及实现

1. `org.springframework.beans.factory.config.BeanDefinition` 接口

 在 `Spring` 提供的 `Bean` 定义类. 描述容器中 `Bean` 的基本信息

2. `org.springframework.beans.factory.support.RootBeanDefinition` 类

    `BeanDefinition` 接口的默认实现, 描述容器中 `Bean` 的基本信息




## `Bean` 扩展接口

1. `org.springframework.context.annotation.ImportBeanDefinitionRegistrar` 接口

   提供  `registerBeanDefinitions`  方法, 在 `@Import` 注解中注入改接口具体实现, 从而向容器注册 `BeanDefinition` , 即 `Bean` 定义.

2. `org.springframework.context.annotation.BeanDefinitionRegistryPostProcessor` 接口

   提供 `postProcessBeanDefinitionRegistry` 方法, 在 `Bean` 注册到容器,但尚未实例化前, 进行后置处理, 或者添加新的定义.

3. `org.springframework.beans.factory.FactoryBean<T>` 接口

   向容器中注入一个` FactoryBean` 用来实例化具体产品 `Bean`

4. `org.springframework.beans.factory.config.BeanFactoryPostProcessor` 接口

   在 `Bean` **注册后**的后置处理器, 提供 `postProcessBeanFactory` 方法, 对注入容器的 `BeanDefinition` 进行修改, 进而修改最终的 `Bean` 实例

5. `org.springframework.beans.factory.config.BeanPostProcessor` 接口

   在 `Bean`  **实例化**的前后进行调用, 自定义实例化细节或者动态修改实例对象信息.

6. `org.springframework.context.annotation.ImportSelector` 接口

   动态添加需要导入到容器的类, 包.
   
7. `org.springframework.beans.factory.InitializingBean` 接口

   在 `Bean` 初始化后,执行相关实现

8. `org.springframework.beans.factory.DisposableBean` 接口

   在 `Bean` 销毁前, 执行相关实现



## `Aware` 接口

1. `org.springframework.beans.factory.Aware` 

   空接口

2. `org.springframework.context.EnvironmentAware` 

   获取环境信息

3. `org.springframework.context.ApplicationContextAware`

   获取容器对象

1. `org.springframework.context.EmbeddedValueResolverAware` 

   获取变量解析器

2. `org.springframework.context.ResourceLoaderAware`

   获取资源加载器

3. `org.springframework.context.ApplicationEventPublisherAware`

   获取 `Spring` 容器事件发布器

4. `org.springframework.context.MessageSourceAware`

   获取消息管理器

5. 



## `Web` 容器扩展接口

1. `javax.servlet.ServletContainerInitializer`  接口, `Servlet` 提供

   在容器初始化阶段进行 `Web.xml` 的配置.

2. `org.springframework.boot.web.servlet.ServletContextInitializer` 接口, `Spring` 提供

   在 `WEB` 容器启动时注入什么, 比如 `Servlet`, `Filter`, `Listener` .



# 常用注解

## 容器定义

| 注解                     | 位置                                            | 说明                                                         | 更多                                      |
| ------------------------ | ----------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| `@SpringBootApplication` | 在主启动类上标注                                | 容器启动入口并向下扫描注入                                   |                                           |
| `@Component`             | 类                                              | 将其类托管给容器                                             | `@Repository` , `@Service` ,`@Controller` |
| `@Import`                | 配置类                                          | 将指定类注入容器                                             |                                           |
| `@ImportResource`        | 配置类                                          | `Spring` 配置文件注入                                        |                                           |
| `@PropertySource`        | 配置类                                          | 属性文件注入                                                 | `@PropertySources`                        |
| `@Configuration`         | 配置类,  其他自动配置注解                       | 配置类, 其下返回 `Bean` 对象托管给容器                       |                                           |
| `@Configurable`          | 类                                              | 当前类中的某些成员需要通过 `@Autowired ` 从容器获得, 但是当前类又想被手工 `new` 出. |                                           |
| `@Bean`                  | 方法, 注解类                                    | 方法的返回对象交给容器托管                                   |                                           |
| `@Autowired`             | 配置类,  `Bean`  方法, 构造器, 注解, 属性, 入参 | 将当前依赖的对象注入到 `Bean` 中                             |                                           |
| `@Order`                 | 配置类,   `Bean`  方法                          | 自定义排序的优先级                                           |                                           |
| `@ComponentScan`         | 配置类                                          | 自定义包扫描的范围                                           | `@ComponentScans`                         |
| `@Conditional`           | 配置类,  `Bean`  方法                           | 在满足条件下注入                                             | `@ConditionalOn**`                        |
| `@DependsOn`             | 配置类,  `Bean`  方法                           | 当前 `Bean` 的依赖对象                                       |                                           |
| `@Description`           | 配置类,  `Bean`  方法                           | 添加一段对当前 `Bean` 对象的描述                             |                                           |
| `@Lazy`                  | 配置类,  `Bean`  方法, 构造器, 属性, 入参       | 对象被懒实例化                                               |                                           |
| `@Primary`               | 配置类,  `Bean`  方法                           | 多态情况下, 标识为主要的 `Bean`                              |                                           |
| `@Qualifier`             | 配置类,  `Bean`  方法, 属性, 入参               | 多态情况下, 筛选需要的 `Bean`                                |                                           |
| `@Profile`               | 配置类,  `Bean`  方法                           | 多环境下, 当前注入的前置环境.                                |                                           |
| `@Scope`                 | 配置类,  `Bean`  方法                           | 当前 `Bean` 的实例化范围                                     |                                           |
| `@Lookup`                | 方法                                            | 重定义当前类的实例化方法, 返回原型类                         |                                           |
| `@Value`                 | 方法, 属性, 入参                                | 为当前对象注入环境属性值                                     |                                           |

## 异步调度

| 注解         | 位置       | 说明                         | 更多         |
| ------------ | ---------- | ---------------------------- | ------------ |
| `@Async`     | 类, 方法   | 对其进行异步支持             |              |
| `@Scheduled` | 方法, 注解 | 定义定时调度任务的日期表达式 | `@Schedules` |
|              |            |                              |              |




##  扩展启用

| 注解                      | 说明                 |
| ------------------------- | -------------------- |
| `@EnableAspectJAutoProxy` | 启用 `AOP`  动态代理 |
| `@EnableAsync`            | 启用异步执行支持     |
| `@EnableScheduling`       | 启用定时调度支持     |



## 帮助注解

| 注解              | 位置                 | 说明                                         | 更多 |
| ----------------- | -------------------- | -------------------------------------------- | ---- |
| `@AliasFor`       | 注解, 属性           | 表示当前注解与指定注解作用相同               |      |
| `@NonNull`        | 属性, 方法, 入参     | 属性,参数或返回值不为空                      |      |
| `@Nullable`       | 属性, 方法, 入参     | 属性,参数或返回值可能为空                    |      |
| `@NonNullFields`  | 属性                 | 属性不为空                                   |      |
| `@NonNullApi`     | 方法, 入参           | 参数或返回值不为空                           |      |
| `@DateTimeFormat` | 类, 属性, 方法, 入参 | 指定日期类的格式                             |      |
| `@NumberFormat`   | 类, 属性, 方法, 入参 | 指定数字的格式                               |      |
| `@Indexed`        | 类                   | 添加索引文件, 在项目打包后执行, 加速注入执行 |      |
| `@Validated`      | 类, 方法, 入参       | 对数据进行校验                               |      |



# 方法入口

## 容器生命周期





## `MVC` 请求

