---
title: Spring全家桶-单元测试篇
abbrlink: 9d7aa68f
date: 2019-04-23 10:27:11
categories:
  - Java
  - Spring
tags: [Java, Spring]
---

> Spring 单元测试的使用

<!--more-->



# 简介

介绍了`spring-test`中单元测试的使用

渣包依赖
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

## 主要注解
| 注解 | 属性 | 解释 |
| --- | --- | --- |
| `@RunWith` | `value` | 测试的运行环境,一般写作 `@RunWith(SpringRunner.class)` |
| `@SpringBootTest` | 	| 标识一个Spring的测试,使用测试上下文容器 |
| `@SpringBootApplication` | `scanBasePackages` | 将测试类单独作为容器启动类,使用上下文容器.指定扫描的包 |
| `@Test`| 		| 测试方法	|
| `@Transactional ` |  | 标识该类或者该方法 |
| `@Rollback` 	|  	 | 事物回滚 |
| `@Commit` 	| 		| 事物提交 |
| `@AutoConfigureMockMvc` |    | 自动注入 `MockMvc` 类进行模拟数据请求 |


##  测试示例

### 普通的单元测试

使用测试上下文容器的测试用例

``` java
@RunWith(SpringRunner.class) // 测试环境
@SpringBootTest // 测试
//@Transactional // 开启事物,可以在测试方法中,使用@Rollback回滚
@Slf4j
public class UserTest {
    @Test
    public void testToString(){
        User user = new User();
        user.setUsername("Jion");
        user.setPassword("12345");
        user.setToken("123456789abcedf");
        log.info("用户 >>" + user.toString());
    }
}
```

使用启动类上下文容器的测试用例

```java
@RunWith(SpringRunner.class)
@SpringBootApplication(scanBasePackages = "top.jionjion.bean")
public class AppConfigTest {

    @Autowired
    private AppConfig appConfig;

    @Test
    public void testGetUser(){
        User user = appConfig.getUser();
        System.out.println(user.hashCode());
    }
}
```

最新版测试. 只需使用 `@SpringBootTest` 声明一个 `package-private` 的测试类,即可完成单元测试

```java
@SpringBootTest
class StudentMapperTest {

    @Autowired
    StudentMapper studentMapper;

    @Test
    void findByUserId(){
        Student student = studentMapper.findByUserId(1);
        Assert.notNull(student,"查询失败!");
    }
}
```



### Http请求的单元测试

带有JSON参数的POST请求单元测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
@Slf4j
public class UserControllerTest {

    // 模拟数据
    @Autowired
    private MockMvc mockMVC;

    /** 测试查询方法 */
    @Test
    public void testGetUser() throws Exception {
        // JSON对象
        JSONObject json = new JSONObject();
        json.put("username", "JionJion");
        json.put("password", "123456");

        // 发送请求
        MvcResult result = mockMVC.perform(
                MockMvcRequestBuilders
                        .post("/auth/user/info")  // 访问路径
                        .contentType(MediaType.APPLICATION_JSON_UTF8) // json格式数据
                        .content(json.toString())
        )
                .andExpect(status().isOk())	// 请求成功
                .andDo(MockMvcResultHandlers.print())  //打印结果,太长的返回结果不作打印
                .andReturn(); // 返回结果

        // 响应
        MockHttpServletResponse response = result.getResponse();
        String content = response.getContentAsString();
        log.info("登录返回:" + content);
        assertNotNull(content);

        // 请求
        MockHttpServletRequest request = result.getRequest();
        String requestURI = request.getRequestURI();
        log.info("请求地址:" + requestURI);
    }
}
```

