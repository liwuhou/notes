### 接收ETH
接收 ETH 可以使用[[receive & fallback|receive 和 fallback函数]]，接收的 ETH 会存于合约的地址中。

### 发送 ETH
发送 ETH 有三种方法，`transfer`、`send`和`call`，其中`call`是推荐使用的方法。

#### transfer
- 用法是 `{接收地址}.transfer({发送的 ETH 数额})`
- `transfer`的 gas 限制是`2300`，足够用于转账的了，但如果对方合约的[[receive & fallback#接收 ETH 的函数 —— receive|receive]]或[[receive & fallback#回退函数 —— fallback|fallback]]函数实现了太复杂的逻辑的话，就会导致转账失败
- `transfer`如果转账失败，会自动 [[Error#revert|revert]]（回滚交易）
- `{发送 ETH 数额}`不能小于 0，否则也是会触发 `revert`

```sol
// SPDX-License-Identifier: MIT
pragma solidity ^8.0.7;

contract transfer {
  function transferETH(address payable _to, uint256 amount) external payable {
    _to.transfer(amount);
  }
}
```

#### send
用法基本同 `transfer`，也是 `{接收地址}.send({发送 ETH 数额})`，不同的地方在于，如果 `send` 转账失败，是不会 `revert`的。然后 `send` 返回值是一个 [[Value Type#布尔类型]]，表示转账的成功与否，需要额外的代码进行处理。

```sol
function sendETH(address payable _to, uint256 amount) external payable {
  bool success = _to.send(amount);
  if (!success) {
    revert SendFail();
  }
}
```

#### call
- `call`的用法是 `{接收地址}.call{value: {发送 ETH 数额}}("")`
- `call`没有[[Gas]]限制，可以支持对方合约中的 `fallback`或`receive`实现复杂的逻辑
- `call`如果转账失败，不会 `revert`
- `call` 的返回值是 `(bool, data)`，其中 `bool` 代表着转账的成功与否，需要额外的代码进行异常处理。

```sol
function callETH(address payable _to, uint256 amount) external payable {
  (bool success, ) = to.call{value: amount}("");

  if(!success) {
    revert CallFaild();
  }
}
```

### 总结
Solidity 中有三种发送 ETH 的方法，其中`call` 没有[[Gas]]的限制，最为灵活，也是最为提倡的方式。而 `transfer`和`send`都有 `2300` gas 的限制，但`transfer`发送失败会自动 `revert`，而 `send` 发送失败不会自动 `revert`交易，需要手动去定义。