---
title: Java基础-Junit5单元测试
abbrlink: b2fc0f7f
date: 2020-11-15 15:04:43
categories:
  - Java
  - Junit5
tags: [Java, Junit5]
---

# 简介

单元测试. [Junit5官网](https://junit.org/junit5/docs/current/user-guide/)


# 注解

## 常用注解

| 注解                 | 范围     | 作用                     | 扩展 |
| -------------------- | -------- | ------------------------ | ---- |
| `@Test`              | 方法     | 测试入口                 |      |
| `@ParameterizedTest` | 方法     | 允许自定义测试方法的入参 | ``@ValueSource`` 指定参数<br />`@EnumSource` 指定注解参数<br />`@MethodSource` 指定方法的返回值作为参数<br />`@CsvSource` 以 CVS 格式指定参数<br />`@ArgumentsSource` 指定类的返回值作为参数, 为 `ArgumentsProvider` 接口的实现 |
| `@RepeatedTest`      | 类, 方法 | 进行重复测试             |      |
| `@TestFactory`       |   方法      |  返回值作为测试工厂                         |      |
| `@TestTemplate` | 方法 | 标识改方法为测试模板 |      |
| `@TestMethodOrder` | 类 | 自定义测试类中的方法顺序 |      |
| `@Order` | 方法 | 自定义测试类中的测试方法顺序 | |
| `@TestInstance` | 类 | 标识测试类的生命周期 |      |
| `@DisplayName` | 类, 方法 | 为测试条例命名 |      |
| `@DisplayNameGeneration` | 类, 方法 | 为测试条例自动生成命名 |      |
| `@BeforeEach` | 方法 | 在每个测试方法前均执行 |      |
| `@AfterEach` | 方法 | 在每个测试方法后均执行 |      |
| `@BeforeAll` | 方法 | 在类中所有测试方法前执行一次 |      |
| `@AfterAll` | 方法 | 在类中所有测试方法后执行一次 |      |
| `@Nested` | 内部类 | 标识以下测试通过内部类完成 |      |
| `@Tag` | 类 | 为当前测试用例贴上标签 |      |
| `@Disabled` | 方法 | 跳过当前测试类 |      |
| `@Timeout` | 方法 | 为测试方法设置超时时间 |      |
| `@ExtendWith` | 类 | 扩展测试类的运行环境 |      |
| `@RegisterExtension` | 类 | 扩展注册信息 |      |
| `@TempDir` | 形参 | 指定参数文件路径 |      |





# 断言
