---
title: Vue 项目从 Vue CLI 2 升级到 Vue CLI 4
date: 2020-01-11
author: asing1elife
categories:
- vue
- vue-cli
- javascript
tags:
- vue
- vue-cli
- javascript
---
> 介绍如何将使用 Vue CLI 2 构建的项目升级到 Vue CLI 4  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址
1. [介绍 - Vue CLI](https://cli.vuejs.org/zh/guide/)
2. [配置参考 - Vue CLI](https://cli.vuejs.org/zh/config/)

## 升级细则
1. 实际升级起来还算比较方便，基本不会影响项目内部的逻辑

### 安装 Vue CLI 4
1. 按照官方文档 [安装 - Vue CLI](https://cli.vuejs.org/zh/guide/installation.html) 直接全局安装即可，之后查看 Vue 版本得到下图结果，说明已经安装成功
	* 如果当前安装了 Vue CLI 的 2.x 版本，官方在上述文档中也给予提示，需要先卸载，再安装
![](http://asing1elife.com/sources/images/EA0E2875-29AA-4F88-B3BE-B1E451DE5454.png)

### 创建新项目
1. 在旧项目上直接升级自然是不太稳妥，还是推荐创建一个新的项目，然后逐步将旧项目的内容迁移到新项目比较稳妥，这样就算迁移失败了，旧项目也不会受到影响
2. 首先通过 `vue create projectName` 创建一个新项目，具体步骤可以参考 [创建一个项目 - Vue CLI](https://cli.vuejs.org/zh/guide/creating-a-project.html)
	* 指定项目名称后，其他使用默认配置即可

### 迁移 package.json
1. 项目创建完之后，需要将旧项目的 `package.json` 中的依赖项都复制到新项目的 `package.json` 中
2. 这时候可以好好检查一下 `dependencies` 和 `devDependencies` 中哪些是需要的，哪些是不需要的，将需要的依赖项复制到新项目的对应节点即可

### 迁移静态资源
1. 如果旧项目中的 `/static/` 目录下存在静态资源，直接全部复制到新项目的 `/public/` 目录即可
2. 同样的，旧项目中 `index.html` 中的内容，也可以直接复制到新项目的 `/public/index.html` 中

### 迁移 src 目录
1. 项目的所有关键内容，自然都是在 `/src/` 目录中，然而这个目录迁移起来几乎没有什么成本
2. 直接将旧项目 `/src/` 目录下的所有内容，复制到新项目的 `/src/` 目录即可

### 创建 vue.config.js ，准备迁移配置
1. Vue CLI 3 之后，官方开始推荐 **零配置搭建项目** ，就像 Java 的 SpringBoot 一样
2. 在 Vue CLI 2 时代，Vue 的项目配置被分散在项目的 `/build/` 和 `/config/` 目录中
3. 然而从 Vue CLI 3 开始，所有的配置都被集成到了 `vue.config.js` 这一个文件中，**并且这个文件默认是不被创建的**
4. 所以我们首先需要在项目根目录创建一个 `vue.config.js` 文件，并且参考 [配置参考 - Vue CLI](https://cli.vuejs.org/zh/config/#vue-config-js) 在文件中添加如下顶层内容

```js
module.exports = {
  // ...
}
```

### 指定项目在开发和生产环境的根请求
1. 如果项目在开发和生产环境有不同的根请求，可以在文件中添加 `publicPath` 进行指定，可以参考 [配置参考 - publicPath](https://cli.vuejs.org/zh/config/#publicpath) ，具体如下
	* 这个配置项之前被叫做 `baseUrl` ，现在已经弃用

```js
module.exports = {
  publicPath: process.env.NODE_ENV === 'production' ? '/sts/' : '/'
}
```

### 迁移项目的开发环境配置
1. 之前项目的开发环境配置是在 `/config/index.js` 的 `module.exports.dev` 节点下
2. 通常会在这个节点下配置开发环境的 `host` 、`port` 以及会通过 `proxyTable` 配置一些对接后端的端口映射代理
3. 这些这些配置也统一在 `vue.config.js` 中完成，可以参考 [配置参考 - devServer](https://cli.vuejs.org/zh/config/#devserver) ，具体如下

```js
module.exports = {
  devServer: {
    // 前端请求的链接
    host: '127.0.0.1',
    // 前端请求的端口
    port: 3002,
    // 代理
    proxy: {
      '/': {
        target: 'http://127.0.0.1:8099',
        changeOrigin: true,
        pathRewrite: {
          '^/': '/'
        }
      }
    }
  }
}
```

### 迁移项目的目录别名
1. 我们在进行项目开发时，通常会给常用的目录指定一些别名，例如 `/components/` 组件目录，完整路径是 `/src/components/` 
2. 为了在引入对应组件时使用更简洁的引入目录，会在 `/build/webpack.base.conf.js` 的 `module.exports.resolve.alias` 节点下配置对应目录的别名
3. 现在这个配置自然也是需要在 `vue.config.js` 中完成，可以参考 [配置参考 - chainWebpack](https://cli.vuejs.org/zh/config/#chainwebpack) ，以及 [配置参考 - webpack 相关](https://cli.vuejs.org/zh/guide/webpack.html)
4. 但上述两份文档中只描述了 `chaninWebpack` 字段的基础内容，没有描述如何配置目录别名，具体代码如下
	* 如果有更多目录需要指定别名，只需要继续在后面添加 `.set(dirName, resolve(absoluteDirName))` 即可

```js
const path = require('path')

function resolve (dir) {
  return path.join(__dirname, dir)
}

module.exports = {
  chainWebpack: (config) => {
    const alias = config.resolve.alias
      alias.set('@', resolve('src'))
      .set('components', resolve('src/components'))
  }
}
```

### 迁移 eslint 配置
1. 正常来说，只需要将旧项目根目录下的 `./eslintrc` 文件复制到新项目的根目录即可
2. 但我的项目在复制完之后启动项目时，总是有几率会抛出这个文件中存在异常的错误，所以就在 `vue.config.js` 中添加了 `lintOnSave: false` 
3. 具体配置可以参考 [配置参考 - lintOnSave](https://cli.vuejs.org/zh/config/#lintonsave)

### 开发环境启动项目
1. 如果之前没有在迁移 `package.json` 内容时，没有直接进行安装，则需要先执行 `npm i`  或者 `npm install` 
2. 之后执行 `npm run serve` 即可启动项目，Vue CLI 2 是通过 `npm run dev` 启动的
3. 启动之后的输出内容也和之前有所不同，如下图
![](http://asing1elife.com/sources/images/85757828-C0A3-48B6-9284-AF05C92E81A4.png)