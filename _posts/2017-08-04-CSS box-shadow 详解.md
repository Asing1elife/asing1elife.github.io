---
title: CSS box-shadow 详解
date: 2017-08-04
author: asing1elife
categories:
 - css
tags:
 - css
 - shadow
---
> box-shadow 是 CSS3 的语法特性，可以实现为元素添加阴影  

## 语法
```js
/* x轴偏移 y轴偏移 模糊半径 大小 颜色 位置 */
box-shadow: offsetX offsetY blur spread color position;
```

## 解析
### offsetX : x轴偏移
1. 取正值向右偏移，负值向左偏移
2. `box-shadow: 20px 0 10px 0 lightblue;`
3. `box-shadow: -20px 0 10px 0 lightblue;` 

### offsetY : y轴偏移
1. 取正值向右偏移，负值向左偏移
2. `box-shadow: 0 20px 10px 0 lightblue;`
3. `box-shadow: 0 -20px 10px 0 lightblue;`

### blur : 模糊半径
1. 取正值，值越大，阴影越模糊
2. 取值为 0 则完全不模糊
3. 取负值无效，按 0 处理
4. 模糊的表现形式是向四周扩散，但四个边角最淡，想要模糊效果比较均匀，可将元素设置为圆形

### spread : 阴影大小
1. 取正值，值越大，阴影越大
2. 取负值，阴影大小的计算方式是元素宽高减去 spread 值，然后 blur 设置的模糊阴影会向内靠拢

### color : 阴影颜色

### position : 位置
1. 默认阴影向外延展，取值 inset 会将阴影改为向内延展

## 扩展
### box-shadow 和 background 一样支持设置多值
```js
box-shadow: 0 0px 10px 10px lightblue,
			  0 0px 10px 10px lightblue inset;
```

### 物体倒影
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>box-shadow</title>
    <style type="text/css">
    #shadowContainer {
        width: 200px;
        height: 200px;
        background-color: red;
        margin: 200px auto;
        position: relative;
        transition: transform 1s;
        border-radius: 50%;
        animation: circleUp 1s 2s both infinite alternate;
    }

    #shadowContainer::after {
        content: "";
        position: absolute;
        bottom: -50px;
        border-radius: 50%;
        width: 100%;
        height: 10px;
        background-color: rgba(255, 67, 66, 1.000);
        transition: transform 1s;
        box-shadow: 0 0 20px 1px rgba(255, 67, 66, 0.5);
        animation: circleShadowUp 1s 2s both infinite alternate;
    }

    @keyframes circleUp {
    	0% {
			transform: translateY(0);
    	}

    	100% {
			transform: translateY(-40px);
    	}
    }

    @keyframes circleShadowUp {
    	0% {
			transform: translateY(0) scale(1);
    	}

    	100% {
			transform: translateY(40px) scale(0.75);
    	}
    }
    </style>
</head>

<body>
    <div id="shadowContainer"></div>
</body>

</html>
```