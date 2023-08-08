---
title: 十分钟，带你了解React 16.8的useState和useEffect
date: 2019-08-27
tags: 
    - React
    - Javascript
    - TenMinutes
summary: Hooks是React 16.8新增的特性，并且他是向后兼容的，facebook也坦言没有计划会在React中移除class组件。你完全可以不使用hooks，一直使用class组件。
---

`Hooks`是**React 16.8**新增的特性，并且他是向后兼容的，**facebook**也坦言没有计划会在**React**中移除class组件。你完全可以不使用`hooks`，一直使用class组件。


### 概念
`React hooks`允许你在非`class`组件中使用`state`和`React`其他的特性：`props`， `context`，`refs` 以及`生命周期`。
`Hook`本质上是一些函数，可以在函数组件中“钩入”其他**React**的特性，使得你不用使用`class`也可以使用React。*值得注意的是，`hooks`是不能在`class`组件中使用的。*

<!-- more -->
### State Hook
这里贴一下官网文档中用`hooks`实现一个计数器的代码

```jsx
// Hooks
import React, {useState} from 'react'

function Test(){
    // 这里生命了一个“count”的state，和一个可以修改count的方法
    const [count, setCount] = useState(0); // 这里传入count这个state的默认值

    return (
        <div>
            <p>You clicked {count} times.</p>
            <button onClick={setCount(count + 1)}>Add Count +1</button>
        </div>
    )
}
```

`useState`就是一个`hooks`，他**接收一个初始化`state`的参数**，并**返回一个当前state的值和一个可以设置state的方法**。借用es6的数组解构语法，我们可以很优雅的用变量名来接收他们。就像上面的`count`和`setCount`。在此期间，我们没有声明任何class组件 ，也不必去写`constructor`这些方法和声明其他的`state`。

当然，在这里你也可以一直使用class，而不去拥抱`hooks`，所以可能写出来的代码，看起来会像这样

```jsx
// class
import React, {Component} from 'react'

class Test extends Component{
    // 这里要初始化state
    constructor(){
        super();
        this.state = {
            count: 0
        }
        // 也别忘记了要把方法绑定到实例里。
        this.handleClickAddCount = this.handleClickAddCount.bind(this);
    }
    handleClickAddCount(){
        // 这里要用setState返回state的更改
        this.setState(({count}) => ({
            count: count + 1
        }))
    }
    render(){
        return (
            <div>
                <p>You clicked {this.state.count} times.</p>
                <button onClick={this.handleClickAddCount}>Add Count +1</button>
            </div>
        )
    }
}
```
上面的class组件，只不过是因为需要有一个`count`的`state`数据，就大费周章地写下了这么多代码，想到这里，你是不是对`hooks`有一点好感了？

### Effect Hooks
在React中，仅仅通过`state`状态的更改来操作DOM，可能还不是我们想要的。我们可能需要做更多“副作用”的东西，譬如在某个`state`变更的时候，同步更改页面的`title`。

贴下`useEffect`这个hooks的代码

```jsx
function Test(){
	const [count, setCount] = useState(0);
	// 相当于 componentDidMount 和 componentDidUpdate:
	useEffect(() => {
		// 使用浏览器的 API 更新页面标题
		document.title = `You clicked ${count} times`;
		console.log('mkLog: count', count);
	});

	return (
		<div>
			<p>You clicked {count} times</p>
			<button onClick={() => setCount(count + 1)}>Click me</button>
		</div>
	)
}
```
这样组件在挂载完毕和`count`改变的时候都会对网站的标题进行设置了，如果用以前的class方式声明组件的话，就发现一些冗余

```jsx
class Test(){
    constructor(){
        super();
        this.state = {
            count: 0,
        }
        this.handleClickAddCount = this.handleClickAddCount.bind(this);
    }
    componentDidMount(){
        const {count} = this.state;
		document.title = `You clicked ${count} times`;
    }
    componentDidUpdate(){
        const {count} = this.state;
		document.title = `You clicked ${count} times`;
    }
    // 两个生命周期中执行了同样的代码
    handleClickAddCount(){
        this.setState(({count}) => ({
            count: count + 1
        }))
    }
    render(){
        return (
            <div>
                <p>You clicked {this.state.count} times.</p>
                <button onClick={this.handleClickAddCount}>Add Count +1</button>
            </div>
        )
    }
}
```
稍有react开发经验的同学应该对这个场景很熟悉了，`componentDidMount`和`componentDidUpdate`里面运行的是同样的代码，使用这个`hooks`可以很明显的减少代码量和复杂度。

除此之外，`useEffect`还能实现`componentWillUnmount`这个生命周期，只是这里有一点区别，放一个官网文档的例子来帮助理解一下：
用class实现一个显示好友是否在线的组件。
```jsx
// ChatAPI 假设是一个（取消）订阅好友的api方法
// 组件挂在的时候，订阅好友的状态
componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}
// 当组件卸载的时候，取消订阅组件的订阅
componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}
```

但是这样就会有一个问题，就是当组件挂载之后，已经显示在屏幕上了，props中的friend已经发生改变了，这时组件展示的还是原来的好友的状态，而且还会因为卸载时取消了错误的好友ID导致内存泄露或崩溃的问题。
这就使得我们不得不在这个class组件中添加`componentDidUpdate`来解决问题

```jsx
// 新加一个props的friend更新时的生命周期钩子
componentDidUpdate(prevProps) {
    if(prevProps.friend.id === this.props.friend.id) return;
    // 取消订阅之前的 friend.id
    ChatAPI.unsubscribeFromFriendStatus(
        prevProps.friend.id,
        this.handleStatusChange
    );
    // 订阅新的 friend.id
    ChatAPI.subscribeToFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}

// 当组件卸载的时候，取消订阅组件的订阅
componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
        this.props.friend.id,
        this.handleStatusChange
    );
}
```
这个场景相信各位用React开发的同学也都会遇到，无形之中多了很多重复的冗余代码，下面展示下`useEffect`的下个特性，可以完美地解决上述的问题

```jsx
function FriendStatus(props) {
    // 使用
    useEffect(() => {
        // 这里会在原来componentDidMount和ComponentDidupdate时调用
        ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange); // ⑴
        // 这里会在componentDidUpdate和componentWillUnmount时调用
        return () => { // ⑵
            ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
        };
    });
}
```
这里有三个需要注意的地方，第一点在组件挂载的时候，仅会执行⑴处代码，而不会执行⑵处的回调函数；第二点，在组件`props`改变的时候，也就是发生类似class组件中的`componentDidUpdate`的时候，⑵的回调函数会比⑴早一步执行；第三点，组件卸载的时候，也就是等同于class中的`componentWillUnmound`的时候，`useEffect`只会调用⑵的回调函数而不会调用⑴处的代码。

另外`useEffect`的第二个可以传递一个数组进去，表示数组中的state变量发生变化时才调用⑵的函数.
```jsx
function Test(){
	const [count, setCount] = useState(0);

	useEffect(() => {
		document.title = `You clicked ${count} times`;
	}, [count]); // Mark: 传递count解绑

    // rest code...
}
```

---

留点家庭作业，尝试着自己设计一个例子来验证上述的结论



最好自己思考过来再来参考下面的代码：
```jsx
import React, {useState, useEffect} from 'react'
import ReactDOM from 'react-dom'

function Test(){
	const [count, setCount] = useState(0);
    useEffect(() => {
        console.log('mount/change');
        
        return () => {
            console.log('done');
        }
	})
	return (
		<div>
			<button onClick={() => {setCount(count + 1)}}>{count}</button>
		</div>
	)
}

function Wrap(){
    // 这里复习下useState
    const [isShow, changeShow] = useState(true);

    return (
        <div>
            {isShow && <Test/>}
            <button onClick={() => changeShow(!isShow)}>{isShow ? 'hide' : 'show'}</button>
        </div>
    )
}

ReactDom.render(
    <Wrap/>,
    document.querySelectot('#root')
)
```

> 每次花个十分钟，懂一个React知识点，走得虽慢，只要坚持走下去，足以致千里。

关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)
