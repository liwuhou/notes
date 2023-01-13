在 Solidity 0.6.x 版本之前，语法只有 `fallback` 函数，用来接收用户发送的 ETH 时调用，以及在被调用函数签名没有匹配到时会调用。而在 0.6 版本之后，Solidity 将`fallback`函数拆分成 `receive` 和 `fallback` 两个函数。

### 接收 ETH 的函数 —— receive
`receive` 只用于处理[[transfer() & send() & call()#接收ETH]]，一个合约最多有一个 `receive`函数，声明方式与一般函数不一样，`receive` 不需要 `function`关键字，不能有任何参数，不能返回任何值，而且必须包含 `external` 和 `payable`。

```sol
contract Test {
  receive() external payable {}
}
```

当合约接收 ETH 的时候，`receive` 会被触发。**`receive` 最好不要执行太多的逻辑**，因为别人使用 `send` 和 `transfer` 方法发送 ETH 的话，[[Gas]]会限制在 2300，而 `receive`太复杂可能会触发 `Out of GAS`报错；如果用[[transfer() & send() & call()#call]] 就可以自定义  `gas` 执行更复杂的逻辑。

有些恶意的合约，会在 `receive` 函数（老版本的话，就是 `fallback`函数）嵌入恶意消耗   `gas` 的内容或者使得执行故意失败的代码，导致一些包含退款和转账逻辑的合约，不能正常工作，因此写包含退款等逻辑的合约的时候，一定要注意这种情况

### 回退函数 —— fallback
`fallback`函数会在调用合约不存在的函数时触发。可用于接收 ETH，也可以用于代理合约 `proxy contract`。`fallback` 跟 `receive` 一样，声明时不需要 `function`，且必须由`external` 修饰，一般也会用 `payable` 修饰，用于接收 ETH。

```sol
fallback() external payable{}
```

### receive 和 fallback 的区别
`receive` 和 `fallback` 都能够用于接收 ETH，他们触发的规则如下：

```
触发fallback() 还是 receive()?
           接收ETH
              |
         msg.data是空？
            /  \
          是    否
          /      \
receive()存在?   fallback()
        / \
       是  否
      /     \
receive()   fallback()
```

简单来说，合约接收`ETH`时，`msg.data`为空且存在`receive()`时，会触发`receive()`；`msg.data`不为空或不存在`receive()`时，会触发`fallback()`，此时`fallback()`必须为`payable`。

`receive()`和`payable fallback()`均不存在的时候，向合约**直接**发送`ETH`将会报错（你仍可以通过带有`payable`的函数向合约发送`ETH`）。