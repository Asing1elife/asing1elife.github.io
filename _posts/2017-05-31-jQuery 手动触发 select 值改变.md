---
title: jQuery 手动触发 select 值改变
date: 2017-05-31
author: asing1elife
categories:
 - jquery
tags:
 - jquery
---
> select 的 option 默认没有 click 事件，所以无法直接通过 jquery 控制 select 下某个 option 被选中  

## 为 select 定义 change 事件
```js
$myTrainingSelect.change(function () {
	console.log("option change")
})
```

## 为需要被选中的 option 赋值 selected
```js
$myTrainingSelect.find("option[value='1'][data-type='open']").prop("selected", true);
```

## 手动触发 select 的 change 事件
```js
$myTrainingSelect.trigger("change");
```