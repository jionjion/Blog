---
title: Spring全家桶-SpringMVC篇
abbrlink: 3796c217
date: 2019-04-17 09:12:04
categories:
  - Java
  - Spring
tags: [Java, Spring, MVC]
---

> SpringMVC的常用处理

<!--more-->



# 常用注解

| 注解              | 用途                                                | 说明 |
| ----------------- | --------------------------------------------------- | ---- |
| `@Autowired`      | 对私有属性实现依赖注入                              |      |
| `@RestController` | 标识这是一个Restful的控制器,返回JSON结果            |      |
| `@RequestMapping` | 匹配请求路径,对所有的请求方式进行响应               |      |
| `@PostMapping`    | 仅对Post请求进行响应                                |      |
| `@RequestParam`   | 标识这是一个请求参数                                |      |
| `@RequestBody`    | 标识这是一个JSON格式的请求,并将JSON对象转为Java对象 |      |

# 常见配置

## 允许跨域

重写配置类

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    /***
     *  跨域请求,允许
     * @param registry 注册
     */
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("*")
                .allowedMethods("POST", "GET", "PUT", "OPTIONS", "DELETE")
                .maxAge(3600)
                .allowCredentials(true);
    }
}
```

## 外部资源路径

允许将`URL `中的访问路径映射到文件系统中,扩展访问范围. 
**注意** 映射磁盘路径以文件夹结尾符结束 `\`

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    /***
     *  添加资源文件夹位置
     * @param registry 注册
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //如果是Windows环境的话 file:=改为=》file:///
        registry.addResourceHandler("/static/**").addResourceLocations("file:" + "S://" + File.separatorChar);
    }
}
```



# 参数绑定

## InitBinder 反序列化

对于 `URL` 中的请求参数, `Form` 表单中的请求参数. 

- 可以通过 `org.springframework.beans.propertyeditors.*` 自定义实现转换
- 通过注入 `java.beans.PropertyEditorSupport` 自定义转换实现

```java
    /**
     * 约定数据转换格式.有且仅针对与URL传参和Form表单传参形式.
     * <p>
     * 对于请求体内参数格式转换, 交给Json序列化与反序列扩展.
     * 参阅 {@code com.hand.hap.core.json}
     *
     * @param binder  注册数据编辑器
     * @param request 请求
     */
    @InitBinder
    public void initBinder(WebDataBinder binder, WebRequest request) {
        // 更多自定义编辑器参阅 org.springframework.beans.propertyeditors.*
        binder.registerCustomEditor(Date.class, new CustomDateEditor(new SimpleDateFormat(PATTERN), true));

        // LocalTime LocalDate, LocalDateTime 类型转换.. 需要重载 registerCustomEditor
        binder.registerCustomEditor(LocalTime.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) {
                if (!StringUtils.isEmpty(text)) {
                    setValue(LocalTime.parse(text, DateTimeFormatter.ofPattern("HH:mm:ss")));
                }
            }
        });
        binder.registerCustomEditor(LocalDate.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) {
                if (!StringUtils.isEmpty(text)) {
                    setValue(LocalDate.parse(text, DateTimeFormatter.ofPattern("yyyy-MM-dd")));
                }
            }
        });
        binder.registerCustomEditor(LocalDateTime.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) {
                if (!StringUtils.isEmpty(text)) {
                    setValue(LocalDateTime.parse(text, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
                }
            }
        });
    }
```





## 注解反序列化

带日期格式的参数转为 `java` 类

通过 `@DateTimeFormat(pattern = "yyyy-MM-dd")` 注解将字符串属性转为对应的日期属性. 仅支持 `java.util.Date` 和 `java.sql.Time` 类



## 对象序列化

参考 [JSON 常用注解](https://blog.jionjion.top/posts/316f1148/)



##请求参数绑定

- `URL` 请求参数
- `Form` 表单参数,封装为 `Bean` 对象
- 封装 `Bean` 对象
- `URL` 路径参数
- `Form` 表单参数,请求参数
- 请求体中 `JSON` 对象封装为 `Bean` 对象
- 请求体中, `Array` 对象转为 `List` 对象 

```java
@Slf4j
@RestController
public class ParameterController {


    /**
     * 直接在URL后跟参数,或通过Params进行查询
     *
     * @param name 用户名
     * @return 响应字符串
     */
    @GetMapping("/parameter/a")
    public String addParameterA(String name) {
        log.info("name is: " + name);
        return "Success";
    }

    /**
     * 通过HttpServletRequest接收
     * 通过在URL中追加参数, 或 Form表单中增加参数属性
     *
     * @param request 请求
     * @return 响应字符串
     */
    @PostMapping("/parameter/b")
    public String addParameterB(HttpServletRequest request) {
        // 获得表单或者查询参数
        String name = request.getParameter("name");
        log.info("name is: " + name);
        return "Success";
    }

    /**
     * 封装一个bean来接收
     * 通过在URL中追加参数, 或 Form表单中增加封装属性
     *
     * @param student 传入Bean参数对象
     * @return 响应字符串
     */
    @PostMapping("/parameter/c")
    public String addParameterC(Student student) {
        log.info("student is: " + student);
        return "Success";
    }

    /**
     * 通过@PathVariable获取路径中的参数
     *
     * @return 响应字符串
     */
    @RequestMapping(value = "/parameter/d/{id}")
    public String addParameterD(@PathVariable String id) {
        log.info("id is: " + id);
        return "Success";
    }


    /**
     * 使用@ModelAttribute注解获取POST请求的FORM表单数据, 其实不使用注解也可
     *
     * @param student 封装Form表单
     * @return 响应字符串
     */
    @RequestMapping(value = "/parameter/e", method = RequestMethod.POST)
    public String addParameterE(@ModelAttribute("student") Student student) {
        log.info("student is: " + student);
        return "Success";
    }

    /**
     * 用注解@RequestParam绑定请求URL中的参数,Form表单中的参数
     *
     * @param name 请求参数
     * @return 响应字符串
     */
    @RequestMapping(value = "/parameter/f")
    public String addParameterF(@RequestParam("name") String name) {
        log.info("name is: " + name);
        return "Success";
    }

    /**
     * 用注解@Requestbody绑定请求参数到方法入参
     *
     * @param student 请求体
     * @return 响应字符串
     */
    @RequestMapping(value = "/parameter/g", method = RequestMethod.POST)
    public String addParameterG(@RequestBody Student student) {
        log.info("student is: " + student);
        return "Success";
    }

    /**
     * 用注解@Requestbody绑定请求参数到方法入参
     * 通过在请求体中,增加数组,完成绑定
     *
     * @param names 请求体,数组对象
     * @return 响应字符串
     */
    @RequestMapping(value = "/parameter/h", method = RequestMethod.POST)
    public String addParameterH(@RequestBody String[] names) {
        log.info("names is: " + Arrays.toString(names));
        return "Success";
    }
}
```



# 切面增强

## 统一异常处理

可以通过单独定义类配合 `@ControllerAdvice` 注解对其加强.

`@ExceptionHandler(value = Exception.class)` 标识其为一个异常处理类

```java
/**
 * @author Jion
 * 全局异常通知
 */
@ControllerAdvice
public class UserExceptionHandler {

    /**
     * 处理异常的类,这里将异常统一捕获,完成分类处理
     */
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public ResultMessage<Object> handle(Exception exception) {
        // 如果属于自定义的异常
        if (exception instanceof UserException) {
            // 强制类型转换
            UserException e = (UserException) exception;

            // 将抛出的异常捕获后包装
            return new ResultMessage<>(e.getCode(), e.getMessage());
        }

        //如果不是自动返回系统的异常
        return new ResultMessage<>(550, exception.getMessage());
    }
}
```



## `Request` 加强

在所有的请求进入前拦截, 在从 `request` 中反序列化为 `java` 参数前后, 进行增强.

- `@ControllerAdvice` 标识其为控制层增强
- 继承自 `RequestBodyAdviceAdapter` 类, 从而实现 `RequestBodyAdvice` 接口

```java
/**
 * 对请求进行加强,仅有在Post/Put请求中有效
 * 默认只是对当前同包和子包进行处理
 *
 * @author Jion
 */
@ControllerAdvice
public class UserControllerRequestAdvice extends RequestBodyAdviceAdapter {

    final Logger logger = Logger.getLogger(UserControllerRequestAdvice.class.getName());

    /**
     * 该方法用于判断当前请求, 是否要执行 beforeBodyRead 方法和 afterBodyRead 或 handleEmptyBody , 会在他们两个之前各执行一次
     *
     * @param methodParameter 请求调用方法的参数对象
     * @param targetType      请求调用方法的参数类型
     * @param converterType   将会使用到的Http消息转换器类类型
     * @return 返回true 则会执行beforeBodyRead
     */
    @Override
    public boolean supports(@NonNull MethodParameter methodParameter, @NonNull Type targetType, @NonNull Class<? extends HttpMessageConverter<?>> converterType) {
        logger.info(() -> "被拦截的方法: " + methodParameter.getMethod());
        logger.info(() -> "输出响应的Java类型为: " + targetType.getTypeName());
        logger.info(() -> "使用的响应转换器为: " + converterType.getName());
        return true;
    }


    /**
     * 在Http消息转换器执转换, 之前执行
     *
     * @param inputMessage  客户端的请求数据
     * @param parameter     请求调用方法的参数对象
     * @param targetType    请求调用方法的参数类型
     * @param converterType 将会使用到的Http消息转换器类类型
     * @return 返回 一个自定义的HttpInputMessage
     */
    @Override
    @NonNull
    public HttpInputMessage beforeBodyRead(HttpInputMessage inputMessage, @NonNull MethodParameter parameter, @NonNull Type targetType, @NonNull Class<? extends HttpMessageConverter<?>> converterType) throws IOException {
        try (InputStream inputStream = inputMessage.getBody()) {
            logger.info(() -> "转化前, 当前输入内容: " + inputStream);
            logger.info(() -> "转化前, 当前参数类型: " + parameter.getParameterType());
            logger.info(() -> "转化前, 输出响应的Java类型为: " + targetType.getTypeName());
            logger.info(() -> "转化前, 使用的响应转换器为: " + converterType.getName());

            // 读取请求体
            byte[] body = inputMessage.getBody().readAllBytes();

            logger.info(() -> "读取到内容: " + new String(body));

            // 构造新的读取流, 替换原有请求流
            return new HttpInputMessage() {
                @Override
                @NonNull
                public HttpHeaders getHeaders() {
                    return inputMessage.getHeaders();
                }

                @Override
                @NonNull
                public InputStream getBody() {
                    return new ByteArrayInputStream(body);
                }
            };
        }
    }

    /**
     * 在Http消息转换器执转换, 之后执行.一般是转换后的Java类, 用来封装参数
     *
     * @param body          转换后的对象
     * @param inputMessage  客户端的请求数据
     * @param parameter     请求调用方法的参数类型
     * @param targetType    请求调用方法的参数类型
     * @param converterType 使用的Http消息转换器类类型
     * @return 返回一个新的对象
     */
    @Override
    public @NonNull Object afterBodyRead(@NonNull Object body, @NonNull HttpInputMessage inputMessage, @NonNull MethodParameter parameter, @NonNull Type targetType, @NonNull Class<? extends HttpMessageConverter<?>> converterType) {

        logger.info(() -> "转化后, 生成的封装类: " + body.getClass().getName());
        logger.info(() -> "转化后, 消息HttpInputMessage封装类: " + inputMessage.hashCode());
        logger.info(() -> "转化后, 当前参数类型: " + parameter.getParameterType());
        logger.info(() -> "转化后, 输出响应的Java类型为: " + targetType.getTypeName());
        logger.info(() -> "转化后, 使用的响应转换器为: " + converterType.getName());

        return super.afterBodyRead(body, inputMessage, parameter, targetType, converterType);
    }

    /**
     * 在Http消息转换器执转换, 之后执行,
     * 不过这个方法处理的是, body为空的情况
     *
     * @param body          转换后的对象
     * @param inputMessage  客户端的请求数据
     * @param parameter     请求调用方法的参数类型
     * @param targetType    请求调用方法的参数类型
     * @param converterType 使用的Http消息转换器类类型
     * @return 返回一个新的对象
     */
    @Override
    @Nullable
    public Object handleEmptyBody(@Nullable Object body, @NonNull HttpInputMessage inputMessage, @NonNull MethodParameter parameter, @NonNull Type targetType, @NonNull Class<? extends HttpMessageConverter<?>> converterType) {
        logger.info(() -> "转化后且为空, " + Objects.isNull(body));
        logger.info(() -> "转化后且为空, 内容" + body);
        logger.info(() -> "转化后且为空, 消息HttpInputMessage封装类: " + inputMessage.hashCode());
        logger.info(() -> "转化后且为空, 当前参数类型: " + parameter.getParameterType());
        logger.info(() -> "转化后且为空, 输出响应的Java类型为: " + targetType.getTypeName());
        logger.info(() -> "转化后且为空, 使用的响应转换器为: " + converterType.getName());

        logger.info(() -> "handleEmptyBody " + body);
        return super.handleEmptyBody(body, inputMessage, parameter, targetType, converterType);
    }
}
```

单元测试

```java
@SpringBootTest
class UserControllerRequestAdviceTest {

    @Autowired
    ObjectMapper objectMapper;

    @Autowired
    UserController userController;

    @Autowired
    UserExceptionHandler userExceptionHandler;

    @Autowired
    UserControllerRequestAdvice userControllerRequestAdvice;

    /**
     * 仅对含有 Request Body 的请求进行拦截
     *
     * @throws Exception 抛出异常
     */
    @Test
    void requestHandler() throws Exception {
        User user = new User();
        user.setId(1);
        user.setUsername("username");
        user.setPassword("password");
        user.setAddress("ShangHai");
        user.setBirthday(new Date());

        MockMvc mockMvc = MockMvcBuilders.standaloneSetup(userController).setControllerAdvice(userExceptionHandler).setControllerAdvice(userControllerRequestAdvice).build();
        // json 请求体
        mockMvc.perform(MockMvcRequestBuilders.post("/user/user").contentType(MediaType.APPLICATION_JSON).content(objectMapper.writeValueAsString(user)).accept(MediaType.APPLICATION_JSON)).andExpect(MockMvcResultMatchers.status().isOk()).andDo(MockMvcResultHandlers.print());
    }
}
```



## `Response` 加强

通过 `@ControllerAdvice` 配合 `ResponseBodyAdvice` 接口, 对响应内容进行拦截, 其中泛型 `<Object>` 为 `Controller` 中的返回值..

```java
@ControllerAdvice
public class UserControllerResponseAdvice implements ResponseBodyAdvice<Object> {

    final Logger logger = Logger.getLogger(UserControllerResponseAdvice.class.getName());

    /**
     * 是否执行响应拦截
     *
     * @param returnType    返回Java类的类型
     * @param converterType 响应内容Http消息转换器类类型
     * @return 是否执行拦截
     */
    @Override
    public boolean supports(@NonNull MethodParameter returnType, @NonNull Class<? extends HttpMessageConverter<?>> converterType) {
        logger.info(() -> "准备判断是否执行响应拦截");
        logger.info("返回类型 " + returnType.getParameterType());

        return true;
    }

    /**
     * @param body                  响应内容, 一般是业务类
     * @param returnType            Controller 中的方法返回类型
     * @param selectedContentType   响应 ContentType 类型
     * @param selectedConverterType 响应内容 Http消息转换器类类型
     * @param request               当前请求
     * @param response              当前响应
     * @return 响应输出对象
     */
    @Override
    public Object beforeBodyWrite(Object body, @NonNull MethodParameter returnType, @NonNull MediaType selectedContentType, @NonNull Class<? extends HttpMessageConverter<?>> selectedConverterType, @NonNull ServerHttpRequest request, @NonNull ServerHttpResponse response) {

        logger.info(() -> "响应转换前, 当前要转换的类" + body.getClass().getName());
        logger.info(() -> "响应转换前, 方法定义的返回类型" + returnType.getParameterType());
        logger.info(() -> "响应转换前, 使用的响应转换器为: " + selectedConverterType.getName());
        logger.info(() -> "响应转换前, 当前请求" + request);
        logger.info(() -> "响应转换前, 当前响应" + response);
        return body;
    }
}
```

测试类

```java
@SpringBootTest
class UserControllerResponseAdviceTest {

    @Autowired
    ObjectMapper objectMapper;

    @Autowired
    UserController userController;

    @Autowired
    UserExceptionHandler userExceptionHandler;

    @Autowired
    UserControllerResponseAdvice userControllerResponseAdvice;


    /**
     * 对所有响应拦截
     *
     * @throws Exception 抛出异常
     */
    @Test
    void responseHandler() throws Exception {
        MockMvc mockMvc = MockMvcBuilders.standaloneSetup(userController).setControllerAdvice(userExceptionHandler).setControllerAdvice(userControllerResponseAdvice).build();

        mockMvc.perform(MockMvcRequestBuilders.get("/user/users/1").accept(MediaType.APPLICATION_JSON)).andExpect(MockMvcResultMatchers.status().isOk()).andDo(MockMvcResultHandlers.print());
    }
}
```

