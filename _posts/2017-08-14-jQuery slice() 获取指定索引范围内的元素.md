---
title: jQuery slice() 获取指定索引范围内的元素
date: 2017-08-14
author: asing1elife
categories:
 - jquery
tags:
 - jquery
---
> slice(start [, end]) 获取指定索引范围内的元素  

## 准备一些测试数据
```html
<ul>
  <li>list item 1</li>
  <li>list item 2</li>
  <li>list item 3</li>
  <li>list item 4</li>
  <li>list item 5</li>
</ul>
```

## 将索引为 2 之后的 li 背景色改为红色
```js
$("li").slice(2).css("background-color", "red");
```

## 将索引为 2 之后 4 之前，且不包含 4 的 li 背景色改为红色
```js
$("li").slice(2, 4).css("background-color", "red");
```