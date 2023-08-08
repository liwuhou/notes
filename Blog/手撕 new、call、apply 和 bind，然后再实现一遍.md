---
title: 手撕 new、call、apply、bind，然后再实现一遍
date: 2021-01-26
tags: Javascript
summary: 虽然在工作中， `call` 、 `apply` 和 、 `bind` 不算很常用，但是想必大家在面试中还是会经常碰到这类题目的。这篇小文章，通过简单又详实的描述，带大家更了解这四大天王...
---


虽然在工作中， `call` 、 `apply` 和 、 `bind` 不算很常用，但是想必大家在面试中还是会经常碰到这类题目的。同时，在阅读一些比较好的开源项目的时候，会发现里面会经常地使用上述 api 去复用原有的方法，达到节约内存和优化代码的效果。接下来，就让我带你简单又详细的手撕(剖析)和自己实现 `new` 、 `call` 、 `apply` 和 `bind`。

<!-- more -->
### new

经常用 new 来创建构造函数，很直观的就会觉得是一个从构造函数中生成实例的关键字。那么其实拆解出步骤，new 做了如下这些事

1. 生成一个新的实例对象
2. 让实例对象可以访问构造函数原型链上的属性
3. 将构造函数的作用域绑定在这个实例对象上
4. 执行构造函数的代码
5. 返回这个实例对象

值得注意的是，new 关键字期望返回一个引用类型的数据，如果我们在构造函数中 `return` 了某个引用类型，那么 new 就返回这个引用类型，如果我们强行的返回非引用类型的数据的话，那么返回的还是实例对象。

说了那么多，还是来看看如下代码

```js
function Person1(){
	this.name = 'william'
}
function Person2(){
	this.name = 'william'
	return (console.log('trigger') // undefined
}
function Person3(){
	this.name = 'william'
	return []
}

var p1 = new Person1() // 实例对象 {name: 'william'}
var p2 = new Person2() // 还是实例对象，虽然 return 触发了，也打印了 trigger
var p3 = new Person3() // []
```

ok，现在分析下 new 拆解之后的这些步骤，其中 1 和 5 应该是没有难度的，疑难点在于 2 和  3。

这里涉及到链接构造函数原型链上的属性和改变函数作用域。

首先，链接某个对象的原型链，应该能够想到使用 `Object.create` 这个方法，也就是上篇文章里面的原型式继承。

[js 中的六种继承](https://www.notion.so/js-9ed178c556694a0e9eb3e90b523d4988)

其次，在 js 中，若想改变一个函数的作用域，肯定也能想到 `call` 、 `apply` 和 `bind` 方法，这里我们就先用上他们，等会再一起好好说道说道。

> 因为js 里也不能重载操作符或者定义一个新的操作符，所以这里我们用函数来模拟

```js
function myNew(cto, ...args){
	const obj = {}
	obj.__proto__ = Object.create(cto.prototype) // 生产中不建议使用 __proto__
	
	return cto.apply(obj, args) // apply 可以改变一个函数的 this 指向
}
```

在 `Function` 对象中，拥有着 `apply` 、 `call` 和 `bind` 这三个方法，可以用来改变调用他们的函数的 this 指向，也就是改变函数的作用域。

```js
const fn = function(){}

fn.call(thisArg, params1, params2, ...)
fn.apply(thisArg, [params1, params2, ...])
fn.bind(thisArg, params1, params2, ...)
```

都是涉及到改变函数的作用域，不同之处在于， `bind` 是返回一个改变了 this 指向的函数，而 `call` 和 `apply` 是改变了函数的 this 指向后马上执行并返回结果。然后 `call` 和 `apply` 的传参形式不同， `call` 从第二个参数开始，都是给调用的函数(例子中的 `fn`)提供的参数，而 `apply` 第二个参数是数组，数组里面的每一项都是给调用函数的传参。

看个例子

```js
const wiliam = {
	name: 'william',
	greet(greet_word){
		return `${greet_word} ${this.name}`
	}
}

const abby = {
	name: 'abby'
	// abby 对象上是没有 greet 这个方法
}

console.log(william.getName('hello')) // hello william
console.log(william.getName.call(abby, 'hello')) // hello abby
console.log(william.getName.apply(abby, 'hello')) // hello abby
const greetToAbby = william.getName.bind(abby, 'hello')) // 返回的是一个改变了 this 的函数
console.log(greetToAbby()) // hello abby

```

这种改变函数指向的方法，在日常开发中也经常用到，可以节省很多的代码量，下面例举些常用的场景

**借用Object.prototype.toString来判断数据类型**

```js
function getType(obj){
	if (typeof obj !== 'object') return obj

	return Object.prototype.toString.call(obj).replace(/^$/, '$1')
}
```

通过 `Object.prototype.toString` 来判断传入的 `obj` 的字符串，从而确定数据类型

**解构数组传参**

```js
// 在没有 es6 的...解构之前
const arr = [1, 5, 7, 9, 11]
const min = Math.min.apply(null, arr) // 1
const max = Math.max.apply(null, arr) // 11
// or
Array.prototype.push.apply(arr, [0, 0, 0]
console.log(arr) // [1, 5, 7, 9, 11, 0, 0, 0]

// 现在推荐直接使用...
Math.min(...arr)
Math.max(...arr)
arr.push(...[0, 0, 0])
```

`React Component` 中改变因为中间变量(`onClick`)丢失了 this 的问题

```js
class Test extends React.Component{
	handleClick(){}

	render(){
		return (
			<div onClick={this.handleClick.bind(this)}></div>
		)
	}
}
```

**改造多参数函数**

 

```js
const add = (...args) => args.reduce((total, cur) => total += cur, 0)
console.log(add(1, 2, 3)) // 6

// 改造出一个给任何数加 10 的方法
const addTen = add.bind(null, 10)

console.log(addTen(1)) // 11
```

用得好也是可以简化很多代码和逻辑的

**类数组借用数组原型上的方法**

```js
const arrayLike = {
	0: 'william',
	1: 'abby',
	length: 2
}

// 借用 slice 改造成数组
const arr = Array.prototype.slice.call(arrayLink) // 返回一个数组 ['william', 'abby']

// 借用 push 添加个项
Array.prototype.push.call(arrayLike, 'skye')
console.log(arrayLike) // { '0': 'william', '1': 'abby', '2': 'skye', length: 3 }
```

### call 和 apply 的实现

 `call` 、 `apply` 和 `bind` 的结果是改变方法的 this 指向，然后在 js 对象中，对象方法的 this 指向都是指向自己的，那么我们就可以在这个地方上做点文章了。

`call` 和 `apply` 基本原理是相同的，只是传参的形式不同而已。

```js
Function.prototype.call = function(context, ...args){
	context = context || window // 默认指向 window
	context.__fn = this // 这里的 this 是调用 call 方法的函数自身
	// const res = eval('context.fn(...args)') // 不推荐使用，能不用 eval 就别用 eval
	const res = (new Function('return function(context, args){return context.__fn(...args)}'))()(
		context, args
	)
	delete context.__fn // 删除污染
	return res
}
```

实现就是这么简单，先通过 `context` 这个临时变量来指代上下文，然后通过将函数挂载到传入的 `context` 上，执行就得到我们想要的结果了。

这里有必要说一下，虽然使用了 `new Function` 来代替 `eval` ，三重嵌套函数看上去比上面的复杂很多，但是函数调用的性能开销其实很小，其实是能比 `eval` 快好几个数量级，也更安全。

点这查看MDN文档里，全局对象 (eval)[[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/eval](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/eval)] 和 (Function)[[https://developer.mozilla.org/zh-CN/docs/Glossary/Function](https://developer.mozilla.org/zh-CN/docs/Glossary/Function)]的详细信息。

### bind 的实现

bind 的实现其实也跟 `call` 基本一致，不同之处在于， `bind` 不需要马上执行，只是返回一个改变了作用域的函数即可

```js
Function.prototype.bind = function(context, ...args){
	const self = this;
	const fn = function(){
		self.apply(this instanceof self ? this : context, [...args, ...arguments]);
	}
	
	if(this.prototype){
		fn.prototype = Object.create(this.prototype)
	}

	return fn
}
```

这里的源码其实不难理解，首先通过包装一层函数，里面用我们上面说过的 `apply` 或者 `call` 改变了作用域的函数调用。

然后接着判断函数是否还有原型属性，然后使用 `Object.create` 来确保函数的原型能挂载到我们的  `fn` 上面。

接着返回 `fn` 。

### 总结

走完这一遭，是不是发现其实这些手写 `bind` 、 `call` 等都是纸老虎了。它其实更多的是对前端基础的考察，很多东西都只是简单概念的叠加而已。如果那一块的内容卡住了，可以先查阅下相关的资料，自己多阅读和敲打几遍实例，也可以在下方留言，跟大家一起讨论下。