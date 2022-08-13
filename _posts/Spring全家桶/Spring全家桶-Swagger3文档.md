---
title: Spring全家桶-Swagger3文档
abbrlink: 6b0021f0
date: 2020-11-04 09:56:06
categories:
  - Java
  - Spring
tags: [Java, Spring, Swagger]
---

> 项目通过 SpringFox 引入 Swagger3 对进行接口管理

<!--more-->



# SpringFox

通过 `SpringFox` 将 `Swagger` 引入到项目中, 生成API文档. 目前已经支持 `Swagger3` 版本

## 快速启用

`springfox-boot-starter` 引入 `swagger3`

#### Pom文件

```xml
<!-- springfox-boot-starter 引入 swagger -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

#### 配置类

在该配置文件中, 继承自 `org.springframework.web.servlet.config.annotation.WebMvcConfigurer` 接口, 配置静态资源映射路径, 同时为了兼容 `Swagger2` 旧版路径.
`@EnableOpenApi` 开启 `Swagger3` 
`Docket` 为文档对象, 代表了生成的文档结构对象.

```java
import io.swagger.annotations.Api;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import springfox.documentation.builders.*;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.schema.ScalarType;
import springfox.documentation.service.*;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

@EnableOpenApi
@Configuration
public class Swagger3Config implements WebMvcConfigurer {

    @Bean
    public Docket createRestApi() {
        //返回文档摘要信息
        return new Docket(DocumentationType.OAS_30)
                // 文档开头描述
                .apiInfo(apiInfo())
                // 生成文档的入口
                .select()
                // 通过注解方式检索..指定方法注解, 类注解, 基础扫描包.. 最新的注解版本包 io.swagger.core.v3.*
//                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class)) // 2.0
//                .apis(RequestHandlerSelectors.withMethodAnnotation(Operation.class)) // 3.0
                .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
                .apis(RequestHandlerSelectors.basePackage("top.jionjion.swagger.controller"))
                // 文档路径
                .paths(PathSelectors.any())
                .build()
                // 配置文档的全局参数..
                .globalRequestParameters(getGlobalRequestParameters())
                .globalResponses(HttpMethod.GET, getGlobalResponseMessage())
                .globalResponses(HttpMethod.POST, getGlobalResponseMessage());
    }

    /** 接口信息，包括标题, 联系人, 版本等 */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Swagger3接口文档示例")
                .description("这是描述...嘤嘤嘤")
                .contact(new Contact("springfox", "http://springfox.github.io/springfox/", null))
                .version("1.0")
                .build();
    }

    /** 全局,通用参数..每次请求时,必须携带 */
    private List<RequestParameter> getGlobalRequestParameters() {
        List<RequestParameter> parameters = new ArrayList<>();
        parameters.add(new RequestParameterBuilder()
                .name("version")
                .description("客户端的版本号")
                .required(true)
                .in(ParameterType.QUERY)
                .query(q -> q.model(m -> m.scalarModel(ScalarType.STRING)))
                .required(false)
                .build());
        return parameters;
    }

    /** 全局,通用响应信息 */
    private List<Response> getGlobalResponseMessage() {
        return Collections.singletonList(new ResponseBuilder().code("404").description("找不到资源").build());
    }


    /** 如果被拦截,添加放行 */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.
                addResourceHandler("/swagger-ui/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/springfox-swagger-ui/")
                .resourceChain(false);
    }

    /** 支持跳转,兼容2.0 */
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController( "/swagger-ui/")
                .setViewName("forward:" + "/swagger-ui/index.html");
    }
}
```

#### 实体类

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

@ApiModel("api通用返回数据")
public class ResultDto<T> {

    /** uuid 序列化ID */
    private static final long serialVersionUID = 7498486491866881777L;

    /** 标识代码，0表示成功，非0表示出错 */
    @ApiModelProperty("标识代码,0表示成功，非0表示出错")
    private Integer code;

    /** 提示信息，通常供报错时使用 */
    @ApiModelProperty("提示信息,供报错时使用")
    private String msg;

    /** 正常返回时返回的数据 */
    @ApiModelProperty("返回的数据")
    private T data;

    /** 无参数构造器 */
    public ResultDto() {

    }

    /** 全参数构造器 */
    public ResultDto(Integer status, String msg, T data) {
        this.code = status;
        this.msg = msg;
        this.data = data;
    }

    /** 返回成功数据 */
    public ResultDto<T> success(T data) {
        return new ResultDto<T>(ResponseCode.SUCCESS.getCode(), ResponseCode.SUCCESS.getMsg(), data);
    }

    public static ResultDto<?> success(Integer code,String msg) {
        return new ResultDto<>(code, msg, null);
    }

    /** 返回出错数据 */
    public static ResultDto<?> error(ResponseCode code) {
        return new ResultDto<>(code.getCode(), code.getMsg(), null);
    }

    public Integer getCode() {
        return code;
    }
    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }
    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
}
```

#### 数据库对象

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.math.BigDecimal;

@ApiModel("商品模型")
public class Goods {
    /** 商品id */
    @ApiModelProperty("商品id")
    Long goodsId;

    /** 商品名称 */
    @ApiModelProperty("商品名称")
    private String goodsName;

    /** 商品标题 */
    @ApiModelProperty("商品标题")
    private String subject;

    /** 商品价格 */
    @ApiModelProperty("商品价格")
    private BigDecimal price;

    /** 库存 */
    @ApiModelProperty("商品库存")
    int stock;

    public Long getGoodsId() {
        return this.goodsId;
    }
    public void setGoodsId(Long goodsId) {
        this.goodsId = goodsId;
    }

    public String getGoodsName() {
        return this.goodsName;
    }
    public void setGoodsName(String goodsName) {
        this.goodsName = goodsName;
    }

    public String getSubject() {
        return this.subject;
    }
    public void setSubject(String subject) {
        this.subject = subject;
    }

    public BigDecimal getPrice() {
        return this.price;
    }
    public void setPrice(BigDecimal price) {
        this.price = price;
    }

    public int getStock() {
        return this.stock;
    }
    public void setStock(int stock) {
        this.stock = stock;
    }
}
```

#### 接口控制器

```java
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import org.springframework.web.bind.annotation.*;
import springfox.documentation.annotations.ApiIgnore;
import top.jionjion.swagger.dto.ResponseCode;
import top.jionjion.swagger.dto.ResultDto;
import top.jionjion.swagger.pojo.Goods;

import java.math.BigDecimal;
import java.util.Map;

@Api(tags = "商品信息管理接口")
@RestController
@RequestMapping("/goods")
public class GoodsController {

    /** 使用 3.0 版本的 */
    @Operation(summary = "商品详情,针对得到单个商品的信息")
    @GetMapping("/one")
    public ResultDto<Goods> one(@Parameter(description = "商品id,正整数") @RequestParam(value="goodsid",required = false,defaultValue = "0") Integer goodsid) {
        Goods good = new Goods();
        good.setGoodsId(Long.valueOf(goodsid));
        good.setGoodsName("电子书");
        good.setSubject("学python,学ai");
        good.setPrice(new BigDecimal(60));
        good.setStock(10);
        ResultDto<Goods> result = new ResultDto<>();
        return result.success(good);
    }

    /** 使用 2.0 版本的 */
    @ApiOperation(value = "商品详情,通过路径表达式查询")
    @ApiImplicitParam(name = "id", value = "用户ID", required = true, dataType = "Long")
    @GetMapping("/{id}/good")
    public ResultDto<Goods> findById(@Parameter(description = "商品id,正整数") @PathVariable(value="id") Integer goodsid) {
        Goods good = new Goods();
        good.setGoodsId(Long.valueOf(goodsid));
        good.setGoodsName("教程");
        good.setSubject("学习Java");
        good.setPrice(new BigDecimal(80));
        good.setStock(10);
        ResultDto<Goods> result = new ResultDto<>();
        return result.success(good);
    }

    @ApiOperation(value = "提交订单")
    @ApiImplicitParams({
        @ApiImplicitParam(name="userid",value="用户id",dataTypeClass = Long.class, paramType = "form",example="12345"),
        @ApiImplicitParam(name="goodsid",value="商品id",dataTypeClass = Integer.class, paramType = "form",example="12345"),
        @ApiImplicitParam(name="mobile",value="手机号",dataTypeClass = String.class, paramType = "form",example="13866668888"),
        @ApiImplicitParam(name="comment",value="发货备注",dataTypeClass = String.class, paramType = "form",example="请在情人节当天送到")
    })
    @PostMapping("/order")
    public ResultDto<?> order(@ApiIgnore @RequestParam Map<String,String> params) {
        System.out.println(params);
        return ResultDto.success(ResponseCode.SUCCESS.getCode(), ResponseCode.SUCCESS.getMsg());
    }
}
```



## 注解说明

旧版本的注解在 `io.swagger.annotations.*`  包下, 新版本的注解在 `io.swagger.v3.oas.annotations.*` 包下.

### 注解汇总

| 注解                 | 版本 | 范围                                                 | 作用                                 |
| -------------------- | ---- | ---------------------------------------------------- | ------------------------------------ |
| `@Api`               | 2.0  | `controller` 类                                      | 作为入口, 生成标识类下的所有接口文档 |
| `@ApiOperation`      | 2.0  | `controller` 类中的方法                              | 接口生成描述                         |
| `@ApiImplicitParam`  | 2.0  | `controller` 类中的方法, `@ApiImplicitParams` 注解中 | 接口参数描述, 非显示参数             |
| `@ApiImplicitParams` | 2.0  | `controller` 类中的方法                              | 接口参数组描述, 非显示参数           |
| `@ApiParam`          | 2.0  | `controller` 类中的方法参数前                        | 接口参数描述. 接口显示参数           |
| `@ApiResponse`       | 2.0  | `controller` 类中的方法,`@ApiResponses` 注解中       | 返回状态描述                         |
| `@ApiResponses`      | 2.0  | `controller` 类中的方法                              | 返回状态组描述                       |
| `@ApiModel`          | 2.0  | `Dto` 类                                             | 生成模型类                           |
| `@ApiModelProperty`  | 2.0  | `Dto` 类中的属性                                     | 生成模型类的属性                     |
|                      |      |                                                      |                                      |



### 详细说明

#### `@Api`

