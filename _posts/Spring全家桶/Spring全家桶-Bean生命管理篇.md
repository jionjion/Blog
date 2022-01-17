---
title: Spring全家桶-Bean生命管理篇
abbrlink: 7f310c97
date: 2020-08-11 21:01:46
categories:
  - Java
  - Spring
tags: [Java, Transaction]
---

> Bean 的生命周期管理

<!--more-->



## Bean生命周期处理

### `@Bean` 注解

通过 `@Bean` 配合 `@Configuration` 注解对其进行声明, 在其中 `initMethod` 和 `destroyMethod` 分别指定其初始化和销毁方法

```java
/** 配置类 */
@Configuration
public class AppConfig {

    @Bean(name = {"user","USER"}, initMethod = "init", destroyMethod = "destroy")
    public User getUser(){
        return new User();
    }
}

/** Bean对象 */
public class User {
    public void init(){ log.info("初始化数据...."); }

    public void destroy(){ log.info("准备销毁数据...."); }    
}
```



###  `@PostConstruct` 和 `@PreDestroy` 注解

通过使用 `javax.annotation.PostConstruct` 和 `javax.annotation.PreDestroy` 注解, 分别在 `Bean` 初始化完成,或者销毁时执行.

```java
@Component
public class User {
    @PostConstruct
    public void init(){
        log.info("初始化数据....");
    }

    @PreDestroy
    public void destroy(){
        log.info("准备销毁数据....");
    }    
}
```



### `InitializingBean` 和 `DisposableBean` 接口

 `org.springframework.beans.factory.InitializingBean` 和 `org.springframework.beans.factory.DisposableBean` 接口, 分别在 `Bean` 初始化后和销毁前执行相关回调



### `BeanFactoryPostProcessor` 接口

通过 `org.springframework.beans.factory.config.BeanFactoryPostProcessor` .配置 `BeanFactory` 的后置处理器, 在类注册到容器后进行相关操作.

```java
/** Bean 工厂, 对已经初始化后的Bean进行一些修改,实例化时调用 BeanFactoryPostProcessor 接口 */
@Component
public class PeopleBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        // 获得 Bean的定义
        BeanDefinition people = beanFactory.getBeanDefinition("user");
        // 获得属性,并修改
        MutablePropertyValues propertyValues = people.getPropertyValues();
        propertyValues.addPropertyValue("name","Jion");
    }
}

```



### `BeanPostProcessor` 接口

通过 `org.springframework.beans.factory.config.BeanPostProcessor` 及其子接口, 在类实例化前后进行处理.

**相关子接口**

`org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor` 扩展,添加 `Bean` 初始化前的回调方法; 初始化后但未装配对象属性前的回调方法

`org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor`  扩展,添加 `Bean` 的销毁前执行方法.

```java
/** 在类实例化时,调用 BeanPostProcessor 接口 */
@Component
public class PeopleBeanPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        // 在类初始化前调用, 若有返回值,则将该返回值作为实例化结果
        if ("people".equals(beanName)){
            return new People();
        }
        // 返回为null, 继续初始化
        return null;
    }

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



