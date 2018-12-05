---
title: 纯 CSS 实现 Loading 效果
date: 2018-12-05
author: asing1elife
categories:
 - css
tags:
 - css
---
> 使用一个 div 以及 css 实现 loading 加载器效果  

## 基础效果
### HTML
1. 只需要一个 div 即可

```html
<div class="dount"></div>
```

### CSS
1. 规定 div 的宽高，并指定默认边框宽度
2. 通过 `border-radius: 50%`  将 div 设置为圆形
3. 单独修改左边框颜色 `border-left-color: red;`
4. 通过 `@keyframes` 创建动画方案
	* `from` 和 `to` 表示开始和结束
	* 通过 `transform: rotate(0deg)` 实现 div 旋转
5. 使用 `animation` 指定动画方案
	* `linear` 源自 `animation-timing-function` ，表示线性的，匀速的变化
	* `infinite` 源自 `animation-iteration-count` ，表示无限循环

```css
.dount {
  width: 30px;
  height: 30px;
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-radius: 50%;
  border-left-color: red;
  animation: dount-ani 1.2s linear infinite;
}

@keyframes dount-ani {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}
```

## 颜色改为渐变色
1. 改为使用 0% - 100% 表现动画轨迹
2. 使用最基础的 red green blue 三元素表示渐变
3. 开头和结尾都设置为红色的目标是为了让渐变显示的顺滑一些

```css
@keyframes donut-ani {
  0% {
    transform: rotate(0deg);
    border-left-color: red;
  }

  33% {
    border-left-color: green;
  }

  66% {
    border-left-color: blue;
  }

  100% {
    transform: rotate(360deg);
    border-left-color: red;
  }
}
```

## 边框从 1/4 改为 1/8
1. 为了确保始终是一个元素完成动画效果，所以此处使用 `::after` 来穿件 div 的伪元素
2. 将元素设置为绝对定位可以同时左边和顶部做位移，确保伪元素能覆盖父元素的部分内容
3. 边框圆角只设置左上和右上确保伪元素只遮挡父元素一般的内容

```css
#container .donut::after {
	content: '';
  position: absolute;
  left: -4px;
  top: -4px;
  width: 30px;
  height: 15px;
  border: 4px solid #ddd;
  border-bottom: 0;
  border-radius: 30px 30px 0 0;
}
```