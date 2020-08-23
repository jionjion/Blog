---
title: Spring全家桶-SpringMVC篇
abbrlink: 3796c217
date: 2019-04-17 09:12:04
categories:
  - Java
  - Spring
tags: [Java, Spring, MVC]
---

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

### 允许跨域

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

### 外部资源路径

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

## 反序列化

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

