---
title: Spring全家桶-国际化篇
categories:
  - Java
  - Spring
tags:
  - Java
  - Spring
  - SpringBoot
abbrlink: 1a1582e
date: 2020-02-23 13:06:49
---

# 简介

## 属性文件

在 `resourcee/i18n` 目录下创建属性文件 `message.properties` , 作为默认的语言的消息内容.
同时创建 `message_zh_cn.properties` 和 `message_en_us.properties` 分别表示简体中文和美式英文的消息内容.

**示例**

```properties
# 分页组件
pagination-next=Next
pagination-prev=Prev
# 带占位符的提示
account-card-prompt=Welcome, {0}
```



## 项目国际化

### 配置默认语言

创建配置类, 并指定默认的语言版本和设置拦截器,为系统配置拦截语言规则

```java
@Configuration
public class LocaleLanguageConfig {

    /**
     *  默认解析器 其中locale表示默认语言
     */
    @Bean
    public LocaleResolver localeResolver() {
        SessionLocaleResolver localeResolver = new SessionLocaleResolver();
        // 设置默认语言环境,设置默认.不指定具体语言
        localeResolver.setDefaultLocale(Locale.getDefault());
        return localeResolver;
    }

    /**
     *  默认拦截器
     */
    @Bean
    public WebMvcConfigurer localeInterceptor() {
        return new WebMvcConfigurer() {
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                LocaleChangeInterceptor localeInterceptor = new LocaleChangeInterceptor();
                // lang表示切换语言的参数名,这里未启用
                // localeInterceptor.setParamName("lang");
                localeInterceptor.setIgnoreInvalidLocale(false);
                registry.addInterceptor(localeInterceptor);
            }
        };
    }
}
```

### 消息工具类

消息工具类,提供对外的消息提示.
通过 `org.springframework.context.MessageSource` 类的`getMessage` 方法,获得对应的消息提示

```java
import org.springframework.context.MessageSource;
import org.springframework.context.i18n.LocaleContextHolder;
import org.springframework.stereotype.Component;

@Component
public class MessageUtils {

    /** 消息管理器 */
    private static MessageSource messageSource;

    public MessageUtils(MessageSource messageSource) {
        MessageUtils.messageSource = messageSource;
    }

    /**
     *  获取单个国际化翻译值, 不存在则返回对应的key
     * @param msgKey i18n属性文件中的key
     * @param args 属性文件中,占位符
     * @return 对应语境下的消息
     */
    public static String get(String msgKey, Object[] args) {
        try {
            return messageSource.getMessage(msgKey, args, LocaleContextHolder.getLocale());
        } catch (Exception e) {
            return msgKey;
        }
    }

    /**
     *  获取单个国际化翻译值, 不存在则返回对应的key
     * @param msgKey i18n属性文件中的key
     * @return 对应语境下的消息
     */
    public static String get(String msgKey) {
        return get(msgKey, null);
    }
}
```



测试获得

```java
@Slf4j
public class MessageUtilsTest{
    @Test
    public void testGet(){
        // 获得消息内容
        String result = MessageUtils.get("page-manger-table-stop");
        log.info("消息内容: >>> "  + result );

        // 获得带占位符的消息内容
        String result2 = MessageUtils.get("account-card-prompt", new String[] {"囧囧"});
        log.info("消息内容: >>> "  + result2 );
    }
}
```



## 页面国际化

### Thymeleaf

在 `thymeleaf` 中,通过 `#{属性文件的键}` 获得变量

`th:text="#{account-card-prompt}"`