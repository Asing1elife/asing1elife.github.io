---
title: URLSearchParams 在 IE11 中无法被识别
date: 2018-06-23
author: asing1elife
categories:
 - vue
 - javascript
tags:
 - vue
 - javascript
---
> URLSearchParams 是用于拼接请求参数的工具类，但在 IE11 中无法被直接识别  

## 解决办法
1. 在项目中安装 [url-search-params-polyfill](https://www.npmjs.com/package/url-search-params-polyfill) 
2. 在需要使用 URLSearchParams 的类中引入即可
3. 引入之后可以按照正常操作使用 URLSearchParams

```javascript
import 'url-search-params-polyfill'

...

function dataTruncateMap (objs) {
  let data = new URLSearchParams()
  ...
}
```