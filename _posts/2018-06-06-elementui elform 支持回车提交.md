---
title: elementui elform 支持回车提交
date: 2018-06-06
author: asing1elife
categories:
 - vue
 - elementui
tags:
 - vue
 - elementui
---
> el-form 组件默认不支持回车提交，需要对提交按钮进行一下更改  

## 实现方式
1. 在表单的提交按钮上添加 Vue 原生属性 `native-type="submit"` 可以让按钮变为表单提交按钮
2. 当表单中只有一个输入框时，按钮会默认为提交按钮
3. 设置默认的提交按钮后需要阻止表单默认提交事件，在表单上添加 `@submit.native.prevent` 即可

```javascript
<el-form ref="form" :model="user" :rules="rules" class="login-form" @submit.native.prevent>
  <el-row :gutter="20">
    <el-col :span="24">
      <el-form-item prop="username">
        <el-input v-model="user.username" placeholder="请输入用户名" autofocus>
          <in-icon slot="prefix" :name="userIcon"></in-icon>
        </el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input v-model="user.password" type="password" placeholder="请输入密码">
          <in-icon slot="prefix" :name="passwordIcon"></in-icon>
        </el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" native-type="submit" class="submit-btn" @click="submitForm">登录</el-button>
      </el-form-item>
    </el-col>
  </el-row>
</el-form>
```