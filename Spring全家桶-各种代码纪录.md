---
title: Spring全家桶-各种代码纪录
abbrlink: 9ddfcb32
date: 2019-12-04 21:46:11
categories:
  - Java
  - Spring
tags: [Java, Spring]
---



## 各种代码纪录



### `AOP`切面示例

```java
@Aspect
@Component
public class ApiRequestAspect {

    /** 切点 */
    @Pointcut("execution(* top.jionjion.web.console.api..*.*(..))")
    public void executionPointcut(){ }

    /** 通知类型 */
    @Around("executionPointcut()")
    public Object aspectMetho(ProceedingJoinPoint joinPoint) throws Throwable {

        // 织面
        MethodSignature methodSignature =  (MethodSignature) joinPoint.getSignature();
        // 请求, 响应
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        HttpServletResponse response = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse();

        // 执行结果
        Object result = joinPoint.proceed();

        // 执行切点方法
        Method method = methodSignature.getMethod();

        return result;
    }
}
```



### 从当前线程池中获得`Request` | `Response` | `Session`

获得`request`和`response`

```java
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

{
HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
HttpServletResponse response = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse();
}
```

获得 `Session`

```java
import org.springframework.beans.factory.annotation.Autowired;
import javax.servlet.http.HttpSession;

{
    @Autowired
    HttpSession session;    
}
```



### `Controller` 中获得传入参数

#### Form表单参数

通过 `@RequestParam` 注解配合 `Map` 集合框架获得传入参数

```java
@PostMapping("/account/account-change")
public String accountChangeAction(@RequestParam Map<String,String> params){        
    String phone = params.get("phone");
    String email = params.get("email");
    String address = params.get("address");

    return "redirect:/account/account-info";
}
```

通过 `@RequestParam` 注解,指定参数名获得

```java
@PostMapping("/account/account-change")
public String formString(@RequestParam(value = "username",defaultValue = "ambity") String username){
    log.info(username);
    return username;
}
```

直接通过对象接收

```java
public String formToObj(User user){

    log.info(JSON.toJSONString(user));
    return JSON.toJSONString(user);
}
```

#### JSON参数

通过 `@RequestBody` 配合对象或者 `Map` 进行获取

```java
@RequestMapping("/jsonToObj")
public String formToObj(@RequestBody User user){
    log.info(JSON.toJSONString(user));
    return JSON.toJSONString(user);
}

@RequestMapping("/jsonToMap")
public String jsonToMap(@RequestBody Map<String,String> user){
    log.info(JSON.toJSONString(user));
    return JSON.toJSONString(user);
}
```

