---
title: CKEditor5 集成 Vue
date: 2020-01-11
author: asing1elife
categories:
- vue
- ckeditor
- javascript
tags:
- vue
- ckeditor
- javascript
---
> 在 Vue 中集成 CKEditor5   

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 写在前面的话
1. 这已经是为同一个产品写的第三篇关于富文本编辑器的笔记了，这东西真的是别人的东西，怎么都不好用，所以说一个好产品里面的富文本编辑器，最终必然是要走向高度定制化
	* 简单在笔记软件中搜索了一下，就有以下三篇，这还只是和 Vue 项目相关的，之前还使用过 [Simditor](https://simditor.tower.im) 开发过好几个项目
![](http://asing1elife.com/sources/images/629C66F6-0FE0-45CE-9973-D0ECCF64794C.png)
2. 最久远的一篇是项目刚开始时直接用的 Github 上别人集成好的组件
3. 之后提出想要在编辑器中增加表单的需求，稍作查询，发现之前使用的这个 vue-quill-editor 是基于 quill 1.3.x 版本，而这个版本的 quill 不支持表格
4. 随后发现 quill 2.x 版本已经发布，是支持表格的，只需要稍作扩展即可把内置的表格功能打开，虽然 2.x 目前还是开发版，但貌似已经有不少人在用
5. 这次没有去找现成的别人集成好的 Vue 版本，而是自己用 quill 2.x 的原始版本集成了一个出来，用着还不错
6. 但用了没几天，就发现 2.x 版本做了一个很要命的修改，他的代码块竟然不再使用标准的 `<pre></pre>` 进行包裹，而是使用 `<div class="ql-code-block"></div>` ，而且代码块里面的每行代码都会被一个单独的 `<div>` 包裹，这就很搞了，在前端显示的时候，其他的代码高亮插件都不好识别了
7. 本来想着 quill 之所以著名就是因为他的模块化，做二次开发非常方便，任何组件都可以重写，但一番尝试后，只能作罢，`/formats/code.js` 这个组件被依赖的地方太多了，实在弄不好，不过更大可能性是我现在的水平有限！！
8. 一条路走不通，最快捷的方式自然是换条路，于是就找到了老牌的 CKEditor ，当然我这种小萌新并不知道这个编辑器，是我领导跟我说这个编辑器历史悠久
9. 经过一番研究，终于成功把 CKEditor5 集成到了产品中，所以又产生了一篇关于富文本编辑器的笔记

## 相关网址
1. [CKEditor 5 - 官网](https://ckeditor.com/ckeditor-5/)
2. [CKEditor 5 - 文档](https://ckeditor.com/docs/ckeditor5/latest/builds/index.html)
3. [CKEditor 5 - Github](https://github.com/ckeditor/ckeditor5)

## 基础功能集成
1. 在官方提供的 [Vue.js component - CKEditor 5 Documentation](https://ckeditor.com/docs/ckeditor5/latest/builds/guides/integration/frameworks/vuejs.html) 中其实已经描述把集成步骤描述的非常清晰，但这里还是需要稍微提点一下

### 快捷版本直接集成
1. 在官方提供的 [CKEditor 5 demo](https://ckeditor.com/ckeditor-5/demo/) 中，有以下几个支持快速集成的版本，就是官方已经把功能模块都集成进去了，直接安装即可快速上手使用
![](http://asing1elife.com/sources/images/D48A58F7-77F7-40DE-AE8B-D3EBB6F1F862.png)

#### 安装依赖包
1. 一般自然是使用 Classic 版本，官方在 Vue 的集成引导中使用的也是这个版本，只需要在项目根目录执行以下安装脚本即可
	* `@ckeditor/ckeditor5-vue` 是官方提供的与 Vue 对接的编辑器框架，内部没有直接集成任何版本的 CKEditor ，但提供了对接任意版本 CKEditor 的接口，源码可以查看 [GitHub - ckeditor/ckeditor5-vue: Official CKEditor 5 Vue.js component.](https://github.com/ckeditor/ckeditor5-vue)
	* `@ckeditor/ckeditor5-build-classic` 是官方提供已经集成好的 Classic 版本的 CKEditor5 ，内部已经提前集成好了上图工具栏中提供的一些功能，源码可以查看 [GitHub - CKEditor 5 Build Classic](https://github.com/ckeditor/ckeditor5-build-classic)

```sh
npm install --save @ckeditor/ckeditor5-vue @ckeditor/ckeditor5-build-classic
```
2. 在官方文档中建议把 `ckeditor5-vue` 进行全局安装，也就是在 `main.js` 中引入
3. 这里其实不建议这么做，因为我们并不是在所有地方都需要使用到编辑器，只是在某些页面会用到，所以没必要全局引入
4. 当然，官方在 [Using component locally](https://ckeditor.com/docs/ckeditor5/latest/builds/guides/integration/frameworks/vuejs.html#using-component-locally) 也提到了如何局部引入，个人还是建议使用这种方式

#### 对编辑器组件进行简单封装
1. 下面提供一个将 `ckeditor5-vue` 和 `ckeditor5-build-classic` 进行简单集成的纯净版组件代码
2. 一般我们在使用外部插件时，都推荐在项目中先构建一个基础组件
3. 比如下面这段代码，就是将 CKEditor 的 Vue 插件和 Classic 插件进行最基础集成后命名为 inEditor 的项目内部组件
4. 之后项目中需要使用编辑器，只需要引入 inEditor 组件即可，极大程度的减少了重复代码
5. 在这个组件里，除了对两者进行集成外，还提供了外部引入 inEditor 时能实现 `v-model` 效果的功能，除此之外没有其他操作

```js
<template>
  <ckeditor :editor="editor" :value="editorData"></ckeditor>
</template>

<script>
  import ClassicEditor from '@ckeditor/ckeditor5-build-classic'
  import CKEditor from '@ckeditor/ckeditor5-vue'

  export default {
    name: 'inEditor',
    props: {
      value: {
        type: String,
        default: ''
      },
      placeholder: {
        type: String,
        default: '请输入内容'
      }
    },
    data () {
      return {
        // 编辑器组件需要获取编辑器实例
        editor: ClassicEditor,
        editorData: '',
        editorConfig: {
          // 可以控制编辑器的提示文本
          placeholder: this.placeholder,
        }
      }
    },
    watch: {
      'value' (val) {
        if (!this.editor) {
          return
        }
        
        // 外部内容发生变化时，将新值赋予编辑器
        if (val && val !== this.editorData) {
          this.editorData = this.value
        }
      },
      'editorData' (val) {
        if (val && val !== this.value) {
          // 编辑器内容发生变化时，告知外部，实现 v-model 双向监听效果
          this.$emit('input', val)
        }
      }
    },
    created() {
      // 编辑器组件创建时将外部传入的值直接赋予编辑器
      this.editorData = this.value
    },
    components: {
      // 编辑器组件的局部注册方式
      ckeditor: CKEditor.component
    }
  }
</script>

<style scoped lang="stylus"></style>
```
7. 但 CKEditor 的快捷集成方式局限性非常大，如果在集成之后，想要扩充工具栏上的功能项，比如增加一个 [GitHub - ckeditor/ckeditor5-code-block](https://github.com/ckeditor/ckeditor5-code-block) 的功能，实现起来就力不从心
8. 反正我通过好几种方式想要在上面这段代码中扩充一个功能，都一直在报错，最后只能尝试使用源码版本再次尝试集成

### 源码版本集成
1. 其实官方在 Vue 的集成文档中，就直接提供了 [源码版本的集成方式](https://ckeditor.com/docs/ckeditor5/latest/builds/guides/integration/frameworks/vuejs.html#using-ckeditor-from-source) ，但这里也有不少坑，当然这些坑其实不是官方的锅，是我自己的
2. 官方在源码版本的集成方式开头就说了需要配置 `vue.config.js` ，并且说到 **Edit the vue.config.js file and use the following configuration. If the file is not present, create it in the root of the application (i.e. next to package.json)** 
3. 我以为就是要在项目根目录创建一个这个文件并添加文档中的对应内容即可，但添加之后没有任何效果
4. 因为这个配置文件，其实是 Vue-CLI 3.x 之后才提供的，而当前项目使用的还是 Vue CLI 2.x ，再加上后来我又没有一直关注 Vue 的版本迭代，都不知道其实 Vue CLI 目前都更新到了 4.x 的版本了
5. 所以说，如果想要进行 CKEditor 源码版本的集成，首先需要确保自己的项目是基于 Vue CLI 3+ 版本构建的
6. 有关 Vue 项目升级 Vue CLI 的方式，可以参考 [Vue 项目从 Vue CLI 2 升级到 Vue CLI 4](bear://x-callback-url/open-note?id=0ED76F8A-350A-4B88-B6EE-FF3C83D25380-991-000077363151022B)

#### 安装依赖包
1. 在官方文档的 [Configuring vue.config.js](https://ckeditor.com/docs/ckeditor5/latest/builds/guides/integration/frameworks/vuejs.html#configuring-vueconfigjs) 中已经明确指出安装依赖包需要指定的脚本，具体如下
	* 下面这段代码是将文档中两个部分指出的需要安装的依赖包进行了统一
	* 可以看到里面包含了 `@ckeditor/ckeditor5-vue` 是之前快捷版本继承时就有用到的
	* 至于 Classic 版本的编辑器依赖包，则是使用的 `@ckeditor/ckeditor5-editor-classic` ，而不是 `@ckeditor/ckeditor5-build-classic`
	* 在这个依赖包后面的一些，就都是一些功能包，以及一个主题包

```sh
npm install --save \
    @ckeditor/ckeditor5-vue \
    @ckeditor/ckeditor5-dev-webpack-plugin \
    @ckeditor/ckeditor5-dev-utils \
    postcss-loader@3 \
    raw-loader@0.5.1 \
    @ckeditor/ckeditor5-editor-classic \
    @ckeditor/ckeditor5-essentials \
    @ckeditor/ckeditor5-basic-styles \
    @ckeditor/ckeditor5-link \
    @ckeditor/ckeditor5-paragraph \
    @ckeditor/ckeditor5-theme-lark
```

2. 一次性安装好依赖包之后，就可以按照 [Configuring vue.config.js](https://ckeditor.com/docs/ckeditor5/latest/builds/guides/integration/frameworks/vuejs.html#configuring-vueconfigjs) 中提供的内容去配置 `vue.config.js`

#### 构建基础组件
1. 官方在 [Using the editor from source](https://ckeditor.com/docs/ckeditor5/latest/builds/guides/integration/frameworks/vuejs.html#using-the-editor-from-source) 也明确的告知了如何通过源码组件进行构建，但这里提两个建议
2. 首先是依旧不建议全局引入 `ckeditor5-vue` ，在二次封装的自定义组件中局部引入即可
3. 其实是不建议按照官方的直接在组件中对 ClassicEditor 进行模块添加，官方构建方式如下图
	* 只截了一部分，具体内容上述链接中有
![](http://asing1elife.com/sources/images/A62B1AFB-0182-4838-91E4-F38012E2034D.png)
4. 建议按照如下图的方式构建自定义的编辑器组件
	* `in-editor` 的父级目录是 `/components/` 
	* `/core/ckeditor.js` 就是对 ClassicEditor 的功能扩展
	* `/plugin/fullscreen.js` 则是一个用于实现全屏功能的自定义插件，其他自定义的插件也会放在这个目录
![](http://asing1elife.com/sources/images/D2760F84-8B52-4087-85C1-A77DC4DD8673.png)

#### 构建 /core/ckeditor.js 内容
1. 对于这个文件中的内容，我们可以参考 [ckeditor5-build-classic/ckeditor.js](https://github.com/ckeditor/ckeditor5-build-classic/blob/master/src/ckeditor.js) 中的内容
2. 这个其实就是之前快速构建版本中提到的 `@ckeditor/ckeditor5-build-classic`
3. 首先将上述链接中的内容整个复制到 `/core/ckeditor.js` 中
4. 然后可以对这个文件中引入的一些依赖包做一个简单过滤，可以将 `UploadAdapter` 、`CKFinder` 、`EasyImage` 移除，这几个功能并不需要
	* 需要同时移除 `import` 和 `ClassicEditor.builtinPlugins` 中的内容
5. 这个时候直接运行代码肯定会报错，因为这其中涉及到的一些依赖的功能包都还没有安装
6. 安装上述构建步骤，已经安装的依赖包只有如下四个

```sh
@ckeditor/ckeditor5-essentials
@ckeditor/ckeditor5-basic-styles
@ckeditor/ckeditor5-link
@ckeditor/ckeditor5-paragraph
```
7. 可以检查一下文件中的引入内容，依次对照，没有被这几个功能包包含的内容都需要被安装，也可以直接前往 [ckeditor5-build-classic/package.json](https://github.com/ckeditor/ckeditor5-build-classic/blob/master/package.json) 查看 `@ckeditor/ckeditor5-build-classic` 安装了哪些依赖包，把没有被过滤掉、且没有安装的依赖包安装即可

#### 构建编辑器组件
1. 因为已经在 `/core/ckeditor.js` 将编辑器核心内容都完成了，所以这里就变的简单和简洁了
2. 将之前通过快速集成方式构建的组件内容直接引入后需要，修改一行内容即可，具体修改如下
	* 可以看到，就只是将之前的 `import ClassicEditor from '@ckeditor/ckeditor5-build-classic'` 后面的内容改成 `'components/in-editor/core/ckeditor'`
	* 其他的内容都和之前的没有区别，在完成了组件集成的同时，组件本身的内容也依旧简洁
3. 现在就可以在具体的功能页去使用这个自定义组件了

```js
<template>
  <ckeditor :editor="editor" :value="editorData"></ckeditor>
</template>

<script>
  import ClassicEditor from 'components/in-editor/core/ckeditor'
  import CKEditor from '@ckeditor/ckeditor5-vue'

  export default {
    ...
  }
</script>

<style scoped lang="stylus"></style>
```

## 扩充工具栏的功能点
1. 如果官方提供的 `@ckeditor/ckeditor5-build-classic` 的功能就完全够用，那么使用上文提供的快速集成方式即可
2. 如果官方提供的功能不够用，那么就需要使用上文提供的源码集成方式，至于工具栏的功能点如何扩充，首先我们可以看一下官方提供了哪些功能包
3. 前往 [GitHub - ckeditor/ckeditor5](https://github.com/ckeditor/ckeditor5) ，直接拉到页面最底部，可以看到 **Features** 部分，就是官方提供的用于扩展的工功能包，如下图
	* 图中只截取了一部分，完整内容上述链接中可以看到
![](http://asing1elife.com/sources/images/DD2E4247-0D0E-4A36-996D-2BE397482523.png)
4. 至于每个功能包应该如何集成和使用，可以前往 [CKEditor 5 Features](https://ckeditor.com/docs/ckeditor5/latest/features/basic-styles.html#installation) 进行查看

### 我集成的一些功能点参考
1. 我在集成后添加了如下图的一些功能，以下会按照现实顺序依次列出
![](http://asing1elife.com/sources/images/760261E7-F36D-4550-9C2E-A1E606AB352F.png)

#### Paragraph 段落和标题
1. `Paragraph` ，位于 `@ckeditor/ckeditor5-paragraph/src/paragraph`
2. 这个功能的下拉框里面还有 `Heading` ，位于 `@ckeditor/ckeditor5-heading/src/heading`
3. Github 地址分别是 [GitHub CKEditor 5 Features - Paragraph](https://github.com/ckeditor/ckeditor5-paragraph) ，以及 [GitHub CKEditor 5 Features - Heading](https://github.com/ckeditor/ckeditor5-heading)
4. 在 `/core/ckeditor.js` 的 `ClassicEditor.defaultConfig` 中添加如下配置可实现自定义，具体还可以参考 [CKEditor 5 Features - Heading](https://ckeditor.com/docs/ckeditor5/latest/features/headings.html)

```js
ClassicEditor.defaultConfig = {
  heading: {
    options: [
      {model: 'paragraph', title: 'Paragraph', class: 'ck-heading_paragraph'},
      {model: 'heading1', view: 'h1', title: 'Heading 1', class: 'ck-heading_heading1'},
      {model: 'heading2', view: 'h2', title: 'Heading 2', class: 'ck-heading_heading2'},
      {model: 'heading3', view: 'h3', title: 'Heading 3', class: 'ck-heading_heading3'}
    ]
  }
}
```

#### Text 文本样式系列
1. `Bold` ，位于 `@ckeditor/ckeditor5-basic-styles/src/bold`
2. `Italic` ，位于 `@ckeditor/ckeditor5-basic-styles/src/italic`
3. `Underline` ，位于 `@ckeditor/ckeditor5-basic-styles/src/underline`
4. `Strikethrough` ，位于 `@ckeditor/ckeditor5-basic-styles/src/strikethrough`
5. Github 地址是 [GitHub CKEditor 5 Features - Basic Styles](https://github.com/ckeditor/ckeditor5-basic-styles)
6. 功能参考在 [CKEditor 5 Features - Basic Styles](https://ckeditor.com/docs/ckeditor5/latest/features/basic-styles.html)

#### Link 文本超链接
1. `Link` ，位于 `@ckeditor/ckeditor5-link/src/link`
2. Github 地址是 [GitHub CKEditor 5 Features - Link](https://github.com/ckeditor/ckeditor5-link)
3. 功能参考在 [CKEditor 5 Features - Link](https://ckeditor.com/docs/ckeditor5/latest/features/link.html)

#### BlockQuote 内容引入标记
1. `BlockQuote` ，位于 `@ckeditor/ckeditor5-block-quote/src/blockquote`
2. Github 地址是 [GitHub CKEditor 5 Features - Block Quote](https://github.com/ckeditor/ckeditor5-block-quote)
3. 功能参考没找到

#### HorizontalLine 内容分隔符
1. `HorizontalLine` ，位于 `@ckeditor/ckeditor5-horizontal-line/src/horizontalline`
2. Github 地址是 [GitHub CKEditor 5 Features - Horizontal Line](https://github.com/ckeditor/ckeditor5-horizontal-line)
3. 功能参考在 [CKEditor 5 Features - Horizontal Line](https://ckeditor.com/docs/ckeditor5/latest/features/horizontal-line.html)

#### Font 字体样式系列
1. `FontColor` ，位于 `@ckeditor/ckeditor5-font/src/fontcolor`
2. `FontBackgroundColor` ，位于 `@ckeditor/ckeditor5-font/src/fontbackgroundcolor`
3. 还支持 `FontFamily` 、`FontSize` ，不过我不需要，就没有引入
4. Github 地址是 [GitHub CKEditor 5 Features - Font](https://github.com/ckeditor/ckeditor5-font)
5. 功能参考在 [CKEditor 5 Features - Font](https://ckeditor.com/docs/ckeditor5/latest/features/font.html)

#### Code 代码系列
1. `Code` ，位于 `@ckeditor/ckeditor5-basic-styles/src/code`
2. `CodeBlock` ，位于 `@ckeditor/ckeditor5-code-block/src/codeblock`
3. Github 地址是 [GitHub CKEditor 5 Features - Code](https://github.com/ckeditor/ckeditor5-code-block)
4. 功能参考在 [CKEditor 5 Features - Code](https://ckeditor.com/docs/ckeditor5/latest/features/code-blocks.html)

#### List 列表
1. `List` ，位于 `@ckeditor/ckeditor5-list/src/list`
2. 这个功能在工具栏上可以通过 `bulletedList` 和 `numberedList` 引入
3. Github 地址是 [GitHub CKEditor 5 Features - List](https://github.com/ckeditor/ckeditor5-list)
4. 功能参考没找到

#### Alignment 文本排列方式
1. `Alignment` ，位于 `@ckeditor/ckeditor5-alignment/src/alignment`
2. Github 地址是 [GitHub CKEditor 5 Features - Alignment](https://github.com/ckeditor/ckeditor5-alignment)
3. 功能参考在 [CKEditor 5 Features - Alignment](https://ckeditor.com/docs/ckeditor5/latest/features/text-alignment.html)

#### Image 图片
1. `Image` ，位于 `@ckeditor/ckeditor5-image/src/image`
2. 这个功能有几个图片扩展功能，分别是
	1. `ImageCaption` ，位于 `@ckeditor/ckeditor5-image/src/imagecaption` ，用于为图片添加大标题
	2. `ImageStyle` ，位于 `@ckeditor/ckeditor5-image/src/imagestyle` ，用于控制图片风格
	3. `ImageToolbar` ，位于 `@ckeditor/ckeditor5-image/src/imagetoolbar` ，用于定义点击图片内容是显示的工具栏
	4. `ImageUpload` ，位于 `@ckeditor/ckeditor5-image/src/imageupload` ，用于图片上传，但具体上传方式还需要自定义，后续会有介绍
	5. `ImageResize` ，位于 `@ckeditor/ckeditor5-image/src/imageresize` ，用于通过拖拽的方式调整图片大小
3. Github 地址是 [GitHub CKEditor 5 Features - Image](https://github.com/ckeditor/ckeditor5-image)
4. 在 `/core/ckeditor.js` 的 `ClassicEditor.defaultConfig` 中添加如下配置可实现自定义，具体还可以参考 [CKEditor 5 Features - Images](https://ckeditor.com/docs/ckeditor5/latest/features/image.html)

```js
ClassicEditor.defaultConfig = {
  image: {
    toolbar: ['imageTextAlternative', '|', 'imageStyle:alignLeft', 'imageStyle:full', 'imageStyle:alignRight'],
    styles: ['full', 'alignLeft', 'alignRight']
  }
}
```

#### Table 表格
1. `Table` ，位于 `@ckeditor/ckeditor5-table/src/table`
2. 这个功能也可以实现选中表格后出现工具栏的效果
	* `TableToolbar` ，位于 `@ckeditor/ckeditor5-table/src/tabletoolbar`
3. Github 地址是 [GitHub CKEditor 5 Features - Table](https://github.com/ckeditor/ckeditor5-table)
4. 在 `/core/ckeditor.js` 的 `ClassicEditor.defaultConfig` 中添加如下配置可实现自定义，具体还可以参考 [CKEditor 5 Features - Tables](https://ckeditor.com/docs/ckeditor5/latest/features/table.html)

```js
ClassicEditor.defaultConfig = {
  table: {
    contentToolbar: [
      'tableColumn',
      'tableRow',
      'mergeTableCells'
    ]
  }
}
```

#### 再提供一下我的 /core/ckeditor.js 内容
```js
import ClassicEditorBase from '@ckeditor/ckeditor5-editor-classic/src/classiceditor'

import Essentials from '@ckeditor/ckeditor5-essentials/src/essentials'
import Autoformat from '@ckeditor/ckeditor5-autoformat/src/autoformat'
import PasteFromOffice from '@ckeditor/ckeditor5-paste-from-office/src/pastefromoffice'

import Paragraph from '@ckeditor/ckeditor5-paragraph/src/paragraph'
import Heading from '@ckeditor/ckeditor5-heading/src/heading'

import Bold from '@ckeditor/ckeditor5-basic-styles/src/bold'
import Italic from '@ckeditor/ckeditor5-basic-styles/src/italic'
import Underline from '@ckeditor/ckeditor5-basic-styles/src/underline'
import Strikethrough from '@ckeditor/ckeditor5-basic-styles/src/strikethrough'

import Link from '@ckeditor/ckeditor5-link/src/link'
import BlockQuote from '@ckeditor/ckeditor5-block-quote/src/blockquote'
import HorizontalLine from '@ckeditor/ckeditor5-horizontal-line/src/horizontalline'

import FontColor from '@ckeditor/ckeditor5-font/src/fontcolor'
import FontBackgroundColor from '@ckeditor/ckeditor5-font/src/fontbackgroundcolor'

import Code from '@ckeditor/ckeditor5-basic-styles/src/code'
import CodeBlock from '@ckeditor/ckeditor5-code-block/src/codeblock'

import List from '@ckeditor/ckeditor5-list/src/list'
import Alignment from '@ckeditor/ckeditor5-alignment/src/alignment'

import Image from '@ckeditor/ckeditor5-image/src/image'
import ImageCaption from '@ckeditor/ckeditor5-image/src/imagecaption'
import ImageStyle from '@ckeditor/ckeditor5-image/src/imagestyle'
import ImageToolbar from '@ckeditor/ckeditor5-image/src/imagetoolbar'
import ImageUpload from '@ckeditor/ckeditor5-image/src/imageupload'
import ImageResize from '@ckeditor/ckeditor5-image/src/imageresize'

import Table from '@ckeditor/ckeditor5-table/src/table'
import TableToolbar from '@ckeditor/ckeditor5-table/src/tabletoolbar'

import Indent from '@ckeditor/ckeditor5-indent/src/indent'
import MediaEmbed from '@ckeditor/ckeditor5-media-embed/src/mediaembed'

export default class ClassicEditor extends ClassicEditorBase {}

ClassicEditor.builtinPlugins = [
  Essentials,
  Autoformat,
  PasteFromOffice,

  Paragraph,
  Heading,

  Bold,
  Italic,
  Underline,
  Strikethrough,

  Link,
  BlockQuote,
  HorizontalLine,

  FontColor,
  FontBackgroundColor,

  Code,
  CodeBlock,

  List,
  Alignment,

  Image,
  ImageCaption,
  ImageStyle,
  ImageToolbar,
  ImageUpload,
  ImageResize,

  Table,
  TableToolbar,

  Indent,
  MediaEmbed
]

ClassicEditor.defaultConfig = {
  toolbar: {
    items: [
      'heading',
      '|',
      'bold',
      'italic',
      'underline',
      'strikethrough',
      '|',
      'link',
      'blockQuote',
      'horizontalLine',
      '|',
      'fontcolor',
      'fontbackgroundcolor',
      '|',
      'code',
      'codeblock',
      '|',
      'bulletedList',
      'numberedList',
      'alignment',
      '|',
      'imageUpload',
      'insertTable'
      // '|',
      // 'indent',
      // 'outdent',
      // 'mediaEmbed'
    ]
  },
  heading: {
    options: [
      {model: 'paragraph', title: 'Paragraph', class: 'ck-heading_paragraph'},
      {model: 'heading1', view: 'h1', title: 'Heading 1', class: 'ck-heading_heading1'},
      {model: 'heading2', view: 'h2', title: 'Heading 2', class: 'ck-heading_heading2'},
      {model: 'heading3', view: 'h3', title: 'Heading 3', class: 'ck-heading_heading3'}
    ]
  },
  image: {
    toolbar: ['imageTextAlternative', '|', 'imageStyle:alignLeft', 'imageStyle:full', 'imageStyle:alignRight'],
    styles: ['full', 'alignLeft', 'alignRight']
  },
  table: {
    contentToolbar: [
      'tableColumn',
      'tableRow',
      'mergeTableCells'
    ]
  },
  language: 'en'
}
```

## 实现图片的服务器上传

### 功能分析
1. 虽然可以通过引入功能包的方式直接实现在工具栏上添加图片上传按钮，但具体的图片上传功能还是需要自己去实现
2. 在 [CKEditor 5 Features - Image Upload](https://ckeditor.com/docs/ckeditor5/latest/features/image-upload/image-upload.html) 中提供了以下几种上传方式
![](http://asing1elife.com/sources/images/5BDA1194-0E53-4C12-8298-005863B78926.png)
3. Easy Image 是将图片上传到 CKEditor 提供的云服务器中，既然是由他们提供云服务，那自然是要收费的
	* 但从名字就可以看出来，这种上传方式肯定是最简单的
4. CKFinder file manager 是将图片上传到一个统一的地址，点击图片上传按钮会先弹出来一个 Finder ，可以对上传的图片进行统一管理
	* 比较复杂，不需要这么炫酷的效果
5. Base64 upload adapter 是将图片通过 Base64 的方式上传，不推荐
6. Simple upload adapter 就是传统的将图片传到服务器，并由服务器返回一个图片的显示地址，用于显示图片
7. 所以我们选择通过 [Simple upload adapter](https://ckeditor.com/docs/ckeditor5/latest/features/image-upload/simple-upload-adapter.html) 的方式上传图片

### 引入 SimpleUpload 功能包
1. 按照 CKEditor 疯狂模块化的风格，SimpleUpload 自然也是个选配的模块功能包，Github 地址是 [GitHub CKEditor 5 Features - Upload](https://github.com/ckeditor/ckeditor5-upload)
2. 使用 `npm install --save @ckeditor/ckeditor5-upload` 即可完成安装
3. 之后在 `/core/ckeditor.js` 中进入引入，方式如下
	* 无关代码已全部省略，完整配置可参考上文中提供的代码内容
	* **需要注意的是，这个功能包并不是单独为 SimpleUpload 准备的，里面还有其他的内容**
	* 所以要拿到具体的属于 SimpleUpload 的文件，需要将引入内容指向 `@ckeditor/ckeditor5-upload/src/adapters/simpleuploadadapter`

```js
import ClassicEditorBase from '@ckeditor/ckeditor5-editor-classic/src/classiceditor'

import SimpleUploadAdapter from '@ckeditor/ckeditor5-upload/src/adapters/simpleuploadadapter'

export default class ClassicEditor extends ClassicEditorBase {}

ClassicEditor.builtinPlugins = [
  SimpleUploadAdapter
]
```

### 在组件中配置上传请求
1. 上传链接的配置的不会放在 `/core/ckeditor.js` 中，而是放在被二次开发的 Vue 组件 `/in-editor/index.vue` 中
2. 因为上传链接最好是支持可以通过外部组件传值进行灵活配置，参考代码如下
	* 无法代码已省略
3. 通过 `editorConfig.simpleUpload.uploadUrl` 就可以指定图片上传到服务器的请求
	* 并且支持从组件外部传入一个具体的图片路径的参数
	* 这里使用 `?filePath=${JSON.stringify(this.imagePath)}` 的方式是因为 SimpleUpload 确实不支持单独配置参数

```js
<template>
  <ckeditor v-model="editorData"
            :editor="editor"
            :config="editorConfig"></ckeditor>
</template>

<script>
  import CKEditor from '@ckeditor/ckeditor5-vue'
  import ClassicEditor from 'components/in-editor/core/ckeditor'

  export default {
    name: 'inEditor',
	  props: {
      imagePath: {
        type: String,
        defaualt: ''
      }
    },
    data () {
      return {
        editor: ClassicEditor,
        editorData: '',
        editorConfig: {
          simpleUpload: {
            uploadUrl: `/api/file/upload/image?filePath=${JSON.stringify(this.imagePath)}`
          }
        }
      }
    }
  }
</script>
```

### 服务端接收上传请求
1. 这里提供 Java 使用 SpringBoot 接收上传的思路，代码如下
	* 接收请求的格式是 **POST**
	* 图片文件通过前端传过来的文件名是 **upload** ，这里需要特别注意
	* 图片的文件地址是可选的，通过 `@RequestParam(required = false)` 指定

```java
@RestController
@RequestMapping("/api/file/upload")
public class FileUploadController {

    @Autowired
    private FileUploadService fileUploadService;

    @PostMapping("/image")
    public String imageUpload(@RequestParam MultipartFile upload, @RequestParam(required = false) String filePath) {
        return fileUploadService.imageUpload(upload, filePath);
    }
}
```
2. 至于前端传到后端的文件名称必须是 **upload** ，这一点真的很神奇
3. 因为我翻遍了文档也没找到配置文件名的参数，最后在 [ckeditor5-upload/simpleuploadadapter.js 第 211 行](https://github.com/ckeditor/ckeditor5-upload/blob/2657a601c7c6dbdf01aae1af328f2789dd7c564e/src/adapters/simpleuploadadapter.js#L211) 找到了答案，如下图
	* 这个 **upload** 的文件名是被写死的
	* 并且这个 `_sendRequest()` 函数的父级 `Adapter` 类在该文件中没有被 export 到外部，所以也无法重写
![](http://asing1elife.com/sources/images/5B8FE429-AE6E-4A0E-A941-475D514E73AD.png)

### 服务端返回上传结果
1. 上传成功或失败的返回内容可以参考 [Simple upload adapter - Server-side configuration](https://ckeditor.com/docs/ckeditor5/latest/features/image-upload/simple-upload-adapter.html#successful-upload)
2. 同样是提供 Java 的上传思路供参考，代码如下

```java
public String imageUpload(MultipartFile file, String filePath) {
    // 上传内容不符合要求，通过如下格式返回
    if (filePath.equalsIgnoreCase("")) {
        return String.format("{\"error\":{\"message\":\"%s\"}}", "请指定图片上传路径");
    }

    // 省略了具体的上传步骤

    // 上传成功后接受图片的显示地址，并按照如下格式返回
    return String.format("{\"url\":\"%s\"}", imageUrl);
}
```

## 为工具栏添加自定义功能
1. CKEditor 支持为编辑器添加自定义的功能，具体可以参考 [Creating a simple plugin](https://ckeditor.com/docs/ckeditor5/latest/framework/guides/creating-simple-plugin.html#step-2-creating-a-plugin)
2. 这里为简单介绍一下我为编辑器添加的全屏模式的实现步骤

### 安装依赖包
1. 要开发自定义功能，需要使用 CKEditor 的基础功能组件 `plugin.js` ，这个组件在 [GitHub CKEditor 5 - Core](https://github.com/ckeditor/ckeditor5-core) 中
2. 要将自定义按钮添加到工具栏中，则需要使用 CKEditor 的按钮组件 `buttonview.js` ，这个组件在 [GitHub CKEditor 5 - UI](https://github.com/ckeditor/ckeditor5-ui) 中
3. 所以只需要运行如下代码即可

```sh
npm install --save @ckeditor/ckeditor5-core \
    @ckeditor/ckeditor5-ui
```

### 创建功能包文件
1. 在官方提供的自定义功能实现引导的最终代码 [Creating a simple plugin - Final Code](https://ckeditor.com/docs/ckeditor5/latest/framework/guides/creating-simple-plugin.html#final-code) 中可以看到，自定义的功能是直接写在了 CKEditor 的构建文件中
2. 这里不推荐这么做，上文中我提供了编辑器组件的推荐目录结构，所以应该在 `/in-editor/plugin/` 目录下创建一个 `fullscreen.js` 文件，完整代码如下
	* 注释写的非常详细，就不再一一讲解了

```js
// 基础功能组件
import Plugin from '@ckeditor/ckeditor5-core/src/plugin'
// 按钮视图组件
import ButtonView from '@ckeditor/ckeditor5-ui/src/button/buttonview'

// 按钮的图标 SVG
const fullscreenIcon = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16">
    <path d="M1.34 8.269a.6.6 0 0 0-.607.592l-.07 5.87a.6.6 0 0 0 .607.607l5.869-.071a.6.6 0 0 0-.015-1.2l-5.254.063.063-5.254a.6.6 0 0 0-.593-.607zM14.73.662L8.861.733a.6.6 0 0 0 .015 1.2l5.254-.063-.063 5.254a.6.6 0 0 0 1.2.015l.07-5.87a.6.6 0 0 0-.607-.607z"/>
</svg>
`

// 自定义组件需要继承基础组件
class Fullscreen extends Plugin {
  // 重写基础组件的初始化函数
  init () {
    // 获取当前编辑器
    const editor = this.editor
    // 从当前编辑器的配置中获取传入的自定义配置名称
    const fullscreen = this.editor.config.get('fullscreen')

    // 将自定义按钮添加到当前编辑器的组件工厂
    editor.ui.componentFactory.add('fullscreen', locale => {
      // 获取当前编辑器的按钮视图
      const view = new ButtonView(locale)

      // 在按钮视图中注入自定义按钮的名称、图标
      view.set({
        label: 'fullscreen',
        icon: fullscreenIcon,
        // 开启提示，当鼠标悬浮到按钮上，会显示 label 指定的 fullscreen 字样
        tooltip: true
      })

      // 点击按钮
      view.on('execute', () => {
        // 执行自定义配置中指定的回调函数
        fullscreen.handler.call()
      })

      // 返回修改后的视图内容
      return view
    })
  }
}

// 对外暴漏自定义组件
export default Fullscreen
```

### 引入自定义功能包
1. 和上文中的 SimpleUpload 一样，在 `/core/ckeditor.js` 中进入引入，方式如下
	* 无关代码已全部省略，完整配置可参考上文中提供的代码内容

```js
import ClassicEditorBase from '@ckeditor/ckeditor5-editor-classic/src/classiceditor'

import Fullscreen from '../plugin/fullscreen'

export default class ClassicEditor extends ClassicEditorBase {}

ClassicEditor.builtinPlugins = [
  Fullscreen
]

ClassicEditor.defaultConfig = {
  toolbar: {
    // 在想要按钮出现的位置引入即可
    items: [
      ...
      'fullscreen'
    ]
  }
}
```

### 在组件中对自定义功能进行配置
1. `el-dialog` 是 ElementUI 的模态窗口，具体用法参考 [Element - component/dialog](https://element.eleme.cn/#/zh-CN/component/dialog)
	* `fullscreen` 表示窗口打开时是直接全屏展现的
2. 在 `el-dialog` 中放置一个编辑器组件，专门用于全屏显示时展现内容
3. 不用实时的监听两个编辑器的内容变化，只需要在全屏窗口显示和关闭时更新对应编辑器的内容即可

```js
<template>
  <div class="in-editor-wrapper">
    <ckeditor v-model="editorData"
              :editor="editor"
              :config="editorConfig"></ckeditor>
    <el-dialog 
      modal
      append-to-body
      fullscreen 
      custom-class="in-editor-modal"
      :visible.sync="fullEditorShow" 
      :title="title">
      <ckeditor v-model="fullEditorData"
                :editor="fullEditor"
                :config="editorConfig"></ckeditor>
    </el-dialog>
  </div>
</template>

<script>
  import CKEditor from '@ckeditor/ckeditor5-vue'
  import ClassicEditor from 'components/in-editor/core/ckeditor'

  export default {
    name: 'inEditor',
    data () {
      return {
        fullEditorShow: false,
        editor: ClassicEditor,
        fullEditor: ClassicEditor,
        editorData: '',
        fullEditorData: '',
        editorConfig: {
          // 自定义的全部显示配置
          fullscreen: {
            // 点击按钮后会执行的回调函数
            handler: () => {
              // 控制用于全屏显示的 Dialog 窗口的显示和关闭
              this.fullEditorShow = !this.fullEditorShow
            }
          }
        }
      }
    },
    watch: {
      // 监听全屏窗口的显示和关闭
      'fullEditorShow' (val) {
        if (val) {
          // 显示时，将当前编辑器的内容赋值给全屏编辑器
          this.fullEditorData = this.editorData
        } else {
          // 关闭时，将全屏编辑器的内容赋值给当前编辑器
          this.editorData = this.fullEditorData
        }
      }
    }
  }
</script>

<style lang="stylus">
  // 全屏样式参考
  .in-editor-modal
    .el-dialog__body
      padding 0
      display flex
      flex-direction column
      overflow hidden
      .ck-editor
        display flex
        flex-direction column
        flex-grow 1
        overflow hidden
        .ck-editor__top
          flex 0 0 38.67px
        .ck-editor__main
          flex-grow 1
          display flex
          overflow hidden
          .ck-content
            flex-grow 1
            overflow auto
</style>
```