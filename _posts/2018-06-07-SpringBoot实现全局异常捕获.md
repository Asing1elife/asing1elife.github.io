---
title: SpringBoot实现全局异常捕获
date: 2018-06-07
author: asing1elife
categories:
 - java
 - springboot
tags:
 - java
 - springboot
 - exception
---
> SpringBoot 对异常可以进行全局捕获，按照如下操作即可  

## 创建全局异常捕获器
1. `@RestControllerAdvice` 是对 `@RestController` 的加强
	* 该注解是 Spring2.3 之后提供的新功能，主要用于对原生 Controller 做一些低侵入性的增加辅助
	* 被该注解标注的类，其中的方法会被应用到 `@RestController` 中
	* 作用与 `@RestController` 中被标注 `@RequestMapping` 的方法
2. `@ExceptionHandler` 是自定义错误处理器，使用时可以注明具体需要处理的错误类型
	* 一般需要标注默认异常和自定义异常即可
3. 该错误捕获方式是将所有错误向上一直抛出至 Spring 容器，由 Spring 自行处理
	* 所以如果在之前进行了 try-catch 操作，会导致 Spring 无法捕获到该异常

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    public Logger log = LoggerFactory.getLogger(getClass());

    @ExceptionHandler(TSharkException.class)
    public ResponseData handleTSharkException(TSharkException e) {
        log.error(e.getMessage());

        ResponseData responseData = new ResponseData();
        responseData.setError(e.getMessage());

        return responseData;
    }

    @ExceptionHandler(Exception.class)
    public ResponseData handleException(Exception e) {
        log.error(e.getMessage());

        String exceptionMessage = "";

        ResponseData responseData = new ResponseData();

        if (e.getMessage().contains("rollback")) {
            exceptionMessage = "数据已被关联或使用，无法删除！";
        }

        responseData.setError(exceptionMessage);

        return responseData;
    }

}
```