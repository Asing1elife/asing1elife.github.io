---
title: Air-datepicker 日期选择器
date: 2017-10-17
author: asing1elife
categories:
 - jquery
tags:
 - jquery
 - datepciker
---
> air-datepicker 是一款基于 jQuery 的简易日期选择器  

## 官网
[Air Datepicker](http://t1m0n.name/air-datepicker/docs/)

## 引用
1. 引入插件所需 css/js 文件

```html
<link rel="stylesheet" type="text/css" href="plugins/air-datepicker/css/datepicker.min.css">
<script type="text/javascript" src="plugins/jquery.min.js"></script>
<script type="text/javascript" src="plugins/air-datepicker/js/datepicker.js"></script>
<script type="text/javascript" src="plugins/air-datepicker/js/i18n/datepicker.zh.js"></script>
```
2. 定义一个 input 用户展现内容
	* `data-language="zh"` 表示插件使用中文

```html
<input type="text" class="datepicker-input" data-language="zh">
```
3. 调用插件初始化方法

```js
$(function() {
    $(".datepicker-input").datepicker();
});
```

## API
1. **classes** 自定义 className
2. **inline** 是否一直可见
	* false [default]
3. **language** 指定 i18n 国际化语言
	* ru [default]
	* 可如上述在 input 中指定 `data-language="zh"`
	* 也可以在插件初始化方法中通过 `language: zh` 指定
4. **startDate** 初始化显示日期
	* new Date() [default]
5. **firstDay** 每周的开始时间
	* 0-6 表示星期天到星期六
	* 没有默认值，若不指定则根据引入的 i18n 规则决定，例如引入 zh ，则第一天为周一，引入 en ，则第一天为周日
6. **weekends** 指定一组日期为周末
	* [6, 0] [default]
7. **dateFormat** 日期格式
	* 没有默认值，若不指定则根据引入的 i18n 规则决定
8. **keyboardNav**  是否允许键盘操作
	* true [default]
9. **position** 日期选择器相对于输入框的显示位置
	* bottom left [default]
10. **offset** 位置偏移值
	* 12 [default]
11. **view** 默认视图
	* days [default]
	* 可选 days months years
12. **minView** 最小视图
	* days [default]
	* 可选 days months years
13. **minDate** 可被选择的最小日期
	* `minDate: new Date()` 表示今天之前的日期都不可被选中
14. **maxDate** 可被选择的最大日期
15. **disableNavWhenOutOfRange** 可选范围之外的日期是否禁止查看
	* true [default] 
16. **multipleDates** 是否可选择多个日期
	* false [default]
17. **multipleDatesSeparator** 自定义多个日期之间的分隔符
	* , [default]
18. **range** 允许选择日期范围
	* false [default]
	* 无法与 **multipleDates** 同时使用
19. **todayButton** 是否显示[今天]按钮
	* false [default]
	* 点击按钮不会直接选择当前日期，只会跳转到当前日期所在的视图页
20. **clearButton** 是否显示[清除]按钮
	* false [default]
21. **autoClose** 选择日期后，自动关闭面板
	* false [default]
22. **timepicker** 是否开启时间选择器
	* false [default]
23. **timeFormat** 时间格式
	* 没有默认值，若不指定则根据引入的 i18n 规则决定
24. **minHours** 小时数最小值
	* 0 [default]
	* 可选范围 0-23 
25. **maxHours** 小时数最大值
	* 23 [default]
	* 可选范围 0-23 
26. **minMinutes** 分钟数最小值
	* 0 [default]
	* 可选范围 0-59
27. **maxMinutes** 分钟数最大值
	* 59 [default]
	* 可选范围 0-59
28. **hoursStep** 小时的滑动间隔
	* 1 [default]
29. **minutesStep** 分钟的滑动间隔
	* 1 [default]