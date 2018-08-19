---
title: export和export default的区别
date: 2017-11-27
author: asing1elife
categories:
 - javascript
 - es6
tags:
 - javascript
 - es6
---
> `export` 本质上就是规定模块[js文件]的对外接口[属性或方法]  
> `export default` 则是在 `export` 的基础上，为规定模块提供一个默认的对外接口  

## export的使用
1. 直接输出

```javascript
export let words = 'hello world!!!'

export function output() {
	// ...
}
```
2. 先定义再输出 **推荐使用**
	* 需要注意的是，对于这种输出方式，不论最终 `export` 决定输出几个接口，都需要使用一堆大括号包裹

```javascript
let firstWords = 'hello'
let secondWords = 'world'
let thirdWords = '!!!'

function output() {
	// ...
}

export {firstWords, secondWords, thirdWords, output}
```

## export default的使用
1. `export default` 用于规定模块的默认对外接口
2. 很显然默认对外接口只能有一个，所以 `export default` 在同一个模块中只能出现一次
3. `export default` 除了不具备 `export` 所拥有的第二种输出方式以外，其在 `import` 方式上也和 `export` 存在一定区别
	* 从以下两种 `import` 方式即可显著看出 `export default` 的 `import` 方式不需要使用大括号包裹
	* 因为对于 `export default` 其输出的本来就只有一个接口，提供的是模块的默认接口，自然不需要使用大括号包裹
4. export的输出与import输入

```javascript
export function output() {
	// ...
}

import {output} from './example'
```
5. export default的输出与import输入

```javascript
export default function output() {
	// ...
}

import output from './example'
```