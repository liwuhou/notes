## VDOM
`Vdom` 即 `virtual dom`，是一种用 json 格式来描述界面的结构，形如：

```json
{
  "type": "ul",
  "props": {
    "className": "list",
    "children": [
      {
        "type": "li",
        "props": {
          "className": "item",
          "style": {
            "background": "blue",
            "color": "pink"
          },
          "children": [
            {
              "type": "TEXT_ELEMENT",
              "props": {
                "nodeValue": "aa",
                "children": []
              }
            }
          ]
        }
      }
    ]
  }
}

```

在日常开发中，基本不会这么去写，可读性太差了，而是去写 `jsx` 

```jsx
const data = {
  item1: 'Roy',
  item2: 'William'
}

const jsx = <ul className="list">
  <li className="item" style={{ background: 'blue', color: 'pink' }} onClick={() => console.log('foo')}>{data.item1}</li>
  <li className="item">{data.item2}<i>bar</i></li>
</ul>;

console.log('jsx -> ', JSON.stringify(jsx, undefined, 2))
```

`jsx` 会被 `babel`  转译为一个生成 `vdom`的函数的调用，如下：

```javascript
var data = {
  item1: 'Roy',
  item2: 'William'
};
var jsx = React.createElement("ul", {
  className: "list"
}, React.createElement("li", {
  className: "item",
  style: {
    background: 'blue',
    color: 'pink'
  },
  onClick: function onClick() {
    return console.log('foo');
  }
}, data.item1), React.createElement("li", {
  className: "item"
}, data.item2, React.createElement("i", null, "bar")));

console.log('jsx -> ', JSON.stringify(jsx, undefined, 2))
```

这个 `React.createElement` 的函数称为 `render function` ，他的作用是根据调用返回相应的 `vdom`

这里不直接将 `jsx` 编译为 `vdom` 的原因在于， `render function` 可以执行动态逻辑，方便我们插入 `state` 和 `props`等数据，简单的包装一下，即可实现组件的逻辑复用。

### VDom 渲染
要渲染这样的 json 结构明显是要用到递归，对不同的类型做不同的处理。