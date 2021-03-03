---
title: Spring全家桶-Mybatis篇
abbrlink: 347b7bff
date: 2019-12-05 08:09:40
categories:
  - Java
  - Spring
tags: [Java, Spring, Mybatis]
---

> SpringBoot 集成 Mybatis 使用.

<!--more-->



## SpringBoot集成Mabatis

### `Pom` 文件引入

```xml
<!-- mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```



### `Application` 配置文件

```properties
# mybatis
mybatis.mapper-locations=classpath:mapping/*Mapper.xml
mybatis.type-aliases-package=top.jionjion.web.guider.bean
mybatis.configuration.call-setters-on-nulls=true
```



### `Mapper.xml` 文件

创建SQL查询文件

文件 `classpath:resources/mapping/WebsiteMapper.xml`

`namespace` 属性指向对应的接口调用类

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="top.jionjion.web.guider.dao.mapper.WebsiteMapper">

    <resultMap id="websiteResultMap" type="top.jionjion.web.guider.bean.Website">
        <id column="id" property="siteId" jdbcType="INTEGER"/>
        <result column="name" property="siteName" jdbcType="VARCHAR"/>
        <result column="uri" property="siteUri" jdbcType="VARCHAR"/>
    </resultMap>

    <!-- 基础Bean字段 -->
    <sql id="base_column_list">
        id, name, uri
    </sql>

    <!-- 通过主键查询 -->
    <select id="findBySiteId" resultMap="websiteResultMap" parameterType="java.lang.Integer">
        select
        <include refid="base_column_list"/>
        from website
        where id = #{siteId,jdbcType=INTEGER}
    </select>

    <!-- 查询全部 -->
    <select id="findAll" resultMap="websiteResultMap">
        select
        <include refid="base_column_list"/>
        from website
    </select>

    <delete id="deleteBySiteId" parameterType="java.lang.Integer">
        delete from website
        where id = #{siteId,jdbcType=INTEGER}
    </delete>

    <!-- 保存 -->
    <insert id="saveOne" parameterType="top.jionjion.web.guider.bean.Website">
        insert into website(id, name, url)
        values (#{siteId,jdbcType=INTEGER}, #{siteName,jdbcType=VARCHAR}, #{siteUri,jdbcType=VARCHAR})
    </insert>
</mapper>
```



### `Mapper` 接口类

根据XML中定义的方法,创建接口类

```java
@Repository
public interface WebsiteMapper {

    /** 通过主键查询 */
    Website findBySiteId(Integer siteId);

    /** 查询全部 */
    List<Website> findAll();

    /** 删除一个 */
    int deleteBySiteId(Integer siteId);

    /** 保存 */
    int saveOne(Website website);

}
```



### 启用

修改启动类

`@MapperScan` 注解,指向接口类的包路径

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author 14345
 * 	启动类
 */
@MapperScan(basePackages = {"top.jionjion.web.guider.dao.mapper"})
@SpringBootApplication
public class GuiderApplication{
	public static void main(String[] args) {
		SpringApplication.run(GuiderApplication.class, args);
	}
}
```



## 分页插件

### `Pom` 文件引入

```xml
<!-- 分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>RELEASE</version>
</dependency>
```



### `application` 配置文件

```properties
#pagehelper分页插件
pagehelper.helper-dialect=mysql
pagehelper.reasonable=true
pagehelper.support-methods-arguments=true
pagehelper.params=count=countSql
```



### 使用

通过类 `com.github.pagehelper.PageHelper` 进行相关操作

```java
import com.github.pagehelper.PageHelper;

@Service
public class WebsiteService {
	@Autowired
	private WebsiteMapper websiteMapper;	
    
	/** 分页查询 */
	List<Website> findPageAllBySQL(int pageNum , int pageSize){
		PageHelper.startPage(pageNum, pageSize);
		return websiteMapper.findAll();
	}
}
```

