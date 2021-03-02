---
title: Spring全家桶-Spring-Validation篇
abbrlink: d70559cf
date: 2021-02-08 10:29:45
categories:
  - Java
  - Spring
tags: [Java, Spring]
---

> 通过 `spring-boot-starter-validation` 引入, 在项目中通过 `@Valid` 注解进行校验.

<!--more-->



## 快速入门

### 引入配置

通过在 `Pom` 文件中, 引入配置

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 添加注解

通过在属性上标注注解,限制属性信息.  更多注解参阅`javax.validation.constraints.*` .

```java
@Data
public class Student {
    /**
     * id 主键
     */
    @NotNull(message = "ID不能为空")
    private Long id;

    /**
     * 姓名
     */
    @NotBlank(message = "姓名不能为空")
    private String name;
}

```

### 校验属性

- 通过 `@Valid` 注解直接调用校验器, 如果失败,抛出异常
- 通过 `@Valid` 注解配合 `BindingResult` , 将校验结果传入. 一般多进行自定义异常处理 `@ExceptionHandler`
- 通过声明 `validate(@Valid T)` 方法, 在其中 `Validation.buildDefaultValidatorFactory().getValidator().validate(T)` 进行校验.

```java
package top.jionjion.controller;

import org.springframework.util.CollectionUtils;
import org.springframework.validation.BindingResult;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import top.jionjion.bean.Student;

import javax.validation.ConstraintViolation;
import javax.validation.Valid;
import javax.validation.Validation;
import java.util.Set;

/**
 * 前端传入校验..
 *
 * @author Jion
 */
@RestController
public class StudentController {

    /**
     * 校验不通过时直接抛异常
     * curl -H "Content-Type:application/json" -X POST --data '{"id":"1","name":"Jion"}' http://127.0.0.1:8080/student/a
     *
     * @param student 传入参数
     * @return 处理结果
     */
    @PostMapping("/student/a")
    public Student studentA(@RequestBody @Valid Student student) {
        return student;
    }

    /**
     * 将校验结果放进BindingResult里面，用户自行判断并处理.交由统一异常处理
     *
     * @param student       传入参数
     * @param bindingResult 验证结果
     * @return 处理结果
     */
    @PostMapping("/student/b")
    public Student studentB(@RequestBody @Valid Student student, BindingResult bindingResult) {
        // 参数校验
        if (bindingResult.hasErrors()) {
            String messages = bindingResult.getAllErrors()
                    .stream()
                    .map(ObjectError::getDefaultMessage)
                    .reduce((m1, m2) -> m1 + "；" + m2)
                    .orElse("参数输入有误！");
            throw new RuntimeException(messages);
        }
        return student;
    }

    /**
     * 统一异常处理
     *
     * @param exception 异常
     * @return 信息
     */
    @ExceptionHandler(RuntimeException.class)
    public String exceptionHandler(RuntimeException exception) {
        return exception.getMessage();
    }

    /**
     * 手动调用验证
     *
     * @param student 传入参数
     * @return 处理结果
     */
    @PostMapping("/student/c")
    public Student studentC(@RequestBody Student student) {
        // 参数校验
        validate(student);
        return student;
    }

    /**
     * 手动校验
     */
    private void validate(@Valid Student student) {
        // 不符合要求的项目
        Set<ConstraintViolation<@Valid Student>> validateSet = Validation.buildDefaultValidatorFactory()
                .getValidator()
                .validate(student);

        // 处理并返回
        if (!CollectionUtils.isEmpty(validateSet)) {
            String messages = validateSet.stream()
                    .map(ConstraintViolation::getMessage)
                    .reduce((m1, m2) -> m1 + "；" + m2)
                    .orElse("参数输入有误！");
            throw new RuntimeException(messages);
        }
    }
}
```



## JSR-303 基础校验

### 布尔值校验

当前布尔属性必须符合要求

- `java.lang.Boolean`

| 注解           | 说明           |
| -------------- | -------------- |
| `@AssertTrue`  | 必须为 `true`  |
| `@AssertFalse` | 必须为 `false` |

### 日期校验

当前日期属性必须符合要求

- `java.util.Date`
- `java.util.Calendar`
- `java.time.Instant`
- `java.time.ZonedDateTime`
- `java.time.LocalDateTime`
- `java.time.LocalDate`
- `java.time.LocalTime`
- `java.time.OffsetDateTime`
- `java.time.OffsetTime`
- `java.time.Year`
- `java.time.YearMonth`
- `java.time.MonthDay`

| 注解                        | 说明                       |
| --------------------------- | -------------------------- |
| `@Future`                   | 必须是一个将来的日期       |
| `@FutureOrPresentOrPresent` | 必须是当前日期或将来的日期 |
| `@PastOrPresentOrPresent`   | 必须是当前日期或过去的日期 |
| `@Past`                     | 必须是一个过去的日期       |
| `@PastOrPresentOrPresent`   | 必须是当前日期或过去的日期 |

### 数字校验

当前数字或字符数字是否符合预期

- `java.math.BigDecimal`
- `java.math.BigInteger`
- `java.lang.CharSequence`
- `java.lang.Byte`
- `java.lang.Short`
- `java.lang.Integer`
- `java.lang.Long`

| 注解                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `@Positive`               | 必须为正数                                                   |
| `@PositiveOrZero`         | 必须正数或者零                                               |
| `@Negative`               | 必须为负数                                                   |
| `@NegativeOrZero`         | 必须为负数或者零                                             |
| `@Min`                    | 必须是数字，并且不能小于指定的值                             |
| `@Max`                    | 必须是一个数字，并且不能大于指定的值                         |
| `@DecimalMin`             | 必须是一个值，并且不能小于指定的值. 不支持浮点字符，可以为字符串数值 |
| `@DecimalMax`             | 必须是一个数字，并且不能大于指定的值. 不支持浮点字符，可以为字符串数值 |
| `@Digits(有效位, 精度位)` | 必须是一个数字，其值必须在指定有效位与精度位中               |

### 通用校验

对于对于某些特定或者通用对象,进行验证

| 注解        | 适用属性                                                     | 说明                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@Null`     | `java.lang.Object`                                           | 必须为 `null`                                                |
| `@NotNull`  | `java.lang.Object`                                           | 必须不为 null                                                |
| `@NotEmpty` | `java.util.Collection` ; `java.lang.Object[]` ; `java.lang.String` | 集合, 数组, 字符不能为空                                     |
| `@NotBlank` | `java.lang.String`                                           | 字符串不能为空串                                             |
| `@Size`     | `java.util.Collection` ; `java.lang.Object[]` ; `java.lang.String` | 字符串长度, 集合, 数组必须在指定范围内 如果对象为`null` ，不触发校验 |
| `@Email`    | `java.lang.String`                                           | 必须为邮箱                                                   |
| `@Pattern`  | `java.lang.String`                                           | 指定字符必须符合指定的正则表达式                             |



## Hibernate 扩展校验

在 `org.hibernate.validator.constraints.*`  包下, 扩展了校验.

| 注解                 | 适用属性                                                     | 说明                                                |
| -------------------- | ------------------------------------------------------------ | --------------------------------------------------- |
| `@Length`            | `java.lang.String`                                           | 指定字符串的长度在范围内                            |
| `@Range`             | `java.lang.CharSequence` ; `java.math.BigDecimal` ; `java.math.BigInteger` ; `java.lang.Byte` ; `java.lang.Short` ; `java.lang.Integer` ; `java.lang.Long` | 在指定范围内数字,或者数字字符串                     |
| `@UniqueElements`    | `java.util.Collection` ; `java.util.List` ; `java.util.Set`  | 集合必须没有重复                                    |
| `@URL`               | `java.lang.String`                                           | 必须为URL格式                                       |
| `@LuhnCheck`         | `java.lang.String`                                           | 通过 `Luhn` 算法计算. 一般用于验证身份识别码,银行卡 |
| `@CreditCardNumber ` | `java.lang.String`                                           | 必须为一个信用卡编码                                |
| `@EAN`               | `java.lang.String`                                           | 必须为 `EAN` 商品编码,商品后面一维码编号.不能有空格 |
| `@ISBN`              | `java.lang.String`                                           | 必须为 `ISBN` 图书编码.是否携带 - 无所谓            |



## 自定校验注解

通过声明自定义注解,并在验证器中验证该自定义类.

### 自定义注解

`@Constraint` 注解指向具体的实现类
`groups` 为对应的分组
 `message` 为默认的消息的key
`List` 支持重复注解

```java
import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;

/**
 * 自定义注解,配置条件.
 * 不能还有空格字符串
 *
 * @author Jion
 */
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = NotContainSpaceValidatorForCharSequence.class)
public @interface NotContainSpace {
    // 默认错误消息, 消息 key
    String message() default "top.jionjion.validation.NotContainSpace.message";

    // 分组
    Class<?>[] groups() default {};

    // 负载
    Class<? extends Payload>[] payload() default {};

    // 指定多个时使用，从而支持重复注解
    @Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @interface List {
        NotContainSpace[] value();
    }
}
```

### 自定义实现

通过实现 `javax.validation.ConstraintValidator` 接口, 并在泛型中指定对应的注解和约束类, 创建自定义验证器.

```java

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

/**
 * 自定义验证器.
 * 不能还有空格字符串
 *
 * @author Jion
 */
public class NotContainSpaceValidatorForCharSequence implements ConstraintValidator<NotContainSpace, CharSequence> {
    private static final String SPACE_STRING = " ";

    /**
     * 验证器初始化方法, 在容器加载验证器时执行.
     *
     * @param notContainSpace 注解对象
     */
    @Override
    public void initialize(NotContainSpace notContainSpace) {
        System.out.println("执行初始化方法....");
    }

    /**
     * 验证方法, 不能含有空格字符串
     *
     * @param property 注解标识的属性
     * @param context  验证器上下文对象
     * @return 是否通过验证
     */
    @Override
    public boolean isValid(CharSequence property, ConstraintValidatorContext context) {
        if (property != null && property.toString().trim().contains(SPACE_STRING)) {
            // 注解中, message 属性约定的字符串...
            String messageTemplate = context.getDefaultConstraintMessageTemplate();

            // 禁用默认提示信息, 在注解中, 默认的message的值
            context.disableDefaultConstraintViolation();

            // 设置提示语.
            context.buildConstraintViolationWithTemplate(messageTemplate).addConstraintViolation();
            return false;
        }
        //  返回实例 ConstraintViolationImpl{interpolatedMessage='不能包含字符串!', propertyPath=name,     rootBeanClass=class top.jionjion.validation.NotContainSpaceValidation, messageTemplate='不能包含字符串!'}
        return true;
    }
}
```

### 使用示例

示例代码

```java
public class NotContainSpaceValidation {

    @NotContainSpace(message = "不能包含字符串!")
    public String name;
}
```



## 分组校验

### 创建分组

创建分组的条件, 必须为接口

```java
/**
 * 分组注解条件.必须为接口
 *
 * @author Jion
 */
public class GroupsEnum {

    /**
     * 新增
     */
    public interface Insert {
    }


    /**
     * 更新
     */
    public interface Update {
    }
}
```



### 使用分组

在需要的验证的属性上, 通过注解的 `groups` 属性, 指定需要的校验的分组条件.

```java
import javax.validation.constraints.NotNull;

/**
 * 分组校验
 *
 * @author Jion
 */
public class GroupsValidation {

    /**
     * 只有在Update分组下，限制才会起作用.
     * 如果想在不指定分组下仍会被校验,需要再次标注
     */
    @NotNull(message = "ID不能为空")
    @NotNull(groups = {GroupsEnum.Update.class}, message = "更新时ID不能为空")
    public Long id;
}
```

### 校验

在对应的控制层,通过 `@Validated` 注解指定对应的分组. 
或者在 `validate(T object, Class<?>... groups)` 中指定分组 

```java
import org.springframework.util.CollectionUtils;
import org.springframework.validation.BindingResult;
import org.springframework.validation.ObjectError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import top.jionjion.bean.Student;

import javax.validation.ConstraintViolation;
import javax.validation.Valid;
import javax.validation.Validation;
import java.util.Set;

/**
 * 分组校验
 *
 * @author Jion
 */
@RestController
public class GroupsController {

    /**
     * 将校验结果放进BindingResult里面，用户自行判断并处理.交由统一异常处理
     *
     * @param obj           传入参数
     * @param bindingResult 验证结果
     * @return 处理结果
     */
    @PutMapping("/groups/b")
    @ResponseBody
    public String studentB(@RequestBody @Validated({GroupsEnum.Update.class}) GroupsValidation obj, BindingResult bindingResult) {
        // 参数校验
        if (bindingResult.hasErrors()) {
            return bindingResult.getAllErrors()
                    .stream()
                    .map(ObjectError::getDefaultMessage)
                    .reduce((m1, m2) -> m1 + "；" + m2)
                    .orElse("参数输入有误！");
        }
        return obj.toString();
    }


    /**
     * 手动调用验证
     *
     * @param student 传入参数
     * @return 处理结果
     */
    @PostMapping("/groups/c")
    public Student studentC(@RequestBody Student student) {
        // 参数校验
        validateWhenUpdate(student);
        return student;
    }

    /**
     * 手动校验
     */
    private void validateWhenUpdate(@Valid Student student) {
        // 不符合要求的项目
        Set<ConstraintViolation<@Valid Student>> validateSet = Validation.buildDefaultValidatorFactory()
                .getValidator()
                // 分组
                .validate(student, GroupsEnum.Update.class);

        // 处理并返回
        if (!CollectionUtils.isEmpty(validateSet)) {
            String messages = validateSet.stream()
                    .map(ConstraintViolation::getMessage)
                    .reduce((m1, m2) -> m1 + "；" + m2)
                    .orElse("参数输入有误！");
            throw new RuntimeException(messages);
        }
    }
}

```

