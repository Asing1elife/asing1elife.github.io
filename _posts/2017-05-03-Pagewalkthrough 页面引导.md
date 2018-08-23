---
title: Pagewalkthrough 页面引导
date: 2017-05-03
author: asing1elife
categories:
 - jquery
tags:
 - jquery
---
> pagewalkthrough 可以在页面上生成引导操作，指引用户按照对页面元素进行了解  

## 官网
[jquery-pagewalkthrough](https://github.com/jwarby/jquery-pagewalkthrough)

## 页面插件所需 js/css 文件
```html
<link type="text/css" rel="stylesheet" href="${ctx }/assets/plugins/pagewalkthrough/css/jquery.pagewalkthrough.css" />
<script type="text/javascript" src="${ctx }/assets/plugins/jquery-1.11.1.min.js"></script>
<script type="text/javascript" src="${ctx }/assets/plugins/pagewalkthrough/jquery.pagewalkthrough.js"></script>
```

## 在需要被引导的页面最下方加入引导内容
```html
<div id="walkthrough-content">
    <div id="walkthrough-1">
        <h3>欢迎加入轻实训在线教育平台</h3>

        <p>现在我们先对首页功能做一个初步了解</p>
        <p>点击下一步了解更多...</p>
    </div>
    <div id="walkthrough-2" style="top: 0 !important;">这里是轻实训的LOGO，点击这里可以随时返回首页</div>
    <div id="walkthrough-3">点击这里可以进入我的空间</div>
    <div id="walkthrough-4">点击这里进入系统主菜单</div>
</div>
```

## 初始化插件
```js
$("body").pagewalkthrough({
    name: "introduction",
    steps: [{
        popup: {
            content: "#walkthrough-1",
			  // 弹出方式 : modal/tooltip/mohighlight
            type: "modal"
        }
    }, {
        wrapper: "#logoImage",
        popup: {
            content: "#walkthrough-2",
            type: "tooltip",
            offsetHorizontal: 20,
            offsetVertical: 50,
            offsetArrowVertical: -10,
			  // 弹出层位置 : top/left/right/bottom
            position: "right"
        }
    }, {
        wrapper: ".member-zone-tool",
        popup: {
            content: "#walkthrough-3",
            type: "tooltip",
            offsetHorizontal: 20,
            offsetVertical: 50,
            offsetArrowVertical: -30,
            position: "right"
        }
    }, {
        wrapper: "#memberProfileBtn",
        popup: {
            content: "#walkthrough-4",
            type: "tooltip",
            offsetHorizontal: 0,
            offsetVertical: 50,
            position: "left"
        }
    }],
    onClose: function () {
		  // 引导页关闭触发方法	
	  },
	  // 按钮文字本地化
    buttons: {
        jpwClose: {
            i18n: "点击关闭"
        },
        jpwNext: {
            i18n: "下一步 &rarr;"
        },
        jpwPrevious: {
            i18n: "&larr; 上一步"
        },
        jpwFinish: {
            i18n: "完成 &#10004;"
        }
    }
});
```