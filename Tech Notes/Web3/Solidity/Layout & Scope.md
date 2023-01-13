引用类型（Reference Type）：包括[[Reference Type|数组（array）]]，结构体（struct）和映射（mapping），这类变量的特点就是占用的空间大，赋值的时候是直接传递地址。而且变量也比较复杂，占用存储空间大，使用时也必须声明数据存储的位置。

### 数据位置(Layout)
为了节省链上有限的空间和 *[[Gas|gas]]*，`memory`和`calldata`。
因为在链中，存储在不同的位置消耗的 *gas* 成本不同。`storage` 类型的数据存于链上，类似计算机的硬盘，所以消耗的*gas* 多；而 `memory` 和 `calldata`类型的变量是临时存在内存里的，因此消耗的*gas*较少。

- `storage`：合约里的状态变量默认都是`storage`上的，存储在链上。
- `memory`：函数里的参数和临时变量一般用`memory`，存储在内存中，不上链。
- `calldata`：和`memory`一样，也是存储在内在中，不上链。不同的是`calldata`声明的变量不可修改（**immutable**)，一般用于函数的参数。(这里不同于 js 中 const 声明的变量，`calldata` 声明的变更指针和所指向的数据都是不可改变的)

> 在合约中，类中的声明的引用类型的状态变量不用加数据位置的声明，因为默认就是 `storage`。
> 而函数中的声明引用类型变量必须要有 `storage`，`memory` 或 `calldata` 的声明关键字，来标识其中的数据是存放在何种位置。

### 数据位置和赋值规则
不同的存储位置在相互赋值的时候，会有相同引用或者拷贝一份副本的表现。这是因为不同的数据位置之间相互赋值有相应的规则。

当 `storage` 给 `storage` 赋值的时候，会产生一个相同的引用。

```javascript
uint[] _x = [1, 2, 3];

function Xstorage() public {
  uint[] storage _y = _x;
  _y[0] = 100; // _x 因为有相同的引用，所以相应下标的元素也会被改变
}
```

当 `storage` 赋值给 `memory` 时，会创建一个新的副本，`memory`的变量更改不会影响到`storage`位置的变量

```javascript
uint[] _x = [1, 2, 3];

function Xmemory() public {
  uint[] memory _y = _x;
  _y[0] = 1000; // _x 不受影响
}
```

当`memory`赋值给 `memory`的时候，会产生相同的引用，修复任一变量都会对其余引用的变量产生影响。

```js
uint[] _x = [1, 2, 3];


```

当`memory`赋值给`storage`的时候，会创建一个新的副本

```js
uint[] _x = [];

function Xstorage() public {
  uint[] memory _y = [1, 2, 3];
  _x = _y;
  _y[0] = 100; // 不会影响_x
}
```

> 总结一下，`storage` 和 `memory` 之间的互相赋值会产生新的副本，修改数据不会互相影响。而 `storage`与`storage`，`memory`与`memory`之间的互相赋值，会复制相同的引用，变量之间引用的是同一块数据源，对任一变量的改动会影响到其它地方。

### 变量的作用域(Scope)
**Solidity**的变量分为三种：状态变量（state variable）、局部变量（local variable）和全局变量（global variable）

#### 1. 状态变量
状态变量即合约的内部变量，**合约内的所有函数**均可访问，*gas*消耗高。

```js
contract Contract {
  uint public _number = 0;
  string public z;
}
```

我们也可以在函数内改变状态变量的值

```js
contract Contract {
  bool public win_flag = false; // state variable

  function win() {
    win_flag = true;
  }
}
```

#### 2. 局部变量
局部变量声明在函数内部，函数执行完毕之后变量即失效，这点跟 js 类似。局部变量存储在内存中，不上链，因此消耗的 *gas* 低。

```js
contract Contract {
  uint public _x = 0;

  function LocalVariableFn() {
    uint min_number = 0; // local variable
  }
}

```


#### 3. 全局变量
全局变量是全局范围工作的变量，都是`solidity`预留关键字。他们可以在函数内不声明直接使用：

```
function global() external view returns(address, uint, bytes memory){
  address sender = msg.sender;        
  uint blockNum = block.number;        
  bytes memory data = msg.data;        
  return(sender, blockNum, data);    
}
```

在上面例子里，我们使用了3个常用的全局变量：`msg.sender`, `block.number`和`msg.data`，他们分别代表请求发起地址，当前区块高度，和请求数据。下面是一些常用的全局变量，更完整的列表请看这个[链接](https://learnblockchain.cn/docs/solidity/units-and-global-variables.html#special-variables-and-functions)：

-   `blockhash(uint blockNumber)`: (`bytes32`)给定区块的哈希值 – 只适用于256最近区块, 不包含当前区块。
-   `block.coinbase`: (`address payable`) 当前区块矿工的地址
-   `block.gaslimit`: (`uint`) 当前区块的 `gaslimit`
-   `block.number`: (`uint`) 当前区块的number
-   `block.timestamp`: (`uint`) 当前区块的时间戳，为unix纪元以来的秒
-   `gasleft()`: (`uint256`) 剩余 gas
-   `msg.data`: (`bytes calldata`) 完整call data
-   `msg.sender`: (`address payable`) 消息发送者 (当前 caller)
-   `msg.sig`: (`bytes4`) `calldata` 的前四个字节 (function identifier)
-   `msg.value`: (`uint`) 当前交易发送的`wei`值

