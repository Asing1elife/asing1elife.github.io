---
title: DateTimePicker 日期时间选择器
date: 2017-05-15
author: asing1elife
categories:
 - jquery
tags:
 - jquery
 - datepicker
---
> Bootstrap 官网的有 DatePicker 和 TimePicker ，但不支持共用  
> 所以有网友结合二者开发出 DateTimePicker 插件，可以一直从年份选择到具体时间  

## 官网
[DateTimePicker](http://www.bootcss.com/p/bootstrap-datetimepicker/)

## 该插件依赖于 Bootstrap 的样式，所以需要先引入其 css 文件
```html
<link href="${ctx }/assets/plugins/bootstrap/css/bootstrap.min.css" rel="stylesheet" type="text/css">
```

## 再引入插件自身 css/js 文件
```html
<link href="${ctx }/assets/plugins/bootstrap-datetimepicker/css/bootstrap-datetimepicker.css" rel="stylesheet" type="text/css"/>
<script src="${ctx }/assets/plugins/bootstrap-datetimepicker/js/bootstrap-datetimepicker.js" type="text/javascript"></script>
```

## 如果需要用到国际化，则需要引入对应语言包
```html
<script src="${ctx }/assets/plugins/bootstrap-datetimepicker/js/locales/bootstrap-datetimepicker.zh-CN.js" type="text/javascript"></script>
```

## 定义一个 input 用于展现内容
```html
<input type="text" class="form-control date-time-picker" name="beginTime" readonly>
```

## 调用插件初始化方法
```js 
$(".date-time-picker").datetimepicker({
    format: "yyyy-mm-dd hh:ii",
    language: "zh-CN",
    weekStart: 1,
    autoclose: true,
    todayBtn: true,
    todayHighlight: true
});
```

## API
1. format : mm_dd_yyyy
	* 日期时间显示的格式
	* 关于分钟的格式符为 ii ，而不是 mm ，错写成 mm 将导致无法选择到正确的分钟数
2. weekStart : 0
	* 每周的开始时间
	* 规则是 0-6 对应周日到周六
3. autoclose : false
	* 日期选择关闭后自动关闭选择面板
4. startView : 2 / month
	* 默认显示的视图
	* 支持写入数字或对应字符串
	* 0 - hour / 1 - day / 2 - month / 3 - year / 4 - decade
5. minView : 0 / hour
	* 能提供的最精确时间
	* 规则与 startView 相同
6. maxView : 4 / decade
	* 能提供的最大时间
	* 规则与 startView 相同
7. todayBtn : false
	* 显示“今天”按钮在面板最下方
	* 为 true 则只是将日期跳转到当前日期
	* 为 linked 则会直接选中当前日期
8. todayHighlight : false
	* 高亮当前日期
9. keyboardNavigation : true
	* 支持方向键改变日期
10. language : en
	* 国际化支持
	* 更改属性后还需要在插件的 locales 目录下引入对应国际化 js 文件