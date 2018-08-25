---
title: HTML5+CSS3 为 input 添加一键删除按钮
date: 2018-08-25
author: asing1elife
categories:
 - css
tags:
 - css
---
> 经常可以看到一些 input 输入框在输入文字后，输入框末尾会出现一键删除的按钮  

## 实现方式
### 基础布局
1. `<input/>` 末尾必须添加 **required** 标记，这是 HTML5 的验证标记
2. 准备一个用于显示删除按钮的 DOM 元素，例如以下代码中的 `<span/>` ，图标自备

```html
<div class="invite-code-panel">
  <input type="text" class="invite-code-input" placeholder="请输入实训计划邀请码/助飞码" required>
  <span class="editable-clear-x"></span>
</div>
```

### CSS 控制按钮显示
1. 删除按钮默认隐藏
2. 通过 `:valid +` 可以控制当 `<input/>` 输入内容后 `<span/>` 则显示

```css
.invite-code-panel .editable-clear-x {
  right: 15px;
  display: none;
}

.invite-code-panel .invite-code-input:valid + .editable-clear-x {
  display: inline;
}
```

## 注意点
1. 需要注意的是如果没有为 `<input/>` 添加 **required** 标记，`:valid +` 属性则不会生效
2. 本文只是实现了按钮的自动隐藏/显示，点击按钮后的删除操作需要通过 JS 自行实现