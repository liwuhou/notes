---
title:  使用 Taro 2.x 跨平台开发的一些姿势
date: 2020-05-10
tags:  
  - Javascript
  - Mini Program
summary: 最近工作项目需要切换到使用Taro框架来编写小程序，经过一阵子的摸索之后，总结了一些更好地抹平多端差异的操作。
---

# Taro框架

> **Taro**是一套遵循`React`语法规范的多端解决方案。通过**Taro**编译工具，将源代码编译出不同端（[微信](https://mp.weixin.qq.com/) / [京东](https://mp.jd.com/?entrance=taro) / [百度](https://smartprogram.baidu.com/) / [支付宝](https://mini.open.alipay.com/) / [字节跳动](https://developer.toutiao.com/) 小程序、[快应用](https://www.quickapp.cn/)、H5、React-Native 等）的运行代码。

听着让人挺激动的，**write once, run anywhere**，从我开始写`JQuery`时就已经喊得响亮，彼时还只是能处理下不同浏览器中js的兼容性问题。如今Jq英雄迟暮，以ng、vue和react三大精神小伙为首的前端框架开始飞速发展，已然不满足于前端浏览器的这一亩三分地了...

但是别急着徜徉，理想虽然美好，现实却是很残酷的。随着开发的深入，发现了诸多跨平台的坑点，故整理了一下，才有了这篇水文。

<!-- more -->

### 多端中的样式兼容问题
在所有端的样式中，h5的兼容性最好，小程序的次之，最差的就是RN了。统一多端即是对齐短板**也就是要以RN的约束来管理样式，同时兼顾小程序的限制**，核心可以用三点来概括：

- 使用 `Flex` 布局
- 基于 `BEM` 写样式
- 采用 `style` 覆盖组件的样式

**flex**
这里有一点需要注意的是，RN中的`View`标签，默认的主轴方向是`column`。我们项目中采取的方案是，有用到`flex`布局的情况，都显式地声明下主轴方向。之所以不简单粗暴地将`View` 标签统一为 `display: flex; flex-direction: column`， 是因为 H5 上一些内置组件样式可能会错乱。

**BEM**
BEM其实是块（block）、元素（element）、修饰符（modifier）的缩写，利用不同的区块，功能以及样式来给元素命名。BEM命名规范可以有效的解决react中全局样式污染的问题，并且能让人光是看class名字就知道各个模块间的关系和作用。

**样式**
在taro编译RN端，对样式文件进行处理时，主要可以分为以下几步：

![](http://cdn.liwuhou.cn/blog/20200528103958.png)

上图中的`CSS to StyleSheet`是利用[css-to-react-native-transform](https://github.com/kristerkari/css-to-react-native-transform)[babel-plugin-transform-jsx-to-stylesheet](https://github.com/NervJS/taro/blob/master/packages/babel-plugin-transform-jsx-to-stylesheet)，将`Scss`/`Less`/`Css`文件转换成`React Native StyleSheet`文件和`JSX`语法中的`className`(包括如三目运算符、classnames( ) 等复杂表达式)直接替换为`style`。

以上就是Taro在编译RN时，为我们处理样式的一些操作。除此之外，Taro还提供了一些处理差异样式的条件编译。

**样式文件的条件编译**

假设目录中同时存在 `index.scss` 和 `index.rn.scss`样式文件，在js文件中引用样式文件的时候，只需要书写 `import './index.scss'`， 在RN平台会自动引入 `index.rn.scss`,而其他平台会引入 `index.scss`， 以此达到RN平台与非RN平台的样式文件的条件编译。

**样式代码的条件编译**

指定平台保留:

```scss
/* #ifdef %PLATFORM% */
// 样式代码
/* #endif */
```

指定平台剔除:
```scss
/* #ifndef %PLATFORM% */
// 样式代码
/* #endif */
```

`%PLATFORM%` 用来标记当前编译的平台类型,跟 [`process.env.TARO_ENV`](https://nervjs.github.io/taro/docs/envs.html#processenvtaro_env) 取值相同( `weapp` / `swan` / `alipay` / `h5` / `rn` / `tt` / `qq` / `quickapp`);

**如果是多个平台可以使用空格隔开**

例:

```css {4}
.wrap{
    background: red;
    /* #ifdef weapp h5 */
    background: green;
    /* #endif */
}
```

### 跨平台开发

Taro中暴露了一个`Taro.env.TARO_ENV`的变量来判断当前的编译类型，目前的取值有`weapp` / `swan` / `alipay` / `h5` / `rn` / `tt` / `qq` / `quickapp` 八个取值，可以通过这些个变量来实现不同环境下的代码，而且在编译时会将不属于当前编译类型的代码去掉，只保留当前编译类型下的代码，例如想在微信小程序和 H5 端分别引用不同资源：

```js
if (process.env.TARO_ENV === 'weapp') {
  require('path/to/weapp/name')
} else if (process.env.TARO_ENV === 'h5') {
  require('path/to/h5/name')
}
```

同时也可以在 JSX 中使用，决定不同端要加载的组件

```js
render () {
  return (
    <View>
      {process.env.TARO_ENV === 'weapp' && <ScrollViewWeapp />}
      {process.env.TARO_ENV === 'h5' && <ScrollViewH5 />}
    </View>
  )
}
```

以上这些内置变量虽好，但如果让代码中充斥着大量逻辑判断的语句，也是很影响日后维护的。Taro团队也做了这些考虑，如果有一个组件存在多端差异，完全可以使用组件对应平台命名的格式，比如有个`Test`组件，需要存在微信小程序、百度小程序和H5三个不同的版本，那么可以如下组织脚本文件

`test.js`文件，Test组件的默认形式，可以编译到微信小程序、百度小程序和H5三端之外的其他端使用的版本
`test.h5.js`文件，是Test组件的H5版本
`test.weapp.js`文件是Test组件微信小程序版本
`test.swan.js`文件是Test组件百度小程序版本

四个文件，对外暴露的是统一的接口，它们接受一致的参数，只是内部有针对各自平台的代码实现。

而使用Test组件的时候，引用方式依然和之前保持一致，直接import不带端类型的文件名，Taro编译的时候会自动识别并添加端类型后缀

```js{2}
// import Test from '@/components/test.h5'
import Test from '@/components/test'
```

### 小程序中没有cookies的问题

如果项目中，使用了`cookies`保存一些登录状态的话，那么小程序要多做一些兼容操作。其实也不复杂，就是将请求的`header`中，加多一个`Cookies`字段，如此而已。

```js {41}
import Taro from '@tarojs/taro';

// 保存的cookies
let cookies = Taro.getStorageSync('cookies') || '';
// 小程序环境
const isMP = Taro.getEnv() !== Taro.ENV_TYPE.WEB && Taro.getEnv() !== Taro.ENV_TYPE.RN

// 项目中统一请求的方法
export default const fetch = (option) => {
  const {url, data, method = 'GET', header = {}} = option;

  if(method === 'POST'){
    header['content-type'] = 'application/json'
  }
  if(isMp){
    header['Cookies'] = cookies;
  }

  return Taro.request({
    url,
    data,
    method,
    header
  })
}

export function post(url, data) {
  return fetch({ url, data, method: 'POST' })
}

export function get(url, data) {
  return fetch({ url, data, method: 'GET' })
}

// 登录接口
export const login = (data) => {
  return new Promise((resolve, reject) => {
    fetch({url: '/api/login', data, method: 'POST'}).then((res) => {
      if(res.code === 200){
        // 将响应头中的以逗号隔开的cookies用分号隔开然后储存
        cookies = res.header['Cookies'].replace(/,/g, ';');
        Taro.SetStorage('Cookie', cookies);
      }
      resolve(res);
    }).catch((error) => {
      reject(error);
    })
  })
}
```

### Taro多端开发的实现
开发了一段时间之后，加上一些了解，大概摸清了Taro多端开发的本质。Taro在编译时，会将你写的类React的语法抽象成一个语法树(AST)，随后对这个语法树进行分析和转化，使之生成目标端的语法。

![](http://cdn.liwuhou.cn/blog/20200510150110.png)

基于编译原理，Taro在编译时，会将各端具有差异的部分进行抹平。

**编译时**

例如在转换小程序端的时候，Taro里的`render`会将所有jsx元素在js文件中移除，它们经过一系列转换过后的会被编译成小程序的`wxml`。

例如在jsx中常见的循环操作：

```jsx
<View>
  {
    list.map(item => <Text onClick={this.handleClick}>{item}</Text>)
  }
</View>
```

在编译时，Taro会将上述代码编译为如下的Ast json格式：

```json
{
    "type": "element",
    "tagName": "text",
    "attributes": [
        {"bindtap": "handleCLick"},
        {"wx:for": "{{list}}"},
        {"wx:for-item": "item"}
    ],
    "children": [
        {"type": "text", "content": "{{item}}"}
    ]
}
```

函数的callee（`list`）会作为属性`wx:for`的值，匿名函数的第一个参数会作为`wx:for-item`的值，函数第二个参数是`wx:for-index`，并且`Text`元素的children是一个jsx表达式。有了这么一串数据结构，要生成一段`wxml`就具有可行性了，可以参考Taro源码中的`toHTML`函数——[https://github.com/andrejewski/himalaya/blob/master/src/stringify.js](https://github.com/andrejewski/himalaya/blob/master/src/stringify.js)

由于jsx的写法千变万化，Taro还未能支持到所有的写法，对于一些常用的Taro会进行相应的转换，而一些由于微信小程序的限制，**Taro还未支持的骚操作会使用ESLint警告或在编译时报错来阻止**(这也是Taro初期很多jsx语法不支持的原因)。随着Taro团队的迭代，现在jsx这块的支持程度和补充进展已经很不错了，详细的可以在Taro官网文档[最佳实践](https://nervjs.github.io/taro/docs/best-practice.html)中阅读。

**运行时**

编译时只转化代码语法，包含`jsx`向小程序`xml`的转化。而在组件层面上，Taro团队定制了一套运行时标准来抹平不同平台之间的差异。包含**标准运行时框架**、**标准基础组件库**、**标准端能力 API**，其中运行时框架和API对应`@taro/taro`，组件库则对应`@tarojs/components`，**以微信小程序为标准**，在不同的端实现这些标准，从而达到一套代码向多端代码的无缝转换。

同时配合编译过程，抹平了状态、事件绑定和生命周期的差异。通过运行时对原生api进行拓展，实现了诸如事件绑定时通过 bind 传递参数、通过 Promise 的方式调用原生 API 等特性。

这就是Taro在编译时和运行时对多端平台做的一些努力。

![](http://cdn.liwuhou.cn/blog/公众号二维码1.0.png)