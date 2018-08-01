---
title: SpringMVC配置i18n
date: 2017-11-14
author: asing1elife
categories:
 - java
 - springmvc
 - i18n
tags:
 - java
 - springmvc
 - i18n
---
> i18n是**internationalization**首字母i和末尾字母n以及中间18个字母的简称，意于国际化  

## 在**/src/main/resources**下新建**messages**文件夹
1. 新增以下文件
    1. message.properties用于默认资源
    2. message_zh_CN.properties用于中文资源
    3. message_en_US.properties用于英文资源

## **spring-config.xml**添加配置信息
```xml
<!-- 配置i18n资源文件 -->
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
  <!-- 资源文件路径 -->
  <property name="basename" value="classpath:messages/message"/>
  <!-- 默认编码 -->
  <property name="defaultEncoding" value="UTF-8"/>
  <!-- 指定默认资源文件 -->
  <property name="useCodeAsDefaultMessage" value="true"/>
</bean>
```

## **spring-mvc.xml**添加配置信息
```xml
<!-- 将Locale信息存放于Session中 -->
<!-- id必须是localeResolver，否则会报cannot change HTTP Head ... -->
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.SessionLocaleResolver"/>

<!-- 配置拦截器获取Locale信息 -->
<mvc:interceptors>
  <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
</mvc:interceptors>
```

## **IndexControler.java**中的添加获取Locale信息方法
```java
public class IndexController {
  
  @Autowired
  private MessageSource messageSource;

  @RequestMapping("")
  public String main(Model model, HttpServletRequest request, HttpServletResponse response) {
    // 设置语言格式
    setLanguage(model, request, response, Locale.getDefault());
    
    return "/index";
  }
  
  @RequestMapping("/{language}")
  public String main(Model model, HttpServletRequest request, HttpServletResponse response, @PathVariable String language) {
    // 分割参数
    String[] languages = language.split("_");
    
    // 设置系统语言
    setLanguage(model, request, response, new Locale(languages[0], languages[1]));
    
    return "/index";
  }
  
  /**
   * 设定语言格式
   */
  private void setLanguage(Model model, HttpServletRequest request, HttpServletResponse response, Locale locale) {
    // 获取LocaleResolver
    LocaleResolver localeResolver = RequestContextUtils.getLocaleResolver(request);
    
    // 设置Locale信息
    localeResolver.setLocale(request, response, locale);
    
    // 传递正确的Locale信息到页面
    model.addAttribute("encoding", messageSource.getMessage("encoding", new Object[0], locale));
  }
  
}
```

## 页面中添加spring的tag用于使用**message.properties**中的标签信息
```html
<%@ taglib uri="http://www.springframework.org/tags" prefix="spring" %>
```

## 页面上使用spring标签获取标签信息
```html
<spring:message code="title" />
```

## 首页提供一个下拉框用于手动切换语言
```html
<div class="form select-langugae-panel">
  <div class="form-group">
    <select class="select-content select-content-btn">
      <option value="zh_CN" <c:if test="${encoding eq 'zh_CN' }">selected</c:if>>中文</option>
      <option value="en_US" <c:if test="${encoding eq 'en_US' }">selected</c:if>>English</option>
    </select> <i class="select-arrow"></i>
  </div>
</div>
```