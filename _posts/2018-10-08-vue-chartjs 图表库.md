---
title: vue-chartjs 图表库
date: 2018-10-08
author: asing1elife
categories:
 - vue
 - chartjs
tags:
 - vue
 - chartjs
---
> vue-chartjs 是基于 Chart.js 实现的 vue 版本  

## 官网
1. [vue-chartjs](https://github.com/apertureless/vue-chartjs)
2. [vue-chartjs-docs](https://vue-chartjs.org/#/zh-cn/)

## 需要写在前面的注意点
### 不同图表该使用什么名称进行引用
1. 个人认为不论是安装方式，还是使用方式，chartjs 的文档都写的比较清晰
2. 但唯独不同图表类型的引用名称，真的很难以找到
3. 于是翻阅源码查找到如下对应名称

![](http://asing1elife.com/sources/images/8ECBC6FD-5F85-48B5-BF3C-0A9253523E7D.png)

## Radar 雷达图的使用
### 目录结构
1. 使用如下目录结构是因为图表存在多种类型，每种都需要单独引用，所以应该单独创建一个初始化组件

![](http://asing1elife.com/sources/images/0E2E10E4-9BD2-40F7-AC24-C58DE852864F.png)

### 组件内容
1. 根据官方文档中的实现方式，在引用组件时，需要先将组件进行一次实现，之后再到具体的 DOM 结构中引用这个实现

```js
<script>
  import { Radar, mixins } from 'vue-chartjs'

  const {reactiveProp} = mixins

  export default {
    name: 'inRadar',
    extends: Radar,
    mixins: [reactiveProp],
    props: ['options'],
    mounted() {
      this.renderChart(this.chartData, this.options)
    }
  }
</script>
```

### 组件引用
1. 按照如下引用方式，一开始很容易会认为在初始化数据中除了 `datasets[0].data` 是动态数据，其他都是静态数据，可以放置到一开始的变量声明中指定
2. 但其实不行，整个图表的初始化 都需要在 DOM 元素加载之后才能进行，所以必须将所有指定的规则一起初始化
3. 对于图表大小的控制，最简单的方式还是将图表通过父容器包裹，然后控制父容器的大小

```html
<div class="chart-item">
  <in-radar :chart-data="radarData1"></in-radar>
</div>
```
```js
<script>
	import inRadar from 'components/in-chart/in-radar'
	
	export default {
		data() {
			return {
				radarData1: {}
			}
		},
		mounted() {
			this._initChar()
		},
		methods: {
			_initChar() {
				this.radarData1 = {
  				labels: ['抗压力', '精力', '适应力', '表达力', '逻辑力'],
  				datasets: [{
    					label: '',
    					borderColor: 'rgba(72,207,173,0.9)',
    					backgroundColor: 'rgba(72,207,173,0.5)',
    					data: currentFeature.radarData.one
  				}]
				}
			}
		},
		components: {
 			inRadar
		}
	}
</script>
```