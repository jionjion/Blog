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



## 流查询

通过返回 `org.apache.ibatis.cursor.Cursor<T>` 接口, 进行流操作.

1. 通过 `SqlSessionFactory` 接口管理 `SqlSession` 执行查询
2. 通过 `TransactionTemplate` 管理事物
3. 在执行方法上添加 `@Transactional` 注解管理事物, 但是 **不能在方法内调用**

`Mapper` 文件

```java
import org.apache.ibatis.cursor.Cursor;
import top.jionjion.mybatis.dto.Student;

public interface StudentCursorQuery {

    /**
     * 查询全部, 流查询
     *
     * @return 结果
     */
    Cursor<Student> findAll();
}
```



`Test` 测试类

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.cursor.Cursor;
import org.apache.ibatis.session.SqlSessionFactory;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionTemplate;
import top.jionjion.mybatis.dto.Student;

import java.io.IOException;

/**
 * 流式查询
 *
 * @author Jion
 */
@Slf4j
@SpringBootTest
class StudentCursorQueryTest {

    @Autowired
    SqlSessionFactory sessionFactory;

    @Autowired
    PlatformTransactionManager transactionManager;

    @Autowired
    StudentCursorQuery cursorQuery;

    /**
     * 通过 SqlSessionFactory 进行流查询
     *
     * @throws Session 获取失败
     */
    @Test
    public void findAllBySqlSessionFactory() throws Exception {
        try (Cursor<Student> cursor = sessionFactory.openSession().getMapper(StudentCursorQuery.class).findAll()) {
            cursor.forEach(student -> log.info("流查询: {}", student));
        }
    }

    /**
     * 通过 TransactionTemplate  进行流查询
     */
    @Test
    public void findAllByTransactionTemplate() {
        TransactionTemplate transactionTemplate = new TransactionTemplate(transactionManager);
        transactionTemplate.execute(status -> {
            try (Cursor<Student> cursor = sessionFactory.openSession().getMapper(StudentCursorQuery.class).findAll()) {
                cursor.forEach(student -> log.info("流查询: {}", student));
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        });
    }

    /**
     * 通过 @Transactional 进行流查询
     */
    @Test
    @Transactional
    public void findAllByTransactional() {
        Cursor<Student> cursor = cursorQuery.findAll();
        cursor.forEach(student -> log.info("流查询: {}", student));
    }

    /**
     * 如果不是手动执行开启数据库连接, 会在Mapper方法执行结束后自动管理
     */
    @Test
    public void findAll() {
        Cursor<Student> cursor = cursorQuery.findAll();
        cursor.forEach(student -> log.info("流查询: {}", student));
        // 抛出 java.lang.IllegalStateException: A Cursor is already closed.
        Assertions.fail("未手动执行事物...");
    }
}
```

