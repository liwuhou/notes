---
title: React源码笔记——jsx为何物
date: 2020-10-15
tags:
  - Javascript
  - React
summary: jsx的本质是什么？为什么要用jsx开发？
---

# jsx 为何物

最近开始看点 `React` 的源码，想找点地方输出一些笔记。以前写过 `React` 的[jsx](https://liwuhou.cn/2019/08/14/%E8%B5%B0%E8%BF%9Breact%E2%80%94%E2%80%94jsx/)，如果对 `jsx` 不太熟悉或者想回忆回忆的可以移步去看看。

用了那么久的 `React` ， `jsx` 简直就是天天打照面的了。然鹅用得再熟，还是只是停留在形而上而已，像面试的时候，被问到 jsx 的本质是啥？还能不假思索说出是一个 js 对象，那这个对象怎么来的，是通过什么方式拓展了 js 这种类似 html 语法的能力的，就开始支支吾吾了。为了知其然知其所以然，也为了能满足自己的好奇心，我不由得想像小时候拆电话一样拆开 React 的源码一探究竟。最后拼回去还能不能打电话就不是我考虑的事了（老爸对不起，我下次不敢了）。

<!-- more -->

参考 `React` [官网的文档里对 jsx 的描述](https://reactjs.org/docs/glossary.html#jsx)——JSX 是一种 Javascript 的语法拓展，是一种具有全部 js 能力的模板语法。JSX 会被编译为 `React.createElement()` 方法，该方法调用后会返回一个被称为 `React元素` 的**纯 js 对象**。

> JSX is a syntax extension to JavaScript. It is similar to a template language, but it has full power of JavaScript. JSX gets compiled to React.createElement() calls which return plain JavaScript objects called “React elements”

所以 jsx 经编译之后变为 `React` 中的 `createElement` 方法的调用，负责这个[jsx 编译过程的工具](https://babeljs.io/repl/#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&spec=false&loose=false&code_lz=MYewdgzgLgBApgGzgWzmWBeGAeAFgRgD4AJRBEAGhgHcQAnBAEwEJsB6AwgbgCgeg&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=react&prettier=false&targets=&version=7.12.0&externalPlugins=)呢，不是别人，就是我们工程化中天天打交道的[babel](https://babeljs.io/)。

ok，到这一步 jsx 的神秘面纱已经被我们揭开了，所以下面这两段代码是等价的：

```JSX

const element = <h1 style={{color: '#abcdef'}}>hello JSX!</h1>

```

经过 babel 编译后

```js
const element = React.createElement(
  'h1',
  {
    // 组件的属性或者props等
    style: {
      color: '#abcdef'
    }
  },
  'hello JSX!'
)
```

从上面 React 的官网文档中可以得知，`React.createElement` 方法返回的是一个**纯 js 对象**，所以别着急，让我们再扒一扒 `createElement` 的源码。

```JavaScript
/**
 * Create and return a new ReactElement of the given type.
 * See https://reactjs.org/docs/react-api.html#createelement
 * type       用来标识是一个html元素还是一个React组件
 * config     html元素/组件里面的属性集合
 * children   子元素的集合
 */
export function createElement(type, config, children) {
  let propName;

  // 提取名称
  const props = {};
  // 初始化React元素中这四个重要变量
  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  if (config != null) {
    // 获取config中有效的ref
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    // 获取config中有效的key
    if (hasValidKey(config)) {
      key = '' + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 将config中的属性都转移到变量 props 中
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // 除了type和config之外，后面的参数都是children
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    // 如果只有一个children，直接赋值给props.children
    props.children = children;
  } else if (childrenLength > 1) {
    // 如果有多个children，转换为array赋值给props.children
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    props.children = childArray;
  }

  // 从defaultProps中赋值默认值给props
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }

  // 最后返回的是ReactElement方法的结果
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}

```

这里是[createElement 源码链接](https://github.com/hugewilliam/react/blob/master/packages/react/src/ReactElement.js#L344)，我自己加了点注释，删了一些 react 在 dev 环境的相关代码。可以看出，这个`createElement`只是一个工具人函数，他的作用就只是处理了传入的 `type`、`config`和`children`参数(不算上一些在 dev 环境做的操作的话)，最后输出了一个叫`ReactElement`方法的调用结果。让我们在顺藤摸瓜，摸一摸这个`ReactElement`的方法啥样子：

```javascript
export function ReactElement(type, key, ref, self, source, owner, props) {
  const element = {
    // REACT_ELEMENT_TYPE只是一个常量，用来标识这是一个React元素
    $$typeof: REACT_ELEMENT_TYPE,
    type,
    key,
    ref,
    props,
    // 创造该元素的组件
    _owner: owner
  }
  // 然后就把这个组装好的对象return出去了，下班
  return element
}
```

这里是[ReactElement 源码链接](https://github.com/hugewilliam/react/blob/master/packages/react/src/ReactElement.js#L126)，没想到吧，`ReactElement` 也不过是个逻辑这么简单的工厂函数而已，通过 `createElement`组合而来的参数，组装成一个对象结构的 react 元素，这个 `element`对象呢，本质上只是一个**用来描述 dom 的 JavaScript 对象**，也就是我们一直说的**虚拟 dom**（节点）了。

虚拟 dom 转换为真实 dom 挂载到页面上过程，其实就是 `ReactDOM.render` 方法的调用过程。

```jsx
ReactDOM.render(
  // React元素
  <App />,
  // 真实的dom节点
  document.querySelector('#root'),
  // 可选的回调函数，用来处理渲染完毕之后的逻辑
  [callback]
)
```

`ReactDom.render` 方法接受三个参数，第一个是渲染的 React 元素，第二个参数接收一个**真实 dom**节点，React 元素最终都是渲染到这个真实 dom 元素里的。

![](http://cdn.liwuhou.cn/blog/公众号二维码1.0.png)
