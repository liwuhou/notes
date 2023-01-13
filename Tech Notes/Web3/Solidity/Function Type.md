在 **Solidity** 中函数的形式如下：

```js
    function <function name> (<parameter types>) {internal|external|public|private} [pure|view|payable] [returns (<return types>)]
```
其中，方括号中的是可写可不写的关键字，看着会有些复杂，我们从前往后一个一个关键字拆解

1. `function`：声明函数时的固定写法，想写函数就要以 `function` 关键字开头
2. `<function name>`：函数名
3. `(<parameter types>)`：圆括号中声明的函数的参数及类型
4. `{internal|external|public|private}`：函数可见性说明符，一共 4 种，默认是 `internal`
  - `public`：内部外部均可见。（也可用于修饰状态变量，`public` 变量会自动生成 `getter` 函数，用于查询数值）
  - `private`：只能从本合约内部访问，继承的合约也不可用（也可用于修饰状态变量）
  - `external`：只能从合约外部访问，但是可以用 `this.f()` 来调用（`f` 是函数名）
  - `internal`：只能从合约内部访问，继承的合约可以用（也可用于修饰状态变量）
5. `[pure|view|payable]`：决定函数权限/功能的关键字。
  - `payable`：可支付的，带着它的函数，运行的时候可以发送**ETH**或其它区块链原生通证。
  - `pure`：声明函数不会在链上读取和写入任何状态变量，不会产生 **gas**
  - `view`：声明函数只会在链上读取但不会写入任何状态变量，不会产生 **gas**
  - `default`：如果函数中的功能关键字为缺省，则说明函数会在链上改写状态，会产生**gas**
6. `[returns()]`：函数返回的变量类型和名称，有一个声明变量的过程，同时函数结束时会将其返回

### pure VS view
**Solidity** 引入 `pure` 和 `view` 这两个关键字，主要是为也节省 **gas** 和控制函数的权限。
在 Solidity 中，更改状态才会消耗 **gas**。
假设我们的智能合约中定义了一个状态变量 `number = 5`。

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;
contract FunctionType {
  uint256 public number = 5;
}
```

定义一个 `add()` 函数，每次调用，都给 `number` + 1

```js
function add() external {
  number = number + 1;
}
```

上面的函数是没有问题的，没有对 `add`函数定义 `pure` 和 `view`，可以对合约上的状态变量进行读写。但如果加上了 `pure` 或者 `view`，这段代码就会报错了。

```js
// wrong
function pureAdd() external pure {
  number++;
}

function viewAdd() external view {
  number++;
}
```

既然是 `pure` 声明的函数，就不能读取链上的变量，以`view`关键字声明的函数就不能改写链上的变量。
所以上面两处方法要改写为：

```js
function pureAdd(uint _number) external pure returns(uint new_number) {
  new_number = _number + 1; // 成为一个纯函数，不对链上数据进行读写
}

function viewAdd() external view returns(uint new_number) {
  new_number = number + 1; // 获取到合约上的变量
}
```

> 值得注意的是，调用 `view` 函数虽然是免费的，但你在消耗 **gas** 的函数中调用它时是会计算 **gas** 的 

### internal VS external
`internal` 和 `external` 两个关键字就很好理解了，被 `internal`定义的函数，只能在合约内部被调用，外部是调用不了的。而 `external` 声明的函数，合约内外都能够调用。

### payable
定义函数会发生交易动作的关键字，可以查看当前合约里的 **ETH** 余额， `this` 关键字可以让我们引用合约地址。

```js
function minusPayable() external payable returns(uint256 balance) {
  minus();
  balance = address(this).balance;
}
```

### return 和 returns
Solidity 有两个关键字与函数的输出相关，他们的区别在于：
- `returns`： 加在函数名后面，用于约束和声明函数返回的变量类型及变量名
- `return`：用于函数主体中，返回指定的变量

拿函数返回多个变量举例，在 returns 中定义后约束的函数返回值类型，然后在函数体中使用 `return` 关键字返回数据。

```js
// return multiple variable
function returnMultiple() public pure returns(uint256, bool, uint256[3] memory) {
  return (1, true, [uint256(1), 2, 5])
}
```

也可以在 `returns` 关键字后面一直声明了新变量，这样 **Solidity**会自动给这些变量初始化，并且会自动返回这些函数的值，不需要在函数体中使用 `return` 关键字。

```js
function returnMultiple() public pure returns(uint256 _number, bool _bool, uint256[3] memory _array) {
  _number = 2;
  _bool = true;
  _array = [uint256(3), 2, 1];
  // Don't need use `return`, they will automatically return
}
```

### 解构式赋值
既然函数有返回多个变量的，类似 python 中元组的特性，那么自然也支持使用元组来解构赋值多个变量。

```js
uint256 _number;
bool _bool;
uint256[3] memory _array;

(_number, _bool, _array) = returnMultiple();
```

### 函数签名
简单地说，函数签名即是某个函数接受形参的“模样”，举例来说，上面`returnMultiple`函数的函数签名就是`returnMultiple()`。简而言之，就是`函数名(逗号分隔的参数类型)`。


### 函数重载
Solidity 中允许函数进行重载(`overloading`)，即名字相同但函数签名不同的函数同时存在，他们被视为不同的函数。值得注意的是，Solidity 不允许 [[Constructor & Modifier#Modifier]]重载。
举个例子，我们可以定义两个都叫 `saySomething()`的函数，一个没有任何参数，输出`"Nothing"`；另一个接收一个 `string` 类型的参数，输出这个 `string`。

```sol
function saySomething() public pure returns(string memory) {
  return "Nothing";
}

function  saySomething(string memory something) public pure returns(string memory) {
  return something;
}
```

最终重载函数经过编译器的编译之后，由于不同的参数类型，都变成了不两路的函数选择器（[[Selector]]）。

在 [Remix](https://remix.ethereum.org) 部署之后，看到有两个函数选择器。

![](http://cdn.liwuhou.cn/blog/202301092233121.png)

#### 实参匹配
在调用具有多个函数签名的重载函数时，会把输入的实际参数和函数参数的变量类型做匹配。**如果出现多个匹配到的重载函数，则会报错**。

```sol
function f(uint8 _in) public pure returns(uint8 out) {
  out = _in;
}

function f(uint256 _in) public pure returns(uint256 out) {
  out = _in;
}


```

当调用 `f(50)` 就会报错，因为 `50`又可以是 `uint8`，也可以是`uint256`。编译器不知道该调用哪个函数，因此会报错。

