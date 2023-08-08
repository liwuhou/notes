---
title: 走进React——jsx
date: 2019-08-14
tags: 
    - React
    - Javascript
summary: 用React开发时，最具标识性的，估计就是jsx语法了。它并不是HTML，也不是字符串，官方称之为——一个javascript的语法拓展。
--- 

用React开发时，最具标识性的，估计就是jsx语法了。它并不是HTML，也不是字符串，官方称之为——一个javascript的语法拓展。在用react开发中，个人感觉jsx能极大地提升开发者的开发体验。在传统的开发中，都提倡将视图与逻辑文件分离，而React则是通过一个称之为“组件”的东东来存放视图和逻辑，将业务拆分为一个个的小模块，更多组件的概念在后续的文章中也会放出来跟大家探讨。


### 开始前先说几句
本次所有示例的代码的html结构都如下
```html
<div id="root"></div>
```
ok，交代完毕，开始步入正题。
<!-- more -->

### jsx具体是什么
```jsx
const lastName = 'baz';
// 声明一个组件
const Bar = ({name}) => <div>hello {name}</div>

// <Bar/>可以存放到变量中，还可以引用其它变量
const foo = <Bar name={lastName}/>

ReactDOM.render(foo, document.querySelector('#root'));
```
透过上面的代码，发现jsx是一种类似是**js表达式**的东东，可以赋值给变量。其实jsx元素就是属于js表达式，通过[babal转义工具](https://babeljs.io/repl/#?presets=react&code_lz=MYewdgzgLgBApgGzgWzmWBeGAeAFgRgD4AJRBEAGhgHcQAnBAEwEJsB6AwgbgChRJY_KAEMAlmDh0YWRiGABXVOgB0AczhQAokiVQAQgE8AkowAUAcjogQUcwEpeAJTjDgUACIB5ALLK6aRklTRBQ0KCohMQk6Bx4gA)转义揭开它的面纱后，你会发现这丫就是js：

```javascript
const lastName = 'baz';

// 这个组件在react中称之为函数式组件，更多声明组件的方式会在后续文章中讨论
const Bar = function(props){
    const {name} = props;
    return React.createElement(
        "div", 
        null, 
        "hello ", 
        name
    );
}

const foo = React.createElement(Bar, {
  name: lastName
});

ReactDOM.render(foo, document.querySelector('#root'));
```
ok，既然是表达式，那么就可以更深挖一下，看看它到底想要“表达”什么:

```jsx
const foo = <div>hello world!</div>;
const type = typeof foo;

console.log('mkLog: type', type); // 'object'

```
这些对象就是通过`React.createElement()`方法生成的对象，在React中以某种方式表达的将在视图中渲染出来的元素，称之为“**React元素**”。把它打印出来，大概长这样

```js
Object {
    $$typeof: Symbol(react.element),
    _owner: null,
    _store: {validated: false},
    key: null,
    props: {
        children: 'hello world!'
    },
    ref: null,
    type: 'div'
    ...
}
```
到此为止，我们已经知道了，jsx的本质还是js，React通过一种巧妙的语法，让数据和视图以一种恰到好处的松耦度结合起来，那些看起来很像xml的元素，本质上是js对象，React内部能解析它，并将它渲染成我们在界面上看到的视图。

### 在jsx中使用嵌入值或者变量
因为本质上React是一个js对象，所以可以接受其他值或者变量，甚至是表达式

```jsx
const lastName = 'baz';
const Bar = ({foo}) => <h3>show {foo}</h3>
const toUpperCase = (str) => String(str).toUpperCase();

const app = (
    <div>
        {/* 传入字符串 */}
        <Bar foo="foo"/>
        {/* 传入数字 */}
        <Bar foo={123}/>
        {/* 传入布尔值 */}
        <Bar foo={true}/>
        {/* 传入变量 */}
        <Bar foo={lastName}/>
        {/* 传入表达式 */}
        <Bar foo={'FOO'.toLowerCase()}/>
        <Bar foo={toUpperCase('bar')}/>

        <em>这里因为时间戳不同，可能输出的结果会跟我不一样↓</em>
        <Bar foo={Date.now() % 2 === 0 ? 'event' : 'odd'}/>
    </div>
);

ReactDOM.render(foo, document.querySelector('#root'));

```
![](http://cdn.liwuhou.cn/blog/20200306224327.png)

当传入的值为字符串时，直接用引号括起来传入，当涉及到js变量或者表达式的时候，就用花括号(`{}`)包裹起来。这里还有一些特殊的，如果是一个对象的话，就多嵌套一层花括号，这其实也很好理解，因为你传入了一个对象嘛，更改一个元素的style也是同理的：

```jsx

const Greet = (props) => (
    // 将style传递给元素
    <div style={{...props.style}}>
        hello {props.person.name}
    </div>
);
const person = {
    name: 'william'
}

const app = (
    <div>
        <Greet person={person}/>
        // 这里style中的css属性需要转成驼峰
        <Greet 
            person={{name: 'william'}} 
            style={{color: 'red', fontWeight: 'bold'}} 
        />
    </div>
)
//  WebkitTransition: 'all', msTransition: 'all'
ReactDOM.render(foo, document.querySelector('#root'));
```

**这里有个好玩的事**，如果你需要兼容一些旧式的浏览器，就需要在属性中加入前缀，譬如css3中的`transition`：

```css
.test{
    -webkit-transition: all;
       -moz-transition: all;
        -ms-transition: all;
         -o-transition: all;
            transition: all;
}
```
在react中用到这些前缀的时候，前面的小短线(`-`)可以不用写，但是浏览器前缀的开头字母需要大写（***除了ie***），也就是大驼峰

```jsx
<div style={{
    WebkitTransition: 'all',
    MozTransition: 'all',
    OTransition: 'all',
    msTransition: 'all'
}}>
    hello world!
</div>
```
是的，你没有看错，所有的浏览器的前缀开头字母都需要大写，除了ie的是小写`ms`，这点令人诧异，思来想去，可能是React的开发人员认为ie不是浏览器吧（干得漂亮！）


### jsx中的特定属性
因为html中是对大小写不敏感的，而本质上是js的jsx对大小写敏感而且分隔短线(`-`)会跟运算符减号(`-`)混淆，所以严格要求使用大驼峰输入，例如控制元素被键盘聚焦的`tabindex`属性，在jsx中就必须写为`tabIndex`

```jsx
<div tabIndex="1"></div>
```

另外，还有一些因为在js中是关键字，所以另外开辟了一些属性来避免歧义

```jsx
{/* class => className */}
<span className="test">请假装阅读以下条例并勾选同意</span>
<div>
    <input type="checkbox" id="name"/>
    {/* for => htmlFor */}
    <label htmlFor="name">用户条例</label>
</div>
```

![](http://cdn.liwuhou.cn/blog/20200306224344.png)

还有监听各种事件，这里需要使用'on' + 小驼峰的事件名来监听时间

```jsx
// html的onkeyup、onchange和onclick事件，在jsx中使用小驼峰
<input onKeyUp={handleKeyUpAddTodo} onChange={handleChangeSetInput} value={todoValue}/>
<button onClick={handleClickAddTodo}>添加todo项</button>
```
值得一提的是，react元素中的`onChange`事件，跟原生html的`onchange`事件略有差异，使用起来，有点类似原生的`oninput`事件，并且React依靠了该事件来实时地处理用户输入，这里先买个关子，后面会有涉及。

### 关于放置xss攻击和危险的innerHTML属性
React在渲染DOM的所有输入内容之前，默认都是先转义再输出的，这样就可以有效避免一些别有用心的的[xss攻击](https://baike.baidu.com/item/XSS%E6%94%BB%E5%87%BB),但是如果你执意需要用到输入的某些片段，React也提供了一个api属性，让你如愿，友情提示一下，这里使用的时候，结合使用环境做好防xss攻击的措施：

```jsx
const innerHTML_text = '<span style="color:red">Hello World!</span>';

// 往dangerouslySetInnerHTML属性里传入一个带有__html属性的对象就可以了
<div dangerouslySetInnerHTML={{__html: innerHTML_text}}></div>
```
为什么要搞的这么麻烦呢，这个拗口的api叫`dangerouslySetInnerHTML`就算了，还要往里传入一个key为`__html`的对象。
这里可能是React的开发者认为，使用代码直接设置HTML潜在的风险太大了，所以他们把事情搞得复杂点，你就不会第一时间想着这么玩了（手动滑稽）。

### 结语
今天简单的讲了下React中的jsx语法，这是个很有趣也很吊的js语法拓展，我个人感觉，用react开发，jsx是最能提升开发者的幸福度。这也是react明显能区分与`vue`和`angular`的标识。因为后两者常常用的是他们的模板字符串来进行开发（当然也可以用jsx），简单用列表渲染的需求来比较下`vue`的模板字符串和`react`的jsx

```vue
// vue单文件
<template>
    <div>
        <ul>
            <li 
                v-for="(item, idx) in list" 
                :key="item.id"
            >
                {{item.msg}}
    		</li>
        </ul>
    </div>
</template>
<script>
export default{
    name: 'show-list',
    data(){
        return {
            list: [
                {id: 1, msg: 'foo'},
                {id: 2, msg: 'bar'}
                {id: 3, msg: 'baz'}
            ]
        }
    }
}
</script>
```





```jsx
// jsx
import React, {Component} from 'react';

export default class ShowList extends Component{
    constructor(){
        super();
        this.state = {
            list: [
                {id: 1, msg: 'foo'},
                {id: 2, msg: 'bar'}
                {id: 3, msg: 'baz'}
            ]
        }
    }
    render(){
        const list = this.state.list.map(item => {
            return (
                <li key={item.id}>{item.msg}</li>
            );
        });
        return (
            <div>
                {/* 有两种方式 */}
                <ul className="list_1">
                    {
                        this.state.list.map(item => (
                            <li key={item.id}>{item.msg}</li>
                        ))
                    }
                </ul>
                <ul className="list_2">
                    {list}
                </ul>
            </div>
        )
    }
}
```
一个同样的需求，通过两种不同的方式表达出来，虽是殊途同归，但是某些时候两者实现需求的思路是截然相反的，模板渲染，这里不做评判，改天有时间，尝试着多方位的比较这两者的差异吧。

### 最后说几句
昨天发了一篇文章，被我老妹实力嘲讽（手动捂脸），真的以后要多更新下了，为了自己巩固巩固下所学的东西也是好的。

![](http://cdn.liwuhou.cn/blog/20200306224411.png)

> 每次花个十分钟，懂一个前端知识点，走得慢，但坚持走下去，足以致千里。

关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)
