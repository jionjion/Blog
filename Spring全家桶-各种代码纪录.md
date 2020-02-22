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

