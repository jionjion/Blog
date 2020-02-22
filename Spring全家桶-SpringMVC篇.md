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
