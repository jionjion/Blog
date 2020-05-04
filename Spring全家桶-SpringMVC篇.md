---
title: Spring全家桶-SpringMVC篇
abbrlink: 3796c217
date: 2019-04-17 09:12:04
categories:
  - Java
  - Spring
tags: [Java, Spring, MVC]
---

## 常用注解
`@Autowired`        对私有属性实现依赖注入
`@RestController`   标识这是一个Restful的控制器,返回JSON结果
`@RequestMapping`   匹配请求路径,对所有的请求方式进行响应
`@PostMapping`      仅对Post请求进行响应
`@RequestParam`     标识这是一个请求参数
`@RequestBody`      标识这是一个JSON格式的请求,并将JSON对象转为Java对象
`@JsonProperty`     对响应返回的JSON的属性进行修改
`@JsonDeserialize`  反序列化时指定格式.如日期的格式化`@JsonDeserialize(using = CustomJsonDateDeserializer.class)`
`@JsonFormat`       JSON输出的格式化
`@JsonSerialize`    对响应的JSON嵌入自定义代码
`@jsonignore`       表示不会对该类的属性进行JSON输出
`@JsonIgnoreProperties` 类注解,将某些属性不进行JSON输出

## 常见配置

### 允许跨域

重写配置类

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * @author Jion
 */
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


/**
 * @author Jion
 */
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

