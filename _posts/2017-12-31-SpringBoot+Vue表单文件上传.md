---
title: SpringBoot+Vue表单文件上传
date: 2017-12-31
author: asing1elife
categories:
 - java
 - springboot
 - vue
tags:
 - java
 - springboot
 - vue
 - form
---
> Spring Boot + Vue 的文件上传本身没有什么难点，但如果涉及到是一个表单对象中存在文件，则会比较繁琐  

## 后端实体类
1. Spring Boot中对于文件的接收类型和Spring MVC保持一致，使用**MultipartFile**
2. 与Spring MVC不同的是Spring Boot进行文件上传操作不需要添加配置信息，Spring Boot自身已经默认开启了文件上传功能

```java
...

public class Work {
  ...

    @Transient
    private MultipartFile referenceFile;

  ...
}
```

## 后端接收请求的接口
1. SpringBoot与Vue进行集成，使用**axios**进行异步请求发送，对于接收对象类型的数据一般使用**@RequestBody**的注解将对象解析为JSON格式
2. 但是**MultipartFile**类型的文件无法转换为JSON格式，所以此处需要使用**@ModelAttribute**的注解接收对象信息

```java
...

@RestController
@RequestMapping("/${contextPath}/works")
public class WorkController extends SimpleController<Work, WorkService> {
  ...

    @ApiOperation("保存作业")
    @PostMapping("")
    public ResponseData saveRule(@ModelAttribute Work work) {
  return workService.save(work);
    }

  ...
}
```

## 前端对文件数据的处理
1. 使用默认的文件输入框进行文件上传会影响美观，所以通常将输入框隐藏后通过点击按钮进行调用
2. 由于文件格式的**input**属性是只读的，所以无法使用**v-model**对其数据的更改进行实时获取
3. 所以需要通过**@change**对其数据的更改进行监听，并赋值给表单的对应属性

```javascript
<template>
  <in-form ref="inForm" :form="work" :rules="rules" :is-file="true">
    ...

    <el-button type="success" v-else @click="uploadReferenceFile">
    上传答案 <span v-text="work.reference"></span>
    <input type="file" class="upload-file" ref="referenceFile" @change="setReferenceFile">
    </el-button>

    ...
  </in-form>
</template>

<script type="text/ecmascript-6">
...

  export default {
    ...

    methods: {
...

      // 上传参考答案
      uploadReferenceFile () {
        this.$refs.referenceFile.click()
      },
      // 设置参考答案
      setReferenceFile (item) {
        let currentFile = item.target.files[0]

        this.work.reference = currentFile.name
        this.work.referenceFile = currentFile
      }

...
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  .upload-file
    display none
</style>
```

## 前端发起请求的方式
1. 文件格式的数据无法通过JSON格式进行传递，所以需要使用**FormData**对表单数据进行转换
2. 但**FormData**只能接受**String**和**File**格式的数据，对应**Object**格式的数据无法处理
3. 如果涉及到**Object**格式的数据则需要前后端配合进行数据转换
1. 通常情况下对象中关联的对象只涉及到其中的某一个值，所以前端在处理时可以单独将该值进行传递
2. 后端在接收到该值后，可以在其**Setter**方法中将数据赋予对应的对象即可

```javascript
export function save ({url, data}, isFile) {
  // 带文件的上传功能
  if (isFile) {
    let formData = new FormData()
    
    // 遍历传入的数据
    for (let key in data) {
      // 获取当前值
      let currentData = data[key]
  
      // 对于空值进行过滤
      if (currentData === '') {
        continue
      }
      
      // 将对象中的键值对传入formData中
      formData.append(key, currentData)
    }
    
    data = formData
  }
  
  return fetch({
    url: url,
    method: config.POST,
    data
  })
}
```

## 限制文件大小
1. 只需要在::application.properties::中添加如下配置即可

```yaml
spring.http.multipart.max-file-size=10MB
spring.http.multipart.max-request-size=10MB
```