---
title: 大话 JavaScript 继承
date: 2021-01-17
tags: Javascript
summary: 你知道 JavaScript 中有几种继承方式吗？ES6 的 extends 使用的是何种方式？
---


在面向对象编程中，为了能更好的复用以前的开发代码，缩短开发的周期，提高开发的效率，继承是一种很好的方法。它可以让我们使用继承的方式，去使用原有对象的一些方法或者属性。在 JavaScript 这么灵活的语言之中，常见的继承方式有六种，现在我们就来一个一个的过一遍。

<!-- more -->
### 原型链继承

原型链是比较常见的一种继承方式之一，其中涉及到构造函数、原型和实例，三者之间存在一定的关系，如下图。

![js%20%E4%B8%AD%E7%9A%84%E5%85%AD%E7%A7%8D%E7%BB%A7%E6%89%BF%2061269e36ede9486ba83e0ebbecb5f6ae/Untitled.png](js%20%E4%B8%AD%E7%9A%84%E5%85%AD%E7%A7%8D%E7%BB%A7%E6%89%BF%2061269e36ede9486ba83e0ebbecb5f6ae/Untitled.png)

每一个构造函数都会有一个原型对象(构造函数的 `prototype`) ，原型对象又包含一个指向构造函数的指针(`constructor`)，实例也有一个指向原型的指针(`__proto__`)。

实现原型链继承的代码很简单

```jsx
function Person(){
	this.name = 'parent'
	this.play = [1, 2, 3]
}
function Child() {
	this.type = 'children'
}
Child.prototype = new Person() // *
console.log(new Child())
```

核心代码就是 `Child.prototype = new Person()` 将 `Child` 的原型指针指向 `Person` 的实例。这样 `Child` 的实例就可以访问到 `Person` 的构造函数的属性和方法。

```jsx
var child1 = new Child()
var child2 = new Child()

console.log(child1.name) // parent
child2.play.push('coding')
console.log(child1.play) // [1, 2, 3, 'coding'] *
```

但是，原型链继承的一个缺点就在于两个实例(`child1` 和 `child2` )访问的是同一个原型对象，所以，共享的是同一份数据，这就导致数据可能被篡改的风险。

为了解决这一问题，让我们接着介绍第二种继承方式。

### 构造函数继承（借用 call）

上面原型上的属性共享的原因在于，所有的 `Child` 实例都访问了同一个原型对象，那么我每次 `new` 的一个 `Child` 的时候，都让它拥有一个全新原型对象不就好了嘛，所以这种借用构造函数的继承方法应运而生。

```jsx
function Parent(){
	this.name = 'parent'
	this.play = ['play game', 'coding']
}

// 这里定义一个父类原型上的方法，埋个伏笔 #
Parent.prototype.getName = function(){
	return this.name
}

function Child(){
	Parent.call(this) // call 借用 Parent 的构造函数，这样就能“偷取”定义在父类构造函数的属性和方法
	this.type = 'child' // 子类的构造函数的属性
}

let child = new Child()
console.log(child.name) // parent
console.log(child.getName()) // ?
```

这里因为子类的构造函数 `Child` 通过 `call` 方法，“借用了父类构造函数的代码”（我杜撰的，为了好记，姑且这么说吧），所以他的原型上也有了 `Parent` 定义的 `name` 和 `play` 属性。这个说是继承，其实就是子类照着父类，模仿着声明了这些个变量或方法。虽然子类拥有了父类的属性和方法了，但是他是访问不到父类原型上的属性和方法的，在 `#` 伏笔处定义的父类原型上的方法，在 `？` 处调用是会报错的，报的是 `child.getName is not a function` 。

到这里就有点好玩了，刚才说的第一种(原型链继承)，是可以访问父类原型上的属性和方法，但是父构造函数上的属性和方法会被子类共享；第二种(构造函数继承)，则是可以拥有独立的父构造函数的属性和方法，但是访问不了父类原型对象上的属性和方法。

那我们何不把二者的优点组合一下，各取所长甚好？名字我都想好了，就叫组合继承(不闹，这种方式真的叫这名)。

### 组合继承(原型链继承+构造函数继承)

又“偷”你属性，又继承你爸爸(大雾)

```jsx
function Parent(){
	this.name = 'parent'
	this.play = ['play game', 'coding']
}

Parent.prototype.getNmae = function(){
	return this.name
}

function Child(){
	Parent.call(this) // 借用构造函数 #2
	this.type = 'child'
}

Child.prototype = new Parent(); // 子构造函数的原型指向父类实例 #1

// 然后这里要想原型上的构造器指针指回自己的构造函数
Child.prototype.constructor = Child

const child1 = new Child()
const child2 = new Child()
child2.play.push('daydream')
console.log(child1.play === child2.play) // 不会互相影响
child1.getName()
child2.getName() // 正常输出
```

可以看到，困扰我们的两大问题现在解决了，子类不仅自己独享一份父构造函数上的属性，还能访问父类原型上的属性和方法。但是，随之而来的问题是——父构造函数 `Parent` 被调用的次数太多了。可以看见，从 `Child` 继承(#1)和通过 `call` 调用(#2)，这里 `Parent` 总共执行了两次，多执行一次构造函数就意味着多一份的性能开销。假如 `Parent` 的构造函数代码量很大，每一次的继承都是一笔不小的性能开销。那有什么办法可以避免多执行一次调用呢？

😏 

方法肯定是有的，有个终极大招可以解决，这个大招后面还作为es6 中的 extends 语法糖。那是什么呢，这里先不说，卖个关子，继续说点大招的前置知识。

上面几种继承方法，都或多或少的存在着这样的优点或者那样的缺点，看来在构造函数上下功夫或许已经找不到好的解决方式了。让我们先另辟蹊径，抛开对象的构造函数，如果我单单只是想继承一个对象(实例)的属性或者方法，要咋整呢？

这里就要隆重介绍下 ES5 中的 `Object.create` 方法了，这个方法接收两个参数： `proto` 必传的新创建对象的原型对象(这个很 cool)和 `propertiesObject`  可选的对对象中属性类型的描述，这个参数也就是 `Object.getOwnPropertyDescriptors` 返回的结果。不熟悉的朋友可以去 [mdn 看看 `create` 方法的详细描述和用例]([https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)) 

[Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

先通过一段代码，看看普普通通的对象是怎么被继承的

### 原型式继承

```jsx
const parent = {
	name: 'william'
	play: ['coding', 'play game']
	getName(){
		return this.name
	}
}

const child1 = Object.create(parent)
const child2 = Object.create(parent)

child2.name = 'skye'
console.log(child1.name) // william
console.log(child2.name) // skye
child2.play.push('daydream')
console.log(child1.play) // ['coding', 'play game', 'daydream']
console.log(child2.play) // ['coding', 'play game', 'daydream']
```

上述的代码也可以看到，使用了 `Object.create` 可以继承普通对象(实例)的属性和方法，了解了 `Object.create` 方法之后，可以发现其实也不是什么黑魔法，就是生成了一个原型的 `__proto__` 指针指向传入对象的对象而已。然后生成的对象可以通过原型链来访问原型对象的属性和方法。

```jsx
console.log(child1.__proto__ === child2.__proto__ == parent) // true
```

经过我们刚刚说过的第一种的继承方法——原型链继承，实例使用同一个原型对象，有可能会有篡改原型数据的风险。原型式继承跟原型链异曲同工，所以也难免有这样的弊端。

接着看一种在原型式继承基础上增强了一些功能的继承方法

### 寄生式继承

这种继承方式，虽然优缺点跟原型式继承一样，只是在原型式继承得到的对象基础上，通过工厂模式添加了一些方法和属性

看一下代码就知道了

```jsx
const parent = {
	name: 'william',
	play: ['coding', 'reading', 'play game'],
	getName(){
		return this.name
	}
}

function ChildFactory(origin){
	let child = Object.create(origin)
	child.getPlay = function(){
		return this.play
	}
	
	return child
}

let child = ChildFactory(parent)

console.log(child.getName())
console.log(child.getPlay())
```

这种继承方式也没啥好说的，算得上是原型式继承的一种增强方式吧。

到这里，终极大招的前置知识都已经讲完了，是时候解锁终极大招了。在上面第三种组合式继承中，因为调用了两次父类构造函数的方法，有一定的性能开销损耗。这里开始分析一下。首先第一次开销在于继承的时候， `call` 方法的调用，第二次则是将子构造函数的原型指向父构造函数式的调用。根据刚刚所学的原型式继承，我们可以不用通过 `new Parent()` 来让父构造函数生成实例，可以直接用 `Object.create` 来继承父类的原型，这样就省下一次调用父构造函数的开销了，具体实现看下方代码

```jsx
function Parent(){
	this.name = 'william',
	this.play = ['coding', 'reading']
}

Parent.prototype.getName = function(){
	return this.name
}

function Child(){
	// 组合式继承，利用 call 来继承父构造函数属性
	Parent.call(this)
	this.type = 'child'
}

// 利用 Object.create 来继承原型，就可以省下一次调用父构造函数的开销
Child.prototype = Object.create(Parent.protype)
Child.prototype.constroctor = Child
Child.prototype.getPlay = function() {
	return this.name
}

const child = Child()
console.log(child.name) // william
console.log(child.getName()) // william
console.log(child.getPlay()) // ['coding', 'reading']
```

很明显，寄生组合继承解决了上述五种方案中的痛点，算是比较好地实现了我们想要的继承效果。当然，缺点也还是有的，就是篇幅太长了，搞个继承要写这么多代码，各种 `prototype` ，还要手动将原型上的构造指针指回构造函数……

且慢，时代在发展，js 的语法规范也在趋于完备。就在 es6 之后，不仅新增了类 `class` 的关键字，有了定义类的方式，不用写又臭又长还割裂的构造函数，连寄生组合式继承也有对应的语法糖了，他就是 `extends` 。先来一睹 es6 面向对象继承这一块的风采

```jsx
// es5 这种割裂的声明“类”和其原型上的属性方法
function Parent(name){ // 这里相当构造器
	this.name = name
}

// 还没完，原型上的属性，还要在另一个地方写
Parent.prototype = {
	getName(){
		this.name
	},
	foo: 'bar'
}
// 这里手动再将原型对象上的构造器只会构造函数
Parent.prototype.constructor = Parent

// 然后继承采用寄生组合式... 又写一堆代码(略)

// 有了 es6 之后
class Parent{
	constructor(name) { // 构造器，入参其实构造所用的参数
		this.name = name
	}

	// 这里定义的都是原型上的方法
	getName(){
		return this.name
	}
}

// 子类继承也很方便
class Child extends Parent{
	constructor(name, age){
		// 这里如果子类中存在构造函数，就必须在使用 this 之前先调用 super()
		super(name) // 相当于借用父类的 constructor 跟构造函数式继承中的 call 继承方法类似
		this.age = age
	}
}

const child = new Child('william', 18)
child.getName() // william
```

### 总结

走完这一遭之后，发现其实 js 的这几种常用的继承方式不复杂，都是在原有方案的基础上，为了规避弊端采取的一些“升级”。自己也是到了这几天，系统的整理和学习了继承的这些方法后开始后知后觉。平日里的开发工作很少会涉及到这些，所以往往就忽视了对继承的系统性学习。但是前端中，一些优秀的库和设计模式，都大量使用到了继承等概念。自己想要在代码能力上有进一步的提升，在阅读源码或者学习设计模式之前，最好就先把这些基础的概念打牢为妙，步步为营稳扎稳打地提升自己的编程能力。