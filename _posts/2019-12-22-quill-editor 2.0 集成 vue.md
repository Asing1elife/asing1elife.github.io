---
title: quill-editor 2.0 集成 vue
date: 2019-12-22
author: asing1elife
categories:
- vue
- quill
- editor
- javascript
tags:
- vue
- quill
- editor
- javascript
---
> 在 vue 项目中使用 quill-editor 2.0  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 写在前面的话
1. 之前做过 [vue-quill-editor 富文本框编辑器](http://asing1elife.com/vue/2017/12/07/vue-quill-editor-%E5%AF%8C%E6%96%87%E6%9C%AC%E6%A1%86%E7%BC%96%E8%BE%91%E5%99%A8/) ，这个版本是基于 Github 上一个已经集成好的组件做的二次开发
2. 这个组件是基于 **quill@1.3.6** ，但是现在需要编辑器能支持插入表格，这个需求 **quill@1.3.6** 做不到
3. 但是 **quill@2.0.0-dev.3** 支持在编辑器中插入表格，不过这不是正式版，而是开发版
4. 而原版的 [vue-quill-editor](https://github.com/surmon-china/vue-quill-editor) 两年前就没更新了，所以 quill 的版本一直停留在 1.x
5. 那么要实现新需求，就只能重新集成一个新的了

## 相关网址
1. [官网关于 2.x 中已经失效的 API 介绍](https://quilljs.com/guides/upgrading-to-1-0/#api)
2. [官方文档](https://quilljs.com/docs/quickstart/)
3. [GitHub 源码](https://github.com/quilljs/quill)

## 基础功能集成
1. 最初的实现引导来自 [在Vue中使用富文本编辑器Quill - SegmentFault 思否](https://segmentfault.com/a/1190000019606714?utm_source=tag-newest)

### 项目引入 quill@2.x
1. 从 [Releases · quilljs/quill · GitHub](https://github.com/quilljs/quill/releases) 可以看到当前官方的正式版是 1.3.7
2. 不过直接进入 [GitHub - quilljs/quill](https://github.com/quilljs/quill) 的首页会发现项目的默认分支已经切换到了开发版，所以 2.x 版本虽然是开发版，但实际用起来不会有什么问题，完全可以放心使用
3. 通过 `npm view quill` 可以看到当前的开发版本的具体版本号是 **2.0.0-dev.3** ，所以直接在项目根目录使用 `npm install quill@2.0.0-dev.3 --save` 安装即可，如下图
![](http://asing1elife.com/sources/images/0EB86585-2AAA-4A66-BA90-934245E32F8D.png)

### 创建编辑器组件
1. 真正会被渲染成编辑器的 DIV 是 `in-editor` ，其外层的 `in-editor-wrapper` 只是作为父级包裹一层
	* 因为当编辑器初始化后，其结构是如下图
	* 所以如果没有父级 DIV 进行包裹，初始化的时候会抛出没有容器的错误
![](http://asing1elife.com/sources/images/BECBBEB6-2423-4FEE-BC1F-AFAF4D56A90B.png)

```js
<template>
  <div class="in-editor-wrapper">
    <div class="in-editor"></div>
  </div>
</template>

<script>
  // 引入原始组件
  import Quill from 'quill'

  // 引入核心样式和主题样式
  import 'quill/dist/quill.core.css'
  import 'quill/dist/quill.snow.css'

  export default {
    name: 'inEditor',
    props: {
      // 用于双向绑定
      value: String
    },
    data () {
      return {
        // 待初始化的编辑器
        editor: null,
        // 配置参数
        options: {
          theme: 'snow',
          modules: {
            // 工具栏的具体配置
            toolbar: {
              container: [
                ['bold', 'italic', 'underline', 'strike'],
                ['blockquote', 'code-block'],
                [{'list': 'ordered'}, {'list': 'bullet'}],
                [{'script': 'super'}],
                [{'indent': '-1'}, {'indent': '+1'}],
                [{'size': ['small', false, 'large', 'huge']}],
                [{'header': [1, 2, 3, 4, 5, 6, false]}],
                [{'color': []}, {'background': []}],
                [{'align': []}],
                ['link', 'image']
              ]
            }
          },
          placeholder: '请输入内容 ...'
        }
      }
    },
    watch: {
      // 监听外部值的传入，用于将值赋予编辑器
      'value' (val) {
        // 如果编辑器没有初始化，则停止赋值
        if (!this.editor) {
          return
        }

        // 获取编辑器当前内容
        let content = this.editor.root.innerHTML

        // 外部传入了新值，而且与当前编辑器的内容不一致
        if (val && val !== content) {
          // 将外部传入的HTML内容转换成编辑器识别的delta对象
          let delta = this.editor.clipboard.convert({
            html: val
          })

          // 编辑器的内容需要接收delta对象
          this.editor.setContents(delta)
        }
      }
    },
    mounted () {
      // 初始化编辑器
      this._initEditor()
    },
    methods: {
      // 初始化编辑器
      _initEditor () {
        // 获取编辑器的DOM容器
        let editorDom = this.$el.querySelector('.in-editor')

        // 初始化编辑器
        this.editor = new Quill(editorDom, this.options)

        // 双向绑定
        this.editor.on('text-change', () => {
          this.$emit('input', this.editor.root.innerHTML)
        })
      }
    }
  }
</script>

<style lang="stylus" type="text/stylus">
  .in-editor-wrapper
    flex-grow 1
    display flex
    flex-direction column
    overflow hidden
    .ql-toolbar
      .ql-formats
        .ql-picker-label
          &::before
            position relative
            top -5px
        button
          i.icon
            font-size 14px
    .ql-container
      flex-grow 1
      height 0
      overflow hidden
</style>
```

### 使用编辑器组件
1. 按正常的组件引入方式即可

```js
<template>
  <in-editor v-model="description"></in-editor>
</template>

<script type="text/ecmascript-6">
import inEditor from "components/in-editor"

export default {
  name: "inEditorForm",
  data() {
    return {
      description: 'Hello World'
    };
  },
  components: {
    inEditor
  }
};
</script>

<style lang="stylus" type="text/stylus"></style>
```

## 启用表格功能

### 配置工具栏
1. 在 2.x 中原始版本就已经支持插入表格，只需要按照如下代码的方式，在 **toolbar** 中配置对应按钮即可

```js
options: {
  theme: 'snow',
  modules: {
    // 启用表格功能
    table: true,
    toolbar: {
      container: [
        // 为了减少代码内容，这里被省略了，可以参考上述代码
        ...
        [
          {'table': 'TD'},
          {'table-insert-row': 'TIR'},
          {'table-insert-column': 'TIC'},
          {'table-delete-row': 'TDR'},
          {'table-delete-column': 'TDC'}
        ]
      ]
    }
  },
  placeholder: this.placeholder
}
```

### 渲染表格按钮的图标
1. 按照上述代码对工具栏进行配置后，只能在工具栏上看到 1 个按钮，后续 4 个按钮都看不到，如下图
![](http://asing1elife.com/sources/images/E36EAB6F-AFEA-4210-A8DC-7EC32DD8B532.png)
2. 其实在上图的第 1 个按钮后面还有 4 个按钮，查看代码结构可知，如下图
	* 可以很清楚的看到，一共生成了 5 个按钮，但只有第一个按钮中有 SVG 格式的图标，后续 4 个按钮都是空的
![](http://asing1elife.com/sources/images/C9D32C3D-5F8A-47E1-83D4-62EB0949B89A.png)
3. 为什么后续 4 个按钮没有图标，难道因为开发版的原因，表格功能的按钮图标还没有完整提供？
4. 其实并不是，查看 2.x 的源码目录，在 `/assets/icons` 目录下可以看到表格的图标非常完整，如下图
![](http://asing1elife.com/sources/images/D9C04DA0-2A26-4533-B4A3-B0BCF86877B5.png)
5. 那么为什么有图标，但是却不显示？
6. 继续查看 2.x 的源码目录，在 `/ui/icons.js` 中找到如下代码
	* 可以看到关于表格的按钮，就只引入了第 1 个
	* 后续 4 个按钮虽然有提供 SVG 文件，但并没有引入

```js
...
import tableIcon from '../assets/icons/table.svg';
...

export default {
  ...
  table: tableIcon,
  ...
};
```
7. 既然按钮图标的 SVG 文件是有的，那么只需要扩充一下 `/ui/icons.js` 即可
	* `let icons = Quill.import('ui/icons')` 就是调用 quill 的原生图标库，从 [Issue #1099 · quilljs/quill · GitHub](https://github.com/quilljs/quill/issues/1099) 学来的
	* 之后通过 Lodash 遍历准备好的自定义图标库，将其逐个插入到 `/ui/icons.js` 中即可
	* **注意，是在编辑器组件初始化之前将自定义图标库插入原生图标库**

```js
<script>
  import Quill from 'quill'
  import _ from 'lodash'
  import { ICON_SVGS } from 'components/in-editor/ui/icon'

  export default {
    ...
    mounted () {
      this._initCustomToolbarIcon()
      this._initEditor()
    },
    methods: {
      _initCustomToolbarIcon () {
        // 获取quill的原生图标库
        let icons = Quill.import('ui/icons')

        // 从自定义图标SVG列表中找到对应的图标填入到原生图标库中
        _.forEach(ICON_SVGS, (iconValue, iconName) => {
          icons[iconName] = iconValue
        })
      }
      ...
    }
  }
</script>
```
8. `ICON_SVGS` 所在文件内容如下
	* 这里其实是一个败笔，本来是想通过和 `/ui/icons.js`	 一样的方式把 SVG 文件通过 `import` 方式引入
	* 但不管怎么尝试获得的都是文本内容，最后不得以只能采用如下方式
	* 后期找到解决方案会再次优化

```js
export const ICON_SVGS = {
  'table-insert-row': `<svg viewbox="0 0 18 18">
    <g class="ql-fill ql-stroke ql-thin ql-transparent">
      <rect height="3" rx="0.5" ry="0.5" width="7" x="4.5" y="2.5"></rect>
      <rect height="3" rx="0.5" ry="0.5" width="7" x="4.5" y="12.5"></rect>
    </g>
    <rect class="ql-fill ql-stroke ql-thin" height="3" rx="0.5" ry="0.5" width="7" x="8.5" y="7.5"></rect>
    <polygon class="ql-fill ql-stroke ql-thin" points="4.5 11 2.5 9 4.5 7 4.5 11"></polygon>
    <line class="ql-stroke" x1="6" x2="4" y1="9" y2="9"></line>
  </svg>`,
  'table-insert-column': `<svg viewbox="0 0 18 18">
    <g class="ql-fill ql-transparent">
      <rect height="10" rx="1" ry="1" width="4" x="12" y="2"></rect>
      <rect height="10" rx="1" ry="1" width="4" x="2" y="2"></rect>
    </g>
    <path class="ql-fill" d="M11.354,4.146l-2-2a0.5,0.5,0,0,0-.707,0l-2,2A0.5,0.5,0,0,0,7,5H8V6a1,1,0,0,0,2,0V5h1A0.5,0.5,0,0,0,11.354,4.146Z"></path>
    <rect class="ql-fill" height="8" rx="1" ry="1" width="4" x="7" y="8"></rect>
  </svg>`,
  'table-delete-row': `<svg viewbox="0 0 18 18">
    <g class="ql-fill ql-stroke ql-thin ql-transparent">
      <rect height="3" rx="0.5" ry="0.5" width="7" x="4.5" y="2.5"></rect>
      <rect height="3" rx="0.5" ry="0.5" width="7" x="4.5" y="12.5"></rect>
    </g>
    <rect class="ql-fill ql-stroke ql-thin" height="3" rx="0.5" ry="0.5" width="7" x="8.5" y="7.5"></rect>
    <line class="ql-stroke ql-thin" x1="6.5" x2="3.5" y1="7.5" y2="10.5"></line>
    <line class="ql-stroke ql-thin" x1="3.5" x2="6.5" y1="7.5" y2="10.5"></line>
  </svg>`,
  'table-delete-column': `<svg viewbox="0 0 18 18">
    <g class="ql-fill ql-transparent">
      <rect height="10" rx="1" ry="1" width="4" x="2" y="6"></rect>
      <rect height="10" rx="1" ry="1" width="4" x="12" y="6"></rect>
    </g>
    <rect class="ql-fill" height="8" rx="1" ry="1" width="4" x="7" y="2"></rect>
    <path class="ql-fill" d="M9.707,13l1.146-1.146a0.5,0.5,0,0,0-.707-0.707L9,12.293,7.854,11.146a0.5,0.5,0,0,0-.707.707L8.293,13,7.146,14.146a0.5,0.5,0,1,0,.707.707L9,13.707l1.146,1.146a0.5,0.5,0,0,0,.707-0.707Z"></path>
  </svg>`
}
```

9. 按照上述方式渲染完成后的表格图标如下图
![](http://asing1elife.com/sources/images/9B38200A-C4BD-47EA-963D-BAA379B68A0F.png)

### 扩展表格按钮的功能
1. 表格按钮的图标渲染完成后，点击这几个表格按钮会发现，只有第 1 个按钮有效果，后续 4 个按钮没有任何反应
2. 这其实很容易想通，毕竟后续 4 个按钮默认连图标都没有渲染，所以功能自然也是需要自定义的
3. 实现方式同样参考自 [在Vue中使用富文本编辑器Quill - SegmentFault 思否](https://segmentfault.com/a/1190000019606714?utm_source=tag-newest)
4. 和上文中稍有区别的位置在于我把按钮的触发事件声明在 `methods` 中，而不是直接声明在 `options.modules.toolbar.handlers` 中
	* 因为按照上文直接声明在配置中，会出现编辑器还没有初始化导致 `this.editor = undefined` 的情况
5. 按照如下代码配置后，编辑器的表格插入功能就大功告成了

```js
<script>
  ...
  export default {
    ...
    data () {
      return {
        ...
        options: {
          modules: {
            toolbar: {
              container: [
                ...
              ],
              handlers: {
                'table': this._tableHandler,
                'table-insert-row': this._tableInsertRowHandler,
                'table-insert-column': this._tableInsertColumnHandler,
                'table-delete-row': this._tableDeleteRowHandler,
                'table-delete-column': this._tableDeleteColumnHandler
              }
            }
          }
        }
      }
    },
    methods: {
      ...
      _tableHandler () {
        this.editor.getModule('table').insertTable(2, 3)
      },
      _tableInsertRowHandler () {
        this.editor.getModule('table').insertRowBelow()
      },
      _tableInsertColumnHandler () {
        this.editor.getModule('table').insertColumnRight()
      },
      _tableDeleteRowHandler () {
        this.editor.getModule('table').deleteRow()
      },
      _tableDeleteColumnHandler () {
        this.editor.getModule('table').deleteColumn()
      }
    }
  }
</script>
```

## 重写图片上传功能
1. quill 的原生图片上传是通过将待上传的图片文件转义成 BASE64 格式后直接插入到文本中
2. 图片会作为文本的一部分被直接传入后端，进行持久化操作
3. BASE64 格式的图片非常冗长，这不是个好的解决方案，所以需要优化

### 引入图片上传模块
1. [GitHub - NextBoy/quill-image-extend-module](https://github.com/NextBoy/quill-image-extend-module) 是在 **quill@1.3.6** 阶段就非常好用的图片上传模块
2. 实测发现在 **quill@2.x** 中也能正常使用，在项目根目录执行 `npm install quill-image-extend-module --save-dev` 安装即可
	* 不过该模块的作者已经在 README 中表示不再维护了，所以有时间的话，我可能会自己参考着重写一份，方便后期维护
3. 接下来在组件按照如下方式中引入模块
	* 从 `quill-image-extend-module` 中引入了两个模块，分别是 `ImageExtend`  和 `QuillWatch`
	* `ImageExtend` 用于进行图片上传功能的重写，因为是自定义的 `modules` ，所以放在 `options.modules` 中，和 `toolbar` 同级
	* `QuillWatch` 用于监听图片上传的操作，监听操作需要放置在 `options.modules.toolbar.handles.image` 中，表示是监听图片按钮的点击操作

```js
<script>
  import Quill from 'quill'
  import { ImageExtend, QuillWatch } from 'quill-image-extend-module'

  Quill.register('modules/ImageExtend', ImageExtend)

  export default {
    ...
    data () {
      return {
        options: {
          modules: {
            toolbar: {
              container: [
                ...
                ['link', 'image']
              ],
              handlers: {
                ...
                'image': this._imageHandler
              }
            },
            ImageExtend: {
              loading: true,
              name: 'image',
              size: 2,
              action: `/api/file/upload/image`,
              response: (res) => {
                return res.data
              }
            }
          }
        }
      }
    },
    methods: {
      _imageHandler () {
        QuillWatch.emit(this.quill.id)
      }
    }
  }
</script>
```
4. 实际上是通过异步的表单上传方式将图片上传到了服务端，如果服务端代码正好是 Java ，又正好是 SpringBoot ，可以参考以下代码作为服务端的图片接收接口

```java
@RestController
@RequestMapping("/api/file/upload")
public class FileUploadController {

    @PostMapping("")
    public ResponseData fileUpload(@RequestParam MultipartFile file) {
        ...
    }
}
```

## 设置图片大小
1. 原生的图片上传功能，不仅图片是作为 BASE64 格式进行保存，而且上传的图片无法修改大小
2. 就算将图片的上传方式修改后，图片依旧是无法直接修改大小的，这块的需求也需要手动实现
3. 在 **quill@1.x** 版本中，[GitHub - kensnyder/quill-image-resize-module](https://github.com/kensnyder/quill-image-resize-module) 是一个非常好用的调整图片大小的扩展模块
4. 在之前使用的 [vue-quill-editor/04-example.vue  · GitHub](https://github.com/surmon-china/vue-quill-editor/blob/master/examples/04-example.vue) 都有提供集成案例
5. 但是当我准备把这个模块引入到 2.x 中的时候，安装过程中提示这个模块引入的依赖包都太老了，疯狂报错，所以最后只能罢休
6. 要自己实现一个功能如此强大的模块，肯定是有点来不及，所以我做了一个极简版，实际操作效果如下图
![](http://asing1elife.com/sources/images/BBB99E9D-4FCE-4F51-A6ED-AFAA28F3B32A.png)

### 实现代码
1. 在初始化编辑器时，通过 `this.editor.root.addEventListener` 监听编辑器内容的双击事件
2. 如果双击的对象是图片，则弹出一个对话框，`this.$prompt()` 是 ElementUI 的全局函数，用于弹出对话框
3. 在对话框中输入准备修改的图片宽度，即可按比例调整图片的大小

```js
<script>
  ...

  export default {
    ...
    methods: {
      _initEditor () {
        let editorDom = this.$el.querySelector('.in-editor')

        this.editor = new Quill(editorDom, this.options)

        // 监听图片点击
        this.editor.root.addEventListener('dblclick', this._initImageResize, false)

        // 双向绑定
        this.editor.on('text-change', () => {
          this.$emit('input', this.editor.root.innerHTML)
        })
      },
      _initImageResize (event) {
        let currentTarget = event.target

        // 判断当前点击的是不是图片
        if (currentTarget && currentTarget.tagName && currentTarget.tagName.toUpperCase() === 'IMG') {
          this.$prompt('请输入宽度', '提示', {
            inputValue: currentTarget.width,
            confirmButtonText: '确定',
            cancelButtonText: '取消'
          }).then(({value}) => {
            // 赋值新宽度
            currentTarget.width = value
          }).catch(() => {})
        }
      }
    }
  }
</script>
```

## 实现编辑器的全屏扩展
1. 有时候受制于表单页的布局，编辑器的可编辑区域会比较拘谨
2. 所以需要在工具栏上提供一个按钮，可以让编辑器实现浏览器范围内的全屏放大
3. 这个需求没有使用现成的扩展组件，而是结合** ElementUI** 的 `el-dialog` 组件实现了效果

### 在编辑器组件中引入 el-dialog
1. `el-dialog` 是 ElementUI 的模态窗口，具体用法参考 [Element - component/dialog](https://element.eleme.cn/#/zh-CN/component/dialog)
	* `fullscreen` 表示窗口打开时是直接全屏展现的
2. 在 `el-dialog` 中增加了一个 DIV ，标记为 `in-full-editor` ，用于在窗口展现时初始化一个用于全屏显示的编辑器

```js
<template>
  <div class="in-editor-wrapper">
    <div class="in-editor"></div>
    <el-dialog 
      modal
      append-to-body
      fullscreen 
      custom-class="in-editor-modal"
      :visible.sync="fullEditorShow" 
      :title="title">
      <div class="in-full-editor"></div>
    </el-dialog>
  </div>
</template>
```

### 在工具栏上增加一个全屏按钮
1. 从如下代码中可以看到，在工具栏上新增了一个 `expand` 按钮，显示效果如下图
![](http://asing1elife.com/sources/images/9BC7AB79-E29B-44FF-B7D2-B496B2186F10.png)
2. 图标内容同样直接存放在之前的 `import { ICON_SVGS } from 'components/in-editor/ui/icon'` 中，内容如下
	* 之前的表格按钮内容被省略了
	* 至于初始化按钮的函数中不需要做任何更改

```js
export const ICON_SVGS = {
  ...
  'expand': `<svg viewBox="0 0 18 18">
    <path d="M5.797 9.76a.6.6 0 1 1 .849.848L2.253 15h3.379a.6.6 0 0 1 .592.503l.008.097a.6.6 0 0 1-.6.6H.8a.612.612 0 0 1-.162-.022A.6.6 0 0 1 .2 15.6v-4.832a.6.6 0 0 1 1.2 0l-.001 3.389zM15.588.2a.61.61 0 0 1 .176.025l.041.016a.373.373 0 0 1 .053.021c.007.006.015.01.022.014a.599.599 0 0 1 .31.588l-.002 4.768a.6.6 0 0 1-1.2 0V2.254L10.6 6.642a.6.6 0 0 1-.765.07l-.083-.07a.6.6 0 0 1 0-.848L14.144 1.4h-3.388a.6.6 0 0 1-.592-.503L10.156.8a.6.6 0 0 1 .6-.6z"/>
   </svg>`
}
```

### 实现两个编辑器之间的数据交互
1. 从如下代码中可以看到，`fullEditor` 的初始化并不在 `mounted()` 函数中，而是通过监听 `fullEditorShow` 的显示来对 `fullEditor` 进行初始化
	* 因为当全屏窗口没有展现之前，全屏编辑器自然也是不需要初始化的
2. `expand` 按钮点击后触发的 `_expandHandler()` 函数会修改 `fullEditorShow = true` ，全屏窗口就会展现
3. 在 `_initFullEditor()` 函数中，对全屏编辑器中了初始化以及赋值操作
	* 首先要判断全屏编辑器是否已经初始化，防止重复初始化，在 **quill@2.x** 中已经移除了 `destory()` 函数，所以需要通过这种方式判断
	* 然后初始化操作需要放置在 `$nextTick()` 函数中，因为初始化需要在 DOM 元素加载完毕后才能进行
	* 最后赋值操作则是直接从 `this.editor` 中获取，但需要通过 `setTimeout(() => {}, 20)` 做一个小小的延迟，防止编辑器的初始化还没有完成
4. 仔细对比 `_initFullEditor()` 和 `_initEditor()` 函数可以发现，在全屏编辑器的初始化函数中，没有对数据进行双向绑定
	* 因为全屏编辑器在打开的情况下，当前浏览器窗口就只能处理编辑器的数据，要想处理其他操作，则需要先退出全屏编辑器
	* 所以只需要监听全屏窗口的打开和关闭状态，在打开时将原始编辑器的内容赋值给全屏编辑器，在关闭时将全屏编辑器的内容赋值给原始编辑器即可

```js
<script>
  ...

  export default {
    ...
    data () {
      return {
        title: '请输入内容',
        editor: null,
        fullEditor: null,
        fullEditorShow: false,
        options: {
          modules: {
            toolbar: {
              container: [
                ...
                ['expand']
              ],
              handlers: {
                ...
                'expand': this._expandHandler
              }
            }
          }
        }
      }
    },
    watch: {
      ...
      'fullEditorShow' (val) {
        if (val) {
          this._initFullEditor()
        } else {
          this.editor.setContents(this.fullEditor.getContents())
        }
      }
    },
    methods: {
      ...
      _initFullEditor () {
        // 全屏编辑器不存在，测初始化
        if (!this.fullEditor) {
          this.$nextTick(() => {
            let fullEditorDom = document.querySelector('.in-full-editor')
            this.fullEditor = new Quill(fullEditorDom, this.options)
            this.fullEditor.root.addEventListener('dblclick', this._initImageResize, false)
          })
        }

        // 将当前编辑器的内容赋值给全屏编辑器
        setTimeout(() => {
          this.fullEditor.setContents(this.editor.getContents())
        }, 20)
      },
      ...
      _expandHandler () {
        this.fullEditorShow = !this.fullEditorShow
      }
    }
  }
</script>
```

### 全屏编辑器在全屏窗口中的样式参考
1. 只提供参考，样式这种东西，根据实际情况灵活调整即可

```stylus
.in-editor-modal
  &.is-fullscreen
    display flex
    flex-direction column
    .el-dialog__header
      flex 0 0 24px
    .el-dialog__body
      flex-grow 1
      padding 0
      display flex
      flex-direction column
      overflow hidden
```