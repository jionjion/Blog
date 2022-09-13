---
title: Spring全家桶-Redis-OM篇
categories:
  - Java
  - Spring
tags:
  - Java
  - Spring
abbrlink: 1836f66c
date: 2022-08-24 14:51:03
---

> 介绍在 Spring 中使用 Redis-OM 框架

<!-- more -->



# 简介

使用 `redis-om` 接口实现对象的缓存储存...

`RedisSearch` 是支持搜索的底层搜索引擎，支持 Text（全文搜索）、Tag（精确匹配搜索）、Numeric（范围查询）、Geo（地理范围查询）和 Vector（向量相似性查询）字段...

## 快速启动

### 引入依赖

```xml
<!-- 依赖 -->
<dependency>
    <groupId>com.redis.om</groupId>
    <artifactId>redis-om-spring</artifactId>
    <version>0.5.2-SNAPSHOT</version>
</dependency>
```

### 实体类

 * 添加 `@Document` 注释即可，定义格式省去了自己做转换，`Redis OM`  直观地将域对象保存在 `Redis` 上
 * 使用 `@Indexed` 添加到字段上, 会对其添加声明式索引.根据属性的数据类型选择适当的搜索字段
 * 使用 `@Searchable` 添加到字段上, 标识其为全文检索

```java
@Data
@Document
public class Website {

    /**
     * 区分唯一主键
     */
    @Id
    @Indexed
    private String id;

    /**
     * 全文检索
     */
    @Searchable(nostem = true)
    private String name;

    /**
     * 声明式检索
     */
    @Indexed
    private Set<String> tags = new HashSet<>();

    @Indexed
    private Integer number;

    @Indexed
    private String url;

    private boolean enable;

}
```



### 接口

通过继承 `com.redis.om.spring.repository.RedisDocumentRepository` 实现对对象的操作

```java
@Component
public interface WebsiteCrudRepository extends RedisDocumentRepository<Website, String> {

}
```



### 启动

通过`@EnableRedisDocumentRepositories` 注解, 指向 `@Document` 和 `RedisDocumentRepository` 接口的具体类..实现将其纳入到 `SpringIOC` 中

```java
@SpringBootApplication
@EnableRedisDocumentRepositories(basePackages = "top.jionjion.redisom.*")
public class RedisOmApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisOmApplication.class, args);
    }

}
```



### 测试类

```java
@Slf4j
@SpringBootTest
@SuppressWarnings({"unused", "OptionalGetWithoutIsPresent"})
class WebsiteCrudRepositoryTest {

    @Autowired
    WebsiteCrudRepository repository;


    /**
     * 获得所有主键, 返回类型为迭代器
     */
    @Test
    void getIdsReturnIterable() {
        // 迭代器查找
        Iterable<String> iterable = repository.getIds();
        iterable.iterator().forEachRemaining(id -> log.info("查询到主键: {}", id));
    }

    /**
     * 获得所有主键, 返回类型为分页组件
     */
    @Test
    void getIdsReturnPage() {
        // 分页组件, 一页10个, 页数不能超过数据库最大数量
        Page<String> page = repository.getIds(Pageable.ofSize(1));
        page.forEach(id -> log.info("查询到主键: {}", id));
    }


    /**
     * 根据类属性获取
     */
    @Test
    void getFieldsByIds() throws NoSuchFieldException {
        List<String> ids = Arrays.asList("1", "2", "3", "4");
        Field nameField = Website.class.getDeclaredField("name");

        MetamodelField<Website, String> nameFieldMeta = new MetamodelField<>(nameField, false);
        Iterable<String> iterable = repository.getFieldsByIds(ids, nameFieldMeta);
        iterable.forEach(id -> log.info("查询到索引: {}", id));

        Assertions.fail("程序有问题.... 等正式版");
    }

    /**
     * 根据主键查询
     */
    @Test
    void findById() {
        Optional<Website> websiteOptional = repository.findById("1");
        log.info("是否存在: {}", websiteOptional.isPresent());
        log.info("查询到{}", websiteOptional.orElse(null));
    }

    /**
     * 根据主键判断是否存在
     */
    @Test
    void existsById() {
        boolean exists = repository.existsById("1");
        log.info("是否存在: {}", exists);
    }

    /**
     * 统计当前数据总和
     */
    @Test
    void count() {
        long count = repository.count();
        log.info("当前成员总数: {}", count);
    }

    /**
     * 获得过期时间
     */
    @Test
    void getExpiration() {
        Long expiration = repository.getExpiration("1");
        log.info("剩余过期时间: {}", expiration);
    }

    /**
     * 分页查询全部
     */
    @Test
    void findAllReturnPageable() {
        Page<Website> page = repository.findAll(Pageable.ofSize(1));
        log.info("查询到分页 {}", page);
    }

    /**
     * 迭代查询全部
     */
    @Test
    void findAllReturnIterable() {
        Iterable<Website> iterable = repository.findAll();
        iterable.iterator().forEachRemaining(website -> log.info("迭代查询到成员: {}", website));
    }

    /**
     * 排序并迭代查询全部
     */
    @Test
    void findAllWithSortReturnIterable() {
        Iterable<Website> iterable = repository.findAll(Sort.by("id"));
        iterable.iterator().forEachRemaining(website -> log.info("迭代查询到成员: {}", website));
    }

    /**
     * 根据ID迭代查询全部
     */
    @Test
    void findAllById() {
        List<String> ids = Arrays.asList("1", "2", "3", "4");
        Iterable<Website> iterable = repository.findAllById(ids);
        iterable.iterator().forEachRemaining(website -> log.info("迭代查询到成员: {}", website));
    }


    /**
     * 第一次调用使用生成的 ID 来检查 id 是否在主键集中（如果是，它将被删除）
     * 第二个调用检查 JSON 文档是否存在（如果存在，它将被删除）
     * 第三次调用使用 JSON.SET 命令保存 JSON 有效负载
     * 最后一次调用将保存文档的主键添加到主键集中
     */
    @Test
    void save() {
        Website website = new Website();
        website.setId("1");
        website.setName("<自传>");
        website.setTags(Collections.singleton("人物传记"));
        website.setUrl("https://top.jionjion.com");
        website.setEnable(true);

        // 保存
        Website dbWebsite = repository.save(website);
        log.info("保存后成员: {}", dbWebsite);
    }

    /**
     * 批量保存
     */
    @Test
    void saveAll() {
        List<Website> websiteList = new ArrayList<>();

        Website website1 = new Website();
        website1.setId("2");
        website1.setName("<游记>");
        website1.setTags(Collections.singleton("人物游记"));
        website1.setUrl("https://top.jionjion.com");
        website1.setEnable(true);
        websiteList.add(website1);

        // 批量保存
        Iterable<Website> dbIterable = repository.saveAll(websiteList);
        log.info("保存后成员: {}", dbIterable);
    }

    /**
     * 根据字段属性更新..
     */
    void updateField() {
        Assertions.fail("程序有问题.... 等正式版");
    }

    /**
     * 根据主键删除
     */
    @Test
    void deleteById() {
        repository.deleteById("1");
    }

    /**
     * 根据主键删除和路径表达式
     */
    @Test
    void deleteByIdWithPath() {
        repository.deleteById("1", Path.ROOT_PATH);
    }

    /**
     * 根据查询结果删除
     */
    @Test
    void delete() {
        Optional<Website> website = repository.findById("1");
        repository.delete(website.get());
    }

    /**
     * 根据ID批量删除
     */
    @Test
    void deleteAllById() {
        List<String> ids = Arrays.asList("1", "2", "3", "4");
        repository.deleteAllById(ids);
    }

    /**
     * 根据迭代删除全部
     */
    @Test
    void deleteAllByIterable() {
        Iterable<Website> iterable = repository.findAll();
        repository.deleteAll(iterable);
    }

    /**
     * 删除全部
     */
    @Test
    void deleteAll() {
        repository.deleteAll();
    }
}
```

