# 纯 CSS 实现 Tooltip 效果
> 使用 CSS 可以实现当鼠标悬浮时显示一个小的提示文字内容区  

## 最终效果展现
![](%E7%BA%AF%20CSS%20%E5%AE%9E%E7%8E%B0%20Tooltip%20%E6%95%88%E6%9E%9C/al-tooltip.gif)

## 基础效果
### HTML
1. 对页面 DOM 元素本身没有强制要求，只是用来列举一下配置在 DOM 元素上的自定义属性
2. `al-tooltop` 表示指定的 **className**
3. `tooltop="提示出现在xx"` 表示需要显示在内容区的文字
4. `position="top"` 表示内容区显示的位置
```html
<div id="container">
  <button class="btn al-tooltip" tooltip="提示出现在上方" position="top">TOP-TIP</button>
  <button class="btn al-tooltip" tooltip="提示出现在右侧" position="right">RIGHT-TIP</button>
  <button class="btn al-tooltip" tooltip="提示出现在底部" position="bottom">BOTTOM-TIP</button>
  <button class="btn al-tooltip" tooltip="提示出现在左侧" position="left">LEFT-TIP</button>
</div>
```

### CSS
1. 通过配置在 DOM 元素上的自定义属性，来直接定义内容区样式
2. 定义在 DOM 元素上的 `tooltip="提示出现在xx"` 可以通过 `content: attr(tooltip)` 获取并显示
	* 注意此处的内容是显示在 **伪类元素** 上
3. **CSS 本身没有太多难点，只需要考虑到不同方向对应元素偏移量的控制**
	* 涉及属性包括 `top right bottom left`  
	* `transform: translate(x, y)`
	* `box-shadow: offsetX offsetY blur spread color` ，更多内容可参考 [CSS box-shadow 详解](bear://x-callback-url/open-note?id=358FA3EA-A905-4BE0-A002-0187B3AC8303-315-00002913662BA1AD)
```css
.al-tooltip {
  position: relative;
}

.al-tooltip:hover::before {
  display: block;
}

.al-tooltip::before {
  display: none;
  content: attr(tooltip);
  position: absolute;
  color: #000;
  white-space: nowrap;
  padding: 8px 15px;
  border-radius: 5px;
  z-index: 100;
}

.al-tooltip[position="top"]::before {
  bottom: calc(100% + 10px);
  left: 50%;
  transform: translate(-50%);
  animation: ani-top .3s ease-out;
  box-shadow: 0 5px 10px -2px rgba(0, 0, 0, .4);
}

.al-tooltip[position="right"]::before {
  left: calc(100% + 10px);
  top: 50%;
  transform: translate(0, -50%);
  animation: ani-right .3s ease-out;
  box-shadow: -5px 0 10px -2px rgba(0, 0, 0, .4);
}

.al-tooltip[position="bottom"]::before {
  top: calc(100% + 10px);
  left: 50%;
  transform: translate(-50%);
  animation: ani-bottom .3s ease-out;
  box-shadow: 0 -5px 10px -2px rgba(0, 0, 0, .4);
}

.al-tooltip[position="left"]::before {
  right: calc(100% + 10px);
  top: 50%;
  transform: translate(0, -50%);
  animation: ani-left .3s ease-out;
  box-shadow: 5px 0 10px -2px rgba(0, 0, 0, .4);
}

@keyframes ani-top {
 from {
   opacity: .5;
   bottom: 150%;
 }
}

@keyframes ani-right {
 from {
   opacity: .5;
   left: 150%;
 }
}

@keyframes ani-bottom {
  from {
    opacity: .5;
    top: 150%;
  }
}

@keyframes ani-left {
  from {
    opacity: .5;
    right: 150%;
  }
}
```

## 添加小三角显示
### CSS
1. 内容区本身使用了 `::before` 
2. 那么小三角可以使用 `::after` 表示
3. 小三角本身是没有宽高的，真实内容是通过 `border` 进行显示，更多内容可参考 [CSS绘制三角形](bear://x-callback-url/open-note?id=3A7DF9D9-6CD8-48CA-897A-7DA24E5EE736-294-000054E1FB08C9E0)
	* 显示规则表现为三角形的两侧有边框但是没颜色，底部有边框而且有颜色
	* 例如 🔼 ，则是存在 `border-left` 和 `border-right` 但是没有颜色，只有 `border-bottom` 是存在且有颜色的
4. **同时，为了确保三角和内容区位置显示正确，需要修正内容区距离按钮的距离**
	* 例如顶部的内容区原本距离按钮 `bottom: calc(100% + 5px)` 
	* 在添加了小三角之后，小三角距离按钮为 `bottom: calc(100% + 5px)`
	* 那么内容区距离按钮的距离就应该加上小三角的大小 `bottom: calc(100% + 10px)`
```css
.al-tooltip::after {
  display: none;
  content: '';
  position: absolute;
  z-index: 100;
  width: 0;
	height: 0;
}

.al-tooltip[position="top"]::after {
  bottom: calc(100% + 5px);
  left: 50%;
  transform: translate(-50%);
  animation: ani-top .3s ease-out;
	border-left: 5px solid transparent;
	border-right: 5px solid transparent;
  border-top: 5px solid #fff;
}

.al-tooltip[position="right"]::after {
  left: calc(100% + 5px);
  top: 50%;
  transform: translate(0, -50%);
  animation: ani-right .3s ease-out;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-right: 5px solid #fff;
}

.al-tooltip[position="bottom"]::after {
  animation: ani-bottom .3s ease-out;
  top: calc(100% + 5px);
  left: 50%;
  transform: translate(-50%);
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-bottom: 5px solid #fff;
}

.al-tooltip[position="left"]::after {
  right: calc(100% + 5px);
  top: 50%;
  transform: translate(0, -50%);
  animation: ani-left .3s ease-out;
  border-top: 5px solid transparent;
  border-bottom: 5px solid transparent;
  border-left: 5px solid #fff;
}
```

## 添加主题
### HTML
1. 为 DOM 元素添加 `theme="dark"` 用于指定主题样式
```html
<button class="btn al-tooltip" tooltip="提示出现在右侧" position="right" theme="dark">RIGHT-TIP</button>
```

### CSS
1. 根据从 DOM 元素上匹配到的自定义属性进行相应配置即可
```css
.al-tooltip[theme="dark"]::before {
  color: #fff;
  background-color: #000;
}

.al-tooltip[theme="dark"][position="top"]::after {
  border-top-color: #000;
}

.al-tooltip[theme="dark"][position="right"]::after {
  border-right-color: #000;
}

.al-tooltip[theme="dark"][position="bottom"]::after {
  border-bottom-color: #000;
}

.al-tooltip[theme="dark"][position="left"]::after {
  border-left-color: #000;
}
```