---
title: Spring全家桶-属性文件篇
abbrlink: 5b93ef85
date: 2019-04-21 10:14:39
categories:
  - Java
  - Spring
tags: [Java, Spring]
---

# 介绍
介绍属性文件的读取


## 通过注解方式读取配置信息
### 新建属性文件
在`src\main\resources\properties`目录下新建属性文件`description.properties`.作为系统描述

**属性,数组,列表,集合**
``` properties
#info=信息
info.version = 1.0
info.author = Jion

info.cn_numbers = 〇,一,二,三,四,五,六,七,八,九,十,百,千,万,亿

info.address_list[0] = 127.0.0.1
info.address_list[1] = localhost


eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 使用Bean读取配置类

引入依赖`spring-boot-configuration-processor`,配置文件处理器
官网[参考](https://docs.spring.io/spring-boot/docs/2.1.4.RELEASE/reference/html/configuration-metadata.html#configuration-metadata-annotation-processor)
``` xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
</dependency>
```

`@Component`标识这是一个组件,交由Spring管理
`@PropertySource`指向属性文件的位置,而不是从全局配置文件或者运行环境中获得.
`@ConfigurationProperties`标识属性文件的前缀,该注解默认从全局配置文件或者运行环境中获得.
`@Getter`,`@Setter`,`@ToString`注解由Lombok提供,标示重写`get/set`方法和`toString`方法
**属性不能为static修饰,并不用强制配置Getter/Setter方法**
```java
@Component
@PropertySource("classpath:config/description.properties")
@ConfigurationProperties(prefix="info", ignoreUnknownFields=true)
@Getter
@Setter
@ToString
public class Info {
    private String version;

    private String author;

    @Value("#{'${info.cn_numbers}'.split(',')}")
    private String[] cnNumbers;

    private List<String> addressList;

    private Map<String, String> map;
}
```

### 使用`@Value`读取
**这种方式配置文件需要直接放在resource目录下**
`@Component`标识这是一个组件,交由Spring管理
`@Value`使用`${}`表达式赋值.
```
@Component
@Getter
@Setter
@ToString
public class Info {

    @Value("${info.version}")
    private String version;

    @Value("${info.author}")
    private String author;

    @Value("#{'${info.cn_numbers}'.split(',')}")
    private String[] cnNumbers;

    @Value("${info.address_list}")
    private List<String> addressList;

    @Value("${info.map}")
    private Map<String, String> map;
}
```

### 测试
测试,配置类文件只能通过Spring容器,获取.而不是new的方式
``` java
@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class InfoTest {

    @Autowired
    private Info info;

    /** 测试属性文件读取 */
    @Test
    public void test(){
        log.info("信息类" + info);
        log.info("数组" + Arrays.toString(info.getCnNumbers()));
        log.info("列表" + info.getAddressList());
        log.info("键值对" + info.getMap());
    }
}
```

### `@ConfigurationProperties`和`@Value`的区别
| 项目                       | `@ConfigurationProperties` | `@Value`         |
| -------------------------- | -------------------------- | ---------------- |
| 绑定                       | 通过Bean批量绑定           | 一个个指定绑定   |
| 松散语法(多种命名格式支持) | 支持多种命名格式           |                  |
| `SpEL`表达式               |                            | 支持`SpEL`表达式 |
| `JSR303`数据校验           | 支持数据校验               |                  |

### 随机数与占位符
可以通过以下方式,获得一个随机数
`${random.Value}`
`${random.int}`
`${random.int(min, max)}`
`${random.long}`
`${random.long(min, max)}`
`${random.uuid}`
通过,`${}`获得之前定义的`name`属性,如果不存在,则为`default`默认值.
`${name:default}`
