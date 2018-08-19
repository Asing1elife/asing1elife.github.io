---
title: Javascript 修改 input 验证提示
date: 2017-08-21
author: asing1elife
categories:
 - javascript
tags:
 - java
 - dom
---
> HTML5 为 form 中的 input 提供了一系列的验证提示，通过 Javascript 可以对验证提示的内容进行自定义  

## 定义一个 input 标签，并添加 required 属性
```html
<input type="text" name="username" required>
```

## 定义一个方法，使用 setCustomValidity() 方法添加自定义的验证提示内容
```html
<script type="text/javascript">
function checkInput(input) {
    if (input.value.length < 20) {
        input.setCustomValidity("输入内容不得少于20个字符");
    } else {
        input.setCustomValidity("");
    }
}
</script>
```
	
## 在 input 标签中添加 invalid 属性，指定调用前续方法，并传入当前输入框
```html
<input type="text" name="username" oninvalid="checkInput(this)" required>
```