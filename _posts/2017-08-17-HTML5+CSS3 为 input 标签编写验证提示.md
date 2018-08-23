---
title: HTML5+CSS3 为 input 标签编写验证提示
date: 2017-08-17
author: asing1elife
categories:
 - css
tags:
 - css
 - input
---
> 通过 HTML5 + CSS3 可以实现 input 中有无内容的友好提示  

## 方式
## 定义 input 标签并在后面跟上 span 标签
1. input 标签中必须指定 required 属性
```html
<input type="number" name="age" min="0" required><span class="inputTip"></span>
```

## 为 input 标签编写动态 css 样式
1. 当 input 中没有内容时其对应的伪类标签为 invalid
2. 当 input 中存在内容时其对应的伪类标签为 valid
```html
<style type="text/css">
	input:invalid+span::after {
		content: "✖";
		padding-left: 10px;
	}

	input:valid+span::after {
		content: "✓";
		padding-left: 10px;
	}
</style>
```