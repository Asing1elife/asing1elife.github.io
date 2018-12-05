---
title: CSS 与 JS 结合，实现 JS 和 jQuery 插件
date: 2018-12-05
author: asing1elife
categories:
 - css
 - javascript
 - jquery
tags:
 - css
 - javascript
 - jquery
---
> 使用 JS 和 jQuery 编写简易插件  

## JS 封装插件
### 通用目录结构
1. css 插件依赖的基础 css 文件
2. js 插件核心代码文件
3. demo.html 提供给使用者的测试用例

### CSS
1. 加载器的实训效果可参考 [纯 CSS 实现 Loading 效果]()
2. 加载器作为插件使用，需要有一个固定的父级，所以添加 `dount-wrapper` 

```css
.donut-wrapper {
  width: calc(100% - 2px);
  height: 100%;
  background-color: rgba(255, 255, 255, .9);
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  margin: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.donut {
  width: 30px;
  height: 30px;
  border: 4px solid #ddd;
  border-radius: 50%;
  border-left-color: red;
  position: relative;
  animation: donut-ani 1.2s linear infinite;
}

.donut::after {
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

### JS
1. 本次封装使用的是 **闭包模式**

```js
// 首先创建一个 自调用匿名函数 (function() {})()
// 用处时可以防止插件本身的定义与外部函数冲突的可能性
let ALoading = (function () {
  // 创建 DOM 对象
  let donutWrapper = document.createElement('div')
  let donut = document.createElement('div')

  // 封装获取 DOM 对象的方法，提高复用性
  let _getDom = function (className) {
    // 通过 class 获取的 DOM 对象需要在最后添加 [0] ，因为获取到的是一个 DOM 数组
    // 从 getElements 可以看出获取的不是单个对象
    return document.getElementsByClassName(className)[0]
  }

  // 封装加载器 DOM 对象的创建过程，并返回最终结果
  let _loadingContent = function () {
    // 设置对应 class ，会在 al-loading.css 中响应
    donutWrapper.setAttribute('class', 'donut-wrapper')
    donut.setAttribute('class', 'donut')

    // 将加载器添加到容器中
    donutWrapper.appendChild(donut)

    return donutWrapper
  }

  // 主体方法
  let _loading = function (target, container) {
    // 获取触发点
    let btn = _getDom(target)
    
	  // 触发点添加点击监听
    btn.addEventListener('click', function () {
      let loadingPanel = _getDom(container)

      // 判断响应点的默认定位，如果是 static 则需要修改定位方式，已确保加载器能够正确放置在响应点中
      // loadingPanel.style.postion 无效是因为只能获取行内样式
      // 使用如下方法才能获取外部样式
      if (window.getComputedStyle(loadingPanel).position === 'static') {
        loadingPanel.style.position = 'relative'
      }

      // 响应点添加加载器
      loadingPanel.appendChild(_loadingContent())

      setTimeout(function () {
        // 响应点移除加载器
        loadingPanel.removeChild(donutWrapper)
      }, 3000)
    })
  }

  // 只把允许外部调用的方法返回到外部，这就是闭包的好处
  return {
    loading: _loading
  }
})()
```

### HTML
1. 通常在 HTML 文件顶部引入 CSS 文件
2. 在文件末尾引入 JS 文件，是为了确保 DOM 元素加载完毕后再执行 JS 文件

```html
<!DOCTYPE html>
<html lang="en">

<head>
	<meta charset="UTF-8">
	<title>AL-LOADING-DEMO</title>
	<link rel="stylesheet" href="css/al-loading.css">
	<style>
		#container {
			width: 700px;
			height: 400px;
			border: 1px solid red;
			margin: 0 auto;
			display: flex;
			padding: 10px;
		}

		#container div[class*="-wrapper"] {
			flex: 0 0 50%;
			display: flex;
			justify-content: center;
			align-items: center;
		}

		#container .button-wrapper {
			border-right: 1px solid red;
			padding-right: 10px;
			flex-direction: column;
		}

		#container .content-wrapper {
			font-family: '微软雅黑';
			font-weight: 700;
			font-size: 18px;
			color: #777;
			letter-spacing: 1px;
		}

		#container .btn {
			display: block;
			border: none;
			padding: 6px 12px;
			font-size: 14px;
			color: #fff;
			background-color: #54de7b;
			border-radius: 5px;
			cursor: pointer;
			transition: all .3s;
		}

		#container .btn:hover {
			background-color: #449d44;
		}
	</style>
</head>

<body>
	<div id="container">
		<div class="button-wrapper">
			<button class="btn js-loading-btn">JS-加载内容</button>
		</div>
		<div class="content-wrapper">等待内容加载</div>
	</div>
	<script src="js/al-loading.js"></script>
	<script>
	  ALoading.loading('js-loading-btn', 'content-wrapper')
	</script>
</body>

</html>
```

## jQuery 封装插件
1. 目录机构没有变化
2. CSS 文件内容没有变化

### JS
1. 最外部的方法体其实还是使用的 **闭包模式** 以及 **自调用匿名函数**
2. 只是此处使用了更简洁的 ES6 **箭头函数**

```js
// 箭头函数 ES6
(($) => {
  // $.fn 是 jQuery 命名空间，表示添加的 loading 函数为全局实例
  // 此处的函数没有使用剪头函数是因为方法体内部需要引用调用方法对象的 this
  $.fn.loading = function (container) {
    // 如果外部使用的是剪头函数，此处的 this 将无法获取到正确的 target
    // 因为箭头函数的函数体内的 this 指的是定义时所在的对象，而不是使用时所在的对象
    let $target = $(this)
    // 使用 ES6 的模版字符串可以更快速的拼接字符串
    let $container = $(`.${container}`)

    // 模版字符串
    let loadingContent =
      `<div class="donut-wrapper">
        <div class="donut"></div>
       </div>`

    // jQuery 通过 on 监听点击事件
    $target.on('click', () => {
      // jQuery 可以直接通过 target.css() 获取对应 DOM 元素的 CSS 样式
      if ($container.css('position') === 'static') {
        // 也支持直接赋值，key-value 形式
        $container.css('position', 'relative')
      }

      // 响应点添加加载器
      $container.append(loadingContent)

      setTimeout(() => {
        // 移除加载器
        $container.find('.donut-wrapper').remove()
      }, 3000)
    })
  }
})(jQuery)
```

### HTML
1. 由于插件依赖 jQuery ，所以需要在插件之前引入 jQuery 的 js 文件
2. 在初始化方法之前，需要先定义 jQuery 的通用加载函数 `$(function() {})`

```html
<script src="http://code.jquery.com/jquery-3.1.0.min.js"></script>
<script src="js/al-loading-jquery.js"></script>

$(function () {
	$('.jquery-loading-btn').loading('content-wrapper')
}) 
```

## jQuery 封装插件，并支持回调
### JS
1. 主体方法的参数新增 **callback** ，表示支持传入 **回调函数**
2. 在按钮的点击事件中通过 `callback.call()` 执行回调函数

```js
(($) => {
  $.fn.loadingPlus = function (container, callback) {
    options.target = $target = $(this)
    options.container = $container = $(`.${container}`)

    let loadingContent =
      `<div class="donut-wrapper">
        <div class="donut"></div>
      </div>`

    $target.on('click', () => {
      if ($container.css('position') === 'static') {
        $container.css('position', 'relative')
      }

      $container.append(loadingContent)

      // jQuery 原生函数，用于判断传入参数是否是一个合法的函数
      if ($.isFunction(callback)) {
        callback.call()
      }
    })
  }

  let options = {
    target: null,
    container: null
  }

  // 将移除加载器的函数对外部公开，允许外部自行决定何时移除加载器
  $.fn.loadingPlus.hide = function () {
    options.container.find('.donut-wrapper').remove()
  }
})(jQuery)
```

### HTML
1. 在参数中传入 **回调函数**

```html
<script src="http://code.jquery.com/jquery-3.1.0.min.js"></script>
<script src="js/al-loading-jquery-callback.js"></script>
<script>
	$(function () {
		$('.jquery-callback-loading-btn').loadingPlus('content-wrapper', function () {
			let $this = $(this)

			setTimeout(function () {
				$this.loadingPlus.hide()
			}, 3000)
		})
	}) 
</script>
```

## jQuery 封装插件，并支持具名参数
### JS
1. 主体方法只传入一个 **Object** 类型的参数 **options**
2. 方法体内部提供一个默认的参数集 **defaults**
3. 当参数传入时使用 `$.extend` 将两个参数进行合并后传给一个新的对象值

```js
(($) => {
  $.fn.loadingPlusSuper = function (options) {
    let $target = $(this)

    // 参数内容合并，defaults 是目标对象，options 是合并对象
    obj = $.extend({}, defaults, options || {})

    let loadingContent =
      `<div class="donut-wrapper">
        <div class="donut"></div>
      </div>`

    $target.on('click', () => {
      if (obj.container.css('position') === 'static') {
        obj.container.css('position', 'relative')
      }

      obj.container.append(loadingContent)

      if ($.isFunction(obj.success)) {
        obj.success.call()
      }
    })
  }

  let obj = {}

  let defaults = {
    container: null,
    // 默认的成功回调中提供一个默认的回调函数
    success: function () {
      setTimeout(function () {
        hide()
      }, 3000)
    }
  }

  // 将对外公开的移除函数进行中转允许内部直接使用
  $.fn.loadingPlusSuper.hide = hide = function () {
    obj.container.find('.donut-wrapper').remove()
  }
})(jQuery)
```

### HTML
1. 传参时可以指定传入参数的名称，没有传入的则使用默认参数

```html
<script src="http://code.jquery.com/jquery-3.1.0.min.js"></script>
<script src="js/al-loading-jquery-option.js"></script>
<script>
	$(function () {
		$('.jquery-option-loading-btn').loadingPlusSuper({
			container: $('.content-wrapper')
		})
	}) 
</script>
```