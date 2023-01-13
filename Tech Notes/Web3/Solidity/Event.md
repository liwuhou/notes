Solidity 中事件是 EVM 上日志的抽象，主要具有如下两个特点：
- 响应：应用程序（[ethers.js](https://learnblockchain.cn/docs/ethers.js/api-contract.html#id18) 可以通过 `RPC` 接口订阅和监听这些事件，并在前端做响应。
- 经济：事件是 EVM 上比较经济的存储方式，每个大概消耗 2000 [[Gas]]；相比之下，链上存储一个新变量至少需要 20000[[Gas]]。

### 声明事件
声明一个事件由 `event` 关键字开关，接着是事件名称，括号里面写好事件需要记录的变量类型和变量名。

```sol
event Transfer(address indexed from, address indexed to, uint256 value);
```

以`ERC20`代币合约的`Transfer`事件为例，事件中共记录了 3 个变量`from`、`to`和`value`，分别对应代币的转账地址、接收地址和转账数量，其中`from`和`to`前面带有 `indexed` 关键字，他们会保存在以太坊虚拟机日志的 `topics`中，方便之后检索。

### 释放事件
可以在函数中释放事件

```sol
function _transfer(address from, address to, uint256 amount) {
  _balances[from] = 1000000 // 给转账地址一些初始代币
  _balances[from] -= amount;
  _balances[to] += amount;

  // 释放事件
  emit Transfer(from, to, amount);
}
```

### EVM 中的日志 Log
以太坊虚拟机（EVM）中，用日志（Log）来存储 Solidity 事件，每条日志记录都包含主题 `Topics` 和数据 `data` 两部分

![](http://cdn.liwuhou.cn/blog/202212262141165.png)

#### 主题 Topics
日志的第一部分是主题数组，用于描述事件，长度不能超过`4`。它的第一个元素是事件的签名（哈希）。对于上面的`Transfer`事件它的签名就是：

```sol
keccak256("Transfer(address,address,uint256)")
//0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
```

除了事件签名，主题还可以包含至多 3 个`indexed` 参数，也就是 `Transfer` 中的 `from` 和`to`。

`indexed`标记的参数可以理解为检索事件的索引“键”，方便之后搜索。每个`indexed`参数的大小为固定的 256 比特，如果参数太大了（比如字符串），就会自动计算哈希存储在主题中。

#### 数据 Data
事件中不带`indexed`的参数会被存储在`data`部分中，可以理解为事件的“值”。`data`部分的变量不能被直接搜索，但可以存储任意大小的数据。因此一般`data`部分可以用来存储复杂的数据结构，例如数组和字符串等等，因为这些数据超过了 256 比特，即使存储在事件的`topics`部分中，也是以哈希的方式存储。另外，`data`部分的变量在存储上消耗的[[Gas]]相比于 Topics 上要更少。

