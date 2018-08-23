---
title: jQuery使用Ajax提交表单
date: 2016-12-28
author: asing1elife
categories:
 - jquery
 - ajax
tags:
 - jquery
 - ajax
 - form
---
> 一般表单数据需要使用表单提交，但其实也可以通过 ajax 将表单数据提交到后台。  

## 解决
```html
<form action="#">
	<input type="text" name="name" value="tom"/>
	<input type="text" name="gender" value="male"/>
</form>

// 将表单数据序列化后将得到name=tom&gender=male格式的数据，可直接用于ajax提交
$("form").serialize();
```