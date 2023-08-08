---
title: Koa2+MongoDB+React撸个即时聊天web应用(上)
date: 2020-02-10
tags: 
    - NodeJs
    - React
    - MongoDB
summary: 学习了`Koa2`和`MongoDB`之后，突然就想着撸个实战项目出来看看，想来想去，还是搞个即时聊天应用出来玩玩吧。这篇是前端部分的编写，省略了一些样式的赘述，文章其实还是挺简短的...
---

学习了`Koa2`和`MongoDB`之后，突然就想着撸个实战项目出来看看，想来想去，还是搞个即时聊天应用出来玩玩吧。

项目的所有源码已经放到[Github](https://github.com/hugewilliam/chat_room/tree/simple_chat)
我也把最终的效果放到线上服务器了[摸鱼俱乐部聊天室](https://liwuhou.cn/chat)

前端部分我使用了`React`来搭建界面，因为工作用的都是`Vue`，感觉再不用用`React`就要生疏了……

这篇文章主要讲述前端部分代码的编写，这里我使用的是`create-react-app`脚手架来搭建项目。

<!-- more -->

### 搭建项目

用脚手架生成一个`chat`项目

```shell
# 如果没安装脚手架的就全局装下
$ yarn add global create-react-app

# 创建项目
$ yarn create react-app chat && cd chat

# 当然也可以用npm
$ npm install -g create-react-app
$ npm init react-app chat
```

完事后进入`chat`目录就可以看到以下的结构

```shell
.
├── README.md
├── package.json
├── node_modules/...
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
├── src
│   ├── App.css
│   ├── App.js
│   ├── App.test.js
│   ├── index.css
│   ├── index.js
│   ├── logo.svg
│   ├── serviceWorker.js
│   └── setupTests.js
├── yarn-error.log
└── yarn.lock
```

然后就是将脚手架生成的多余的文件给删掉，使目录结构变成这样

```shell
.
├── package.json
├── node_modules/...
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── robots.txt
├── README.md
├── src
│   ├── App.jsx
│   └── index.js
└── yarn.lock

```
将src下的`app.js`改为`app.jsx`(这样看起来更酷不是吗？)，然后对文件内容进行一些更改。
```jsx {4, 6}
import React from 'react';

function App() {
	// 这里先输出hello world跟编程世界打个招呼
	return (
		<h1>hello world</h1>
	);
}

export default App;
```

将index.js文件也修改下

```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(
	<App/>,
	document.querySelector('#root')
)
```

**这一部分的源代码可以在git仓库里面的`init`分支上查看。**


### 配置路由及安装axios

这个demo中，我路由管理用的是`react-router-dom`，异步请求使用的是`axios`库。
话不多说，先安装相关依赖

```shell {1}
$ yarn add react-router-dom axios 
```

由于这里只是一个很简单的小demo，所以路由并不多。其实也就两个，一个是初始进入时的注册和登录页(`/login`)，还有就是聊天界面(`/chat`)。
在`src`目录新建一个`router`目录，并在目录下放新建`index.js`文件，现在我们来编写应用里可能会用到的路由跳转。
大概撸出来，是这样的

```jsx

import React from 'react';
import {Route, Switch} from 'react-router-dom';
// 这里的页面组件还没有编辑，先占下位置
import Login from '../views/Login';
import Chat from '../views/Chat';

export default class Router extends React.Component{
	render(){
		return (
			<Switch>
				{/* 登录/注册 */}
				<Route exact path="/login" component={Login}/>
				{/* 聊天界面 */}
				<Route exact path="/chat" component={Chat}/>
			</Switch>
		)
	}
}

```

接着在`src`下新建`views`文件夹，往`Login`和`Chat`里随便写点东西。
最后往`App.jsx`中引入`Router`

```jsx {9, 10, 11}

// App.jsx
import React from 'react';
import {HashRouter} from 'react-router-dom';
import Router from './router';

export default function App(){
	return (
		<HashRouter>
			<Router/>
		</HashRouter>
	);
}

```

在终端中输入`yarn start`就可以启动服务，在浏览器中输入`localhost:3000/#/login`和`localhost:3000/#/chat`可以访问相应的组件了。
**这一部分的源代码可以在git仓库里面的`1-1`分支上查看。**

### eject和修改webpack配置

并且由于后续需要改一些`webpack`的配置，这里需要`yarn eject`一下，把脚手架隐藏了的`webpack`的配置都暴露出来。

```shell

$ yarn eject

```

执行之后发现多了个`config`和`scripts`目录，并且`package.json`文件也多了很多内容。详细的可以自行了解`create-react-app`里的`yarn eject`作用，我们先去改下`config`目录下的`webpack.config.js`配置。
> p.s. 对`webpack`不太了解的，可以先去看看我之前做的`webpack`配置的这篇文章——十分钟——[带你了解webpack的主要配置](https://liwuhou.cn/2020/01/07/%E5%8D%81%E5%88%86%E9%92%9F%E2%80%94%E2%80%94%E5%B8%A6%E4%BD%A0%E4%BA%86%E8%A7%A3webpack%E7%9A%84%E4%B8%BB%E8%A6%81%E9%85%8D%E7%BD%AE/)

这里加了两行`alias`(别名)配置，一个是让`webpack`将`@`跟`src`目录的绝对路径，另一个`utils`指向`src/utils`，从而能降低后续引用对应模块的路径的复杂度。

```js {6,7}
{
	// ... 其他配置
	alias: {
		// Support React Native Web
		'react-native': 'react-native-web',
		'@': path.resolve(__dirname, '../src'),
		'utils': path.resolve(__dirname, '../src/utils')
	},
}
```

现在，我们可以很方便的使用别名来引入了，去到`router`目录，修改下`index.js`试试。

``` jsx {2, 3}
// 将第五、第六行引入组件的代码用别名改写一下
import Login from '@/views/Login';
import Chat from '@/views/Chat';
```

重启服务，可以看到`Login`和`Chat`被正确引入了。
**这一部分的源代码可以在git仓库里面的`1-2`分支上查看。**


### 适配移动端

项目里我使用的是`rem`布局，在css中`em`是相对于父节点的`font-size`来参照大小的，而`rem`呢，就是参照`root`(根)节点，在浏览器中，这个根节点就是`html`标签。所以我们通过**获取用户的屏幕尺寸，通过一定比例来改变页面中`html`标签的`font-size`属性**，这样所有使用了`rem`单位的属性，就能响应式的完成不同屏幕尺寸的适配。

> 具体源码我放在`src/utils/fit2rem.js`文件中，原理我就不赘述了，感兴趣的朋友可以阅读源码+百度谷歌。
在`src/index.js`中引入
由于我习惯性以`750px`的设计稿做参照，这里`px`转化为`rem`都要除以75倍，这里就可以在`common.scss`封装一个简易的计算方法

```scss
//px2rem
@function rem($size){
	@return ($size / 75) * 1rem;
}
//  也可以在这里加入其它方法或者mixin 后续哪里用到了引进去撸就行了

// eg: 正方形
@mixin square($h){
    width:$h;
    height:$h;
}
```

**接着引入全局样式**

虽然在React项目中，在组件js文件中引入的样式就是全局的，但是为了增加仪式感，同时也是让以后方便管理和定位问题，还是让我们在`index.js`中引入`css/index.scss`和`normalize.css`。

> normalize.css 是一个有别于传统reset.css的样式重置文件，相比之下，reset.css有时候显得太过暴力了，会造成一些不必要的性能损耗，而normalize.css就温柔多了...

这里记得安装下用到的`node-sass`依赖
```shell

# 这里如果有安装不了的同学，百度一下npm淘宝镜像，用cnpm安装一下
$ yarn add node-sass

```

> p.s. 这里引入的css文件就在仓库直接拷贝吧，我就不放出来了

在`index.js`中引入
```jsx {7, 8}

import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

// 由于我们这里将utils配置了别名，webpack会自己将utils解析为src下的utils目录
import 'utils/fit2rem';
import './index.scss';

ReactDOM.render(
	<App/>,
	document.getElementById('root')
);

```

**这一部分的源代码可以在git仓库里面的`1-3`分支上查看。**

### 实现简单的路由拦截

由于做的是一个即时聊天的demo，就要实现登录和注册功能了。这里先简单的使用`cookie`实现登录权限控制，后面有时间，才改写成`token`的形式吧（其实是我后端部分还不知道怎么弄）。

先封装两个获取`cookie`指的工具函数，编写于`utils/index.js`工具库中。

然后让我们改一下`router`，的给组件`Chat`(聊天界面)增加路由拦截，通过“简单粗暴”地判断是否在cookie中存有我们用户的用户名，来实现路由跳转。如果有一个`username`的cookie，那么就粗暴地认为用户已经登录获得了权限，如果没有，就跳转到`Login`(注册/登录)组件，让用户去注册或者登录。

```jsx {3, 14, 15, 16, 23, 36}
// router/index.js文件中
// ...其他代码不变，引入Redirect重定向组件
import {Route, Switch, Redirect} from 'react-router-dom';
// 引入cookie工具函数
import {getCookie} from 'utils';

export default class Router extends React.component{
	render(){
		return (
			<Switch>
				{/* 登录/注册 */}
				<Route exact path="/login" component={Login}/>
				{/* 有登录权限的聊天界面 */}
				<PrivateRouter path="/">
					<Chat/>
				</PrivateRouter>
			</Switch>
		)
	}
}

// 登录权限控制
function PrivateRouter({children, props}) {
	return (
		<Route
			{...props}
			render={({location}) => {
				const hasAuthority = getCookie('username');
				// 这里将页面路由信息location传递进Login组建
				return (
					hasAuthority ? 
					children : 
					<Redirect
						to={{
							pathname: 'login',
							state: {from: location}
						}}
					/>
				)
			}}
		/>
	)
})
```

由于没登录的时候，访问的任何路由都会被重定向到`/login`路由，这里就需要将跳转前的路由信息传递给`Login`组件，等用户成功登录之后，再从`Login`组件中跳转回来继续访问。
此时已经可以在项目中尝试输入非`/login`的path的时候，都会跳转到login里面了，这是因为我们没有请求接口，也没有手动往浏览器的`cookie`中写入`username`，所以一直被浏览器跳转到`/login`路由的缘故，当我们直接在控制台写入一条`username`的cookie时，就会发现已经可以为所欲为的去任何路由了。

```js
// 在浏览器控制台中，直接操作cookie
document.cookie='username=willliam';
// 在修改浏览器地址栏的地址，就发现不会被拦截在login路由之外了
```

**这一部分的源代码可以在git仓库里面的`1-4`分支上查看。**

### 将Login组件撸出来
我们要实现的组件视图长这样。

![](http://cdn.liwuhou.cn/blog/20200315001611.png)


关于样式的编写，我就不多赘述了，因为我觉得这是基础中的基础。
这里放出主要功能代码，完整的代码，还是可以在源码中看到

```jsx
import React from 'react';
// 这里封装了一个弹框组件
import Alert from '@/components/Alert';

import {login} from '@/api'
import './index.scss';

export default class extends React.Component{
	constructor(props){
		super(props);
		// 这里需要获取到路由跳转的from信息，好让我们登录的时候回到上一个页面
		const {from} = this.props.location.state || {from: {pathname: '/'}}
		this.state = {
			from,
			username: '',
			password: '',
			// 弹框信息
			AlertProps: {
				isShow: false,
				title: '提示',
				description: '',
				onConfirm: this.onAlertConfirm,
			},
		}
	}
	
	// 输入用户名
	handleChangeUsername = (event) => {
		const username = event.target.value.replace(/\s+/, '');
		this.setState({
			username
		})
	}
    
	// 输入密码
	handleChangePawsword = (event) => {
		const password = event.target.value;
		this.setState({
			password
		})
	}

	handleKeyUpLogin = (event) => {
		if(event.keyCode === 13) return this.handleClickSubmitUserData();
	}

	// 进入
	handleClickSubmitUserData = async () => {
		const {username, password} = this.state;
		if(!username) return this.showAlert('请输入摸鱼者大名！');
		if(!password) return this.showAlert('请输入摸鱼口令！');
		if(username.length < 2) return this.showAlert('行走江湖名号怎可少于两个字！');
		if(username.length > 8) return this.showAlert('摸鱼人的大名再大也大不过八个字！');
		if(password.length < 6) return this.showAlert('口令太短，阁下请重新输过！');
		if(password.length > 16) return this.showAlert('口令这么长，我都记不过来！');

		console.log("Login -> handleClickSubmitUserData -> login", login)
		const data = await login({username, password});
		console.log("Login -> handleClickSubmitUserData -> data", data)
		if(data.status !== 1) return this.showAlert(data.message);
		
		// 模拟请求成功后，响应头写入cookie的操作
		document.cookie='username=willliam';
		
		// 验证成功，跳转回前一个页面
		this.props.history.replace(this.state.from);
	}

	// 弹框展示
	showAlert(description, title = '提示'){
		this.setState(({AlertProps}) => ({
			AlertProps: {
				...AlertProps,
				isShow: true,
				title,
				description
			}
		}))
	}

	// 弹框中的确定
	onAlertConfirm = () => {
		this.setState(({AlertProps}) => ({
			AlertProps: {
				...AlertProps,
				isShow: false,
				description: ''
			}
		}))   
	}

	render(){
		return (
			<div classname="login">
				<img className="login__sologan" src="./sologan.svg" alt="武陵人摸鱼为业"/>
				<div className="login__input">
					<label htmlFor="username" className="login__input_label">摸鱼人：</label>
					<input
						type="text"
						id="username" 
						placeholder="请输入昵称"
						value={this.state.username}
						onKeyUp={this.handleKeyUpLogin}
						onChange={}
					/>
				</div>
				<div className="login__input">
					<label htmlFor="password" className="login__input_label">摸鱼口令：</label>
					<input 
							id="password" 
							type="password" 
							placeholder="请输入密码"
							value={this.state.password} 
							onKeyUp={this.handleKeyUpLogin}
							onChange={this.handleChangePawsword}
					/>
					<i 
						className="login__info"
						onClick={() => this.showAlert(`
							初次进入将会注册昵称和账号，后续进入只需要输入已注册的账号和口令即可。
					`)}></i>
				</div>
				<div className="login__submit">
					<input
						value="进 入" 
						type="button" 
						onClick={this.handleClickSubmitUserData}
					/>
				</div>
			</div>
		)
	}
}

```

在`src`目录下新建`api`目录，然后在目录中创建`index.js`用来存放api接口配置。

```js
// @/api/index.js
import axios from 'axios'
import qs from 'qs';

// 注册/登录
export const login = async (params) => {
	// 由于我们还没开始写后端接口，这些先在公共目录下放一个json文件
	const {data} = await axios.get('/api/login.json', qs.stringify(params));
	return data;
}
```

然后把`img`标签用到的svg图像放在`public`下，并在`public`目录中新建`api`目录，里面存放一个叫`login.json`的文件，用来模拟登陆接口。

```json
// /public/api/login.json
{
	"status": 1,
	"data": "ok"
}
```

**这一部分的源代码可以在git仓库里面的`2-1`分支上查看。**

### 将Chat组件撸出来

`/chat`页面的ui图如下，因为这个只是一个检验我koa2和mongodb学习成果的小demo，所以功能非常简陋，登录完就直接进到唯一的聊天室了。
在这里，我用了一个`socket.io`的ws库来实现即时聊天的功能，这个库以前搞俄罗斯方块在线对战小demo的时候用过，还挺熟的，就没那么快多顾虑直接拿来用了。

![](http://cdn.liwuhou.cn/blog/20200315002044.png)

在日常开发中，我们常常会根据功能或者视图封装成组件，因为这样不仅实现功能模块复用的同时，也会把你的逻辑抽离成一个又一个具体且独立的模块，让你不用每次有改动的时候，都在用到相同的功能的地方ctrl+c和ctrl+v。

在`React`中，我常常有意无意将组件构建成一个类似像“纯函数”的单纯组件——接收参数（props），输出结果（视图）。除此之外，啥都不做，我感觉这样以后出bug也容易去定位。

我按照标题栏(`Heading`)、消息列表(`MessageList`)、消息(`Message`)、时间戳(`Timer`)、通知(`Notice`)和输入框(含发送按钮`ChatInput`)拆分成组件。

![](http://cdn.liwuhou.cn/blog/20200315194103.png)

先把聊天室的容器写好，这里老规矩，只展示关键的代码，样式和其他一些细节可以查阅源代码

```jsx
import React from 'React';
import Heading from '@/components/Heading';
import MessageList from '@/components/MessageList';
import ChatInput from '@/components/ChatInput';
export default class Chat extends React.Component{

	render(){
		return (
			<div className="chat">
				<Heading/>
				<MessageList/>
				<ChatInput/>
			</div>
		)
	}
}
```

然后在`components`下编写以上这些组件。

#### 标题栏

首先，标题栏是应用里将被复用得最多的组件之一，这里在聊天界面中就显示聊天室名称和当前在线人数。如果是其他界面就显示当前页标题。

```jsx
import React from 'React';
// Heading组件，比较简单，接收当前标题(heading)和人数(count)
export default function Heading({heading, count}){
	return (
		<div className="chat__heading">{heading}{count ? `(${count})` : ''}</div>
	)
}
```

#### 通知组件

通知组件用来投放一些系统级的通知，诸如用户进入/退出聊天室这些广播消息。这里立个flag，我还会在后续项目中会实现类似微信上的撤回信息和游戏邀请等其他功能。

```jsx
// Notice组件
import React from 'React';
export default function({content = ''}){
	return (
		<div className="notice_wrap">
			{content}
		</div>
	)
}
```

#### 时间戳组件

时间戳组件，即时聊天软件中必不可少的用来显示发信时间的功能根据与当前系统时间的对比，来动态的显示出`刚刚`、`1分钟前`这种比较友好的显示方式。

```jsx
import React from 'react'
import {formatMillisecond} from 'utils'
/**
 * @time    发送时间
 * @now     当前时间
 */
// 组件接收消息发送的时间和系统当前时间，通过formatMillisecond来展示时间
export default function Timer({time, now = Date.now()}){
	if(!time) return null;

	return (
		<div className="message_time">
			<p>{formatMillisecond(time, now)}</p>
		</div>
	)
}
```

这里由于考虑到将来的复用，将`formatMillisecond`函数提取到了公用方法里，感兴趣的同学可以看相关源代码。

接下来是消息内容了，这里我通过判断发消息的用户名跟当前用户名来判断是不是出自本人的消息，如果是本人的，那么理所应当是要在右侧的，其他人都是出现在左侧，这点符合我们日常使用聊天app的习惯。

#### 单条消息内容组件

```jsx
import React from 'react'

// 获取当前用户头像（取用户名）
const getuserIcon = (name = '') => {
	// 是否含有中文
	const hasCn = name.match(/[\u4e00-\u9fa5]/g);
	if(hasCn && hasCn.length){
		return name.slice(name.length - 2);
	}else{
		return name.slice(0, 4);
	}
}

/**
 * @isSelf      是否当前用户的发言
 * @username    这则消息的发言人
 * @content     消息内容
 */
export default function Message({isSelf, username, content}){
	return (
		<div className={`message ${isSelf ? 'right' : 'left'}_message`}>
			<div className="user">
				<span className="user_icon">{getuserIcon(username)}</span>
			</div>
			<div className="speech">
				<div className="username">{username}</div>
				<div className="content">{content}</div>
			</div>
		</div>
	)
}
```

#### 消息列表

因为后端接口也是我们负责的，这里就可以开始考虑接口返回的聊天记录的数据结构了。我打算使用一个对象数组的json格式来表示聊天记录，大致结构如下：

```json
{
	"data": [
		{
			"content": "我是内容", // String
			"time": "发送时间", // Date
			"username": "发消息的用户名", // String
			"_id": "数据库的唯一标识符", // String
			"isShowTime": "如果当前消息时间与上一条消息时间已经隔了有5分钟，就需要重新展示时间了" // Boolean
		}
	]
}
```

既然都全栈了，那么前端这边对数据的处理能少就少了。就拿上面的`isShowTime`这个字段来说，通过后端在插入聊天数据的时候，对比数据中最尾部消息的时间，如果超过五分钟就为`true`，然后前端这边直接展示时间戳组件。

很多情况下，用`Nodejs`做要比直接放在前端做性能要更好，也更能节省用户流量的。这也是`Node`作为后台或者中台，在提供接口或者接口融合和异构转换下的优势。

好了，不小心又扯远了，让我们接着来撸出`MessageList`。

```jsx
// 这里我就用了React hooks的方式写组件了
import React, {useState, useEffect, useRef, Fragment, useImperativeHandle} from 'react';

export default function MessageList({list, ownUsername}){
	// 当前系统时间戳，为节省性能，会5分钟刷新一次
	let [currentDate, setDate] = useState(Date.now());

	// 外部容易和聊天容器的ref
	let chatWrap = useRef(null);
	let messageWrap = useRef(null);

	// 暴露子组件的方法给父组件
	useImperativeHandle(ref, () => ({
		// 是否需要滑动到底部
		isNeedSlider(){
			return (chatWrap.scrollHeight - (chatWrap.current.scrollTop + chatWrap.current.offsetHeight)) > 500;
		},
		// 滑动到底部的方法
		sliderToBottom(){
			chatWrap.current.scrollTop = messageWrap.current.offsetHeight;
		}
	}))

	// 这里每设置一个定时器，每个五分钟刷新下currentDate
	useEffect(() => {
		let timer = setInterval(() => setDate(Date.now()));
		return clearInterval(timer);
	}, []) // 传入空数组，我不希望state更新的时候也调用这个副作用

	return (
		<div className="chat__content" ref={chatWrap}>
			<ul ref={messageWrap}>
				{
					list.map(({_id, time, isShowTime = false, username, content, event = 'chat'}) => (
						<li key={_id}>
							{
								event === 'notice' ? <Notice content={content}/> : (
									<Fragment>
										{isShowTime && <Timer time={time} now={currentDate}/>}
										<Message isSelf={ownUsername === username} content={content} username={username}/>
									</Fragment>
								)
							}
						</li>
					))
				}
			</ul>
		</div>
	)
}
```

#### 输入发送框

最后在封装下`ChatInput`组件，由于我们期望组件越单纯越好，这里会将逻辑都放到外部组件去，只留一个输入框和按钮给到他用来展示和交互。

```jsx
import React from 'react';

export default function ChatInput({onSendMessage}){
	const [message, setMessage] = useState('');
	// 输入框回车发送
	function handleKeyUpMessage(event){
		if(event.keyCode === 13) return handleClickSendMessage;
	}

	// 发送按钮
	handleClickSendMessage(){
		if(!message) return;
		onSendMessage(message);
		setMessage('');
	}
	return (
		<div className="chat__input">
			<input
				className="chat__input_white"
				value={message}
				onChange={setMessage}
				onKeyUp={handleKeyUpMessage}
			/>
			<button
				className="chat__input_btn"
				onClick={this.handleClickSendMessage}
			>发送</button>
		</div>
	)
}
```

#### 在Chat组件中引入

最后回到`Chat`组件，将`Heading`、`MessageList`和`ChatInput`引入

```jsx {30,31,32}
// Chat.jsx
class Chat extends React.Component{
	this.state = {
		heading: '摸鱼俱乐部',
		count: 1,
		messageList: [ // 假数据
			{
				_id: 'xx',
				time: Date.now() - 1000,
				username: '阿五',
				event: 'notice',
				content: '阿五帅气地进入了聊天室'
			},
			{
				_id: 'xxx',
				time: Date.now(),
				username: '阿五',
				event: 'chat',
				content: 'Hello world！',
			}
		]
	}
	onSendMessage = (message) => {
		// 跟接口交互，发送聊天信息
	}
	render(){
		const {heading, count, messageList, ownUsername} = this.state;
		return (
			<div className="chat">
				<Heading heading={heading} count={count}/>
				<MessageList list={messageList} ownUsername={ownUsername}/>
				<ChatInput onSendMessage={this.onSendMessage}/>
			</div>
		)
	}
}
```

### 小结

至此，我们这个小应用中，前端部分的代码算是告一段落了，剩下的只要结合接口，在`Chat`组件的生命周期中调用api接口获取聊天记录，然后在`onSendMessage`方法里请求接口发送消息。在这篇文章中，我们先用`create-react-app`脚手架搭建了一个项目，一步一步的按照组件的规范编写项目中前端部分的代码。其中用了`public`目录放置json文件来模拟接口返回的操作，封装了公用方法和公用组件，也尝试用了`React Hooks`这种比较新的写法。

下一篇将从零开始，说下小应用中，后端代码部分的编写。

> 每次花个十分钟，懂一个React知识点，走得虽慢，只要坚持走下去，足以致千里。

关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)

