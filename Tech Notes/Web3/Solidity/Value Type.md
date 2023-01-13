在 Solidity 中，数值类型包括布尔型，整数型、地址等等，这类变量在赋值的时候是直接传递数值的。

### 布尔类型
布尔型是二值类型，非 `true` 即 `false`

```js
bool public _bool = true;
```

涉及布尔的运算与 Javascript 相同：
- `!` 取反
- `&&` 逻辑与
- `||` 逻辑或
- `==` 等于
- `!=` 不等于

同样的 `&&`和`||` 也有逻辑短路(short-circuiting rule)的特性，这个跟 Javascript 是一样的。

### 整数型

整型即 `Solidity` 的整数，分为有符号（int) 和无符号（uint）两种，并且都有`8`、`16`、`32`、`64`、`128`、`256`几种位大小的数据。`int`和`unit`的默认大小都是`256`。也就是说 `int` 和 `unit` 分别是 `int256` 和 `unit256`的别名。

- `int`： 整数，包括负数，256位
- `unit`：正整数，无符号整型，256 位
- `int8`: 8 位的有符号整型
- `unit8`：8 位的无符号整型

常用的整型运算符包括：
- 比较运算符（返回布尔值）： `<=`, `<`, `>`, `>=`, `!=`, `==`等等
- 算术运算符：`+`，`-`，一元运算符 `-`， `+`， `*`， `/`， `%`和`**`等等
- 位运算符：`<<`， `>>`、`&`，`|`，`^`

### 地址类型
地址类型(address)是一类特殊的数值类型，可以用来存储一个 20 字节的值（以太坊地址的大小）。地址类型也有成员变量，并可作为所有合约的基础。有普通的地址和可以转账 **ETH** 的地址(payable)。而 `payable` 的地址拥有 `balance` 和 `transfer()` 两个成员，方便查阅 **ETH** 余额和发起转账。

```js
// address
address public _address = 0x7A58c0Be72BE218B41C608b7Fe7C5bB630736C71;
address payable = public _address_payable = payable(_address); // It can check the balance and make a tranfer action

uint256 public balance = _address1.balance; // balance of address
```

### 定长字节数组
有点类似 Javascript 中的字符串，字节数组分为两种，一种是定长（`byte`, `bytes8`, `bytes32`），另一种就是不定长的。
定长的字节数组属于数值类型，不定长的是引用类型。定长的`bytes`可以存一些数据，消耗的 **gas** 比较少。

```js
bytes32 public _bytes32 = 'MiniSolidity';
bytes1 public _byte = _bytes32[0];
```

上面的 `MiniSolidity` 变量以字节的方式存储进变量 `_bytes32` ，转换为 16 进制为：`0x4d696e69536f6c69646974790000000000000000000000000000000000000000`
`_byte` 变量存储在 `_bytes32` 的第一个字节，为 `0x4d`。

### 枚举类型
枚举（enum）是`Solidity`中用户定义的数据类型，它主要用于`unit`分配名称，使程序易于阅读和维护。它与 **Typescript** 中的 `enum`类似，使用名称来代码从 `0` 开始的 `unit`

```solidity
enum ActionSet { 
  Buy, Hold, Sell 
}
ActionSet action = ActionSet.Buy;
```

`enum`类型也可以和 `unit` 相互转换，阈值检查转换的正整数是否在枚举的长度内，不然是会报错的。

```solidity
// enum can transfer into unit
function enumToUnit() external view returns(uint) {
  return unit(action);
}
```

> `enum` 是一个比较冷门的特性，几乎没什么人用。