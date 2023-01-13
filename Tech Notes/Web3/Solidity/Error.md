在 Solidity 0.8.4版本中，新增了 `error`类型。可以让我们方便且高效地向用户解释操作失败的原因，同时在抛出异常的同时携带参数，帮助开发者更好地高度。

一个异常可以`contract`之内定义，也可以在`contract`之外定义。

```sol
error TranferNotOwner();
```

或者携带一个参数，暴露更多信息

```sol
error TransferNotOwner(address sener);
```

### revert
在执行当中，`error`必须搭配`revert`（回退）命令一起使用。相当于 js 中的 `throw`。

```sol
function transferOwner(uint256 tokenId, address newOwner) public {
  if (_owners[tokenId] != msg.sender) {
    revert TransferNotOwner(newOwner);
  }

  _owners[tokenId] = newOwner;
}
```

`revert` 不仅可以接着自定义错误回滚，也可以以方法调用，传入一个字符串`revert("description")`。

```sol
function transferOwner(uint256 tokenId, address newOwner) public {
  if (_owners[tokenId] != msg.sender) {
    revert("Transfer not Owner!");
  }
  // 上面等价于
  require(_owners[tokenId] != msg.sender, "Transfer not Owner!")
}
```

当然 `revert` 也不只是抛出错误，他其实相当于回滚，会返回一些未消耗的*gas*。拿下面的代码举例。在 `revert`之前的*gas*消耗了就消耗了，虽然 `number` 因为回滚并不会改成 5，而是依然是 0。但改变 `number`时消耗的*gas*不会返回。但是`revert`语句后面还未执行的操作，也就是未消耗*gas*，就会原路返回。

```sol
uint256 public number;

function test() public {
  number = 5;
  require(condition, "expection"); // 如果从这里 revert，number 的数值虽然不会改变，但是*gas*已经被消耗了。
  // 后面有个消耗大量*gas*的操作，没有执行就会原路返回
}
```

### require
`require` 命令是 Solidity 0.8.0 版本之前常用的抛出异常的方法，目前很多主流合约仍然还在使用这种语法。这种方法方便是方便，它唯一的缺点就是[[Gas]]会随着描述异常的字符串长度的增加而增加，费用会比`error`命令高很多。使用方法：`reuqire(检查条件，"异常的描述")`，当检查条件不成立的时候，就会抛出异常。

```sol
function transferOwner(uint256 tokenId, address newOwner) public {
  require(_owners[tokenId] == msg.sender, "Transfer not Owner!");
  _owners[tokenId] = newOwner;
}
```

### assert
`assert`一般用于调试，因为它不能解释程序出异常的原因。用法也比较简单，`assert(检查条件)`，当条件不成立的时候就抛出一个异常。

```sol
function transferOwner(uint256 tokenId, address newOwner) public {
  assert(_owners[tokenId] != msg.sender);
  _owners[tokenId] = newOwner;
}
```

### 三种抛错方法的*gas*比较
使用 0.8.17 版本编译
1. `error`方法消耗的 *gas* 最少
2. `require`方法消耗的*gas*最多
3. `assert`方法消耗的*gas*在 `error` 和 `require`之间。

`error`既可以告知用户抛出异常的原因，又能节省*gas*，因此建议多用。
而`require` 的方式，随着异常描述文字的增多，*gas*会越来越多。
另外在 Solidity 0.8.0 之前的版本， `assert` 抛出的一个 `panic exception`，会把剩余的 *gas* 都消耗完，并且不会返还，因此旧版本要避免使用。更多细节见[官方文档](https://docs.soliditylang.org/en/v0.8.17/control-structures.html)。