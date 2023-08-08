---
title: 走进React——React的样子
date: 2019-08-13
tags: 
    - React
    - Javascript
summary: React是一个可以快速构建响应式界面的UI库，基于组件化开发，可以让你的数据与界面很大程度的解耦，让你能只专注于数据，而不用去操心界面的展示。
---

`React`是一个可以快速构建响应式界面的**UI**库，基于组件化开发，可以让你的数据与界面很大程度的解耦，让你能只专注于数据，而不用去操心界面的展示。同时他也可以通过不同的渲染器来渲染你的react范式的代码，譬如`react-native`，他可以让用"react范式"写出原生应用，运行在各种移动设备中。


### react的样子
其实react代码的样子很纯粹，纯粹到让你惊呼，这不就是`html`+`js`吗?是的，没有隔壁`vue`家的模板和众多的api，熟悉一下`jsx`,知道组件的生命周期钩子，你就可以用上手写出一下小应用了
一般来说，`react`组件经常使用jsx来编写，他的样子常常像下面这样：

```jsx
// jsx
// 思考题，这里好像没用到“React”，那么可以不用引这个库了吗？只用“Component”行不行？
import React, {Component} from 'react'
import ReactDOM from 'react-dom'

default class Test extends Component{
  render(){
    return (
      <div>{this.props.test}</div>
		)
	}
}

ReactDOM.render(
  <Test test="一串字符" />,
	document.querySelector('#root')
);
```

<!-- more -->

其实，这还不是`react`真正的样子，`jsx`通过[babal转义工具](https://babeljs.io/repl/#?presets=react&code_lz=MYewdgzgLgBApgGzgWzmWBeGAeAFgRgD4AJRBEAGhgHcQAnBAEwEJsB6AwgbgChRJY_KAEMAlmDh0YWRiGABXVOgB0AczhQAokiVQAQgE8AkowAUAcjogQUcwEpeAJTjDgUACIB5ALLK6aRklTRBQ0KCohMQk6Bx4gA)转义为js，真正的样子是这样的：

```js
// js
import React, {Component} from 'react'
import ReactDOM from 'react-dom'

class Test extends Component{
	render(){
		// 现在知道为什么每次都要引入一个在jsx中都好像不会用到的React了吧
		return React.createElement(
			"div",
			this.props.test
		);
	}
}

ReactDOM.render(
	React.createElement(Test, {test: '一串字符'}),
	document.querySelector('#root')
)
```

### state和props
组件内部的状态保存在`state`中，而外部传入的数据存放在`props`中，配合组件中的`state`和`props`可以构建出丰富多样的界面。
`state`和`props`会在进一步的文章中捎带说下
```jsx
class ShowText extends React.Component{
	constructor(){
		super();
		this.state = {
			list: [
				'bar',
				'foo',
				'baz'
			]
		}
	}
	render(){
		return (
			<div>
				<h3>{this.props.title}</h3>
				<ul>
					{
						this.state.list.map(msg => (<li key={msg}>{msg}</li>))
					}
				</ul>
			</div>
		)
	}
}

ReactDOM.render(
	<ShowText title="展示列表"/>,
	document.querySelector('#root')
);
```
界面如下：

![](http://cdn.liwuhou.cn/blog/20200306224259.png)

### 事件监听
同样的`jsx`支持监听事件，及使用回调函数，需要注意的是：React事件的命名采用销驼峰形式，而不是像html一样使用纯小写，使用jsx语法时，需要传入一个函数作为时间处理函数，而不是一个字符串

```jsx
/**
 * 只有bind了，this才会实现react组件的实例中，才可以使用state、setState等api
 * 有四种bind方法
 */
class Test extends Component{
	constructor(){
		super();
		this.state = {}
		// 1、在constructor中bind方法
		this.handleClickMethod1 = this.handleClickMethod1.bind(this);
	}
	render(){
		return (
			<div>
				{/* 1 */}
				<button onClick={this.handleClickMethod1}>按钮1</button>
				{/* 2、绑定回调函数的时候bind一下 */}
				<button onClick={this.handleClickMethod2.bind(this, payload)}>按钮2</button>
				{/* 3、绑定一个箭头函数 */}
				<button onClick={(event) => this.handleClickMethod3(event, payload)}>按钮3</button>
				{/* 4 */}
				<button onClick={this.handleClickMethod4}>按钮4</button>
			</div>
		)
	}
	handleClickMethod1(event){
		console.log('method1');
	}
	handleClickMethod2(payload, event){ // event在所有参数之后
		console.log('method2');
	}
	handleClickMethod3(event, payload){// 参数按绑定传递的顺序
		console.log('method3');
	}
	// 4、用箭头函数，将作用域指向实例
	handleClickMethod4 = (event) => {
		console.log('method4');
	}
}
```
这四种bind方式各有千秋，我会在后续的走进React系列中详聊

### 生命周期
每个组件从创建到挂载都会有一系列的生命周期钩子被触发，这些钩子函数就给了我们在这个时机运行一些代码的机会。
比如这个`conponentWillMount`可以在组件即将被挂载到页面的时候执行，值得一提的是，这些生命周期钩子都是自动绑定实例的， 你不用担心`this`的指向问题

```jsx
class Test extends Component{
	render(){
		return <div>hello world!</div>
	}
	componentWillMount(){
		console.log('will mount');
	}
}
```
在后续的文章中，将会更深入的谈谈组件中的生命周期

### 结语
一年没更新了，我已经无颜再见江东父老了，现在开个新坑，浅浅地说下react，也算自己对知识点和笔记的整理。


> 每次花个十分钟，懂一个React知识点，走得虽慢，只要坚持走下去，足以致千里。

关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)
