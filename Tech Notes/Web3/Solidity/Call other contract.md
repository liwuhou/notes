我们可以在智能合约中调用已经部署的其它合约的方法，复用以太坊网络中的其它程序，在 web3 中，很多项目都依赖于调用其它合约。我们可以在已知合约代码（或接口）和地址的情况下，调用目标合约的函数。

### 四种调用其它合约中的函数的方法
首先要创建一个合约的引用（或称实例），可以通过 `_ContractName(_Address)` 来创建，其中 `_ContractName` 是合约名，`_Address` 是合约地址。然后就可以通过这个合约的引用来调用它的函数了。

### 传入合约地址
可以通过向合约（或者[[Abstract & Interface#interface|接口]]）传入目标合约地址，生成目标合约的引用，然后调用目标合约中的函数。

```sol
import './OtherContract.sol';

contract Test {
  function callOtherContractFn(address _address) external {
    OtherContract(_address).fn(); // 引入合约，传入目标合约的地址，生成引用
  }
}
```

### 传入合约变量
直接传入合约的引用，然后调用其方法

```sol
import './OtherContract.sol';

contract Test {
  function CallOtherContractFn(OtherConTract _address) external view returns(uint x) {
    x = _address.fn();
  }
}
```

### 创建合约的实例
通过创建合约实例，然后通过它来调用目标函数。其实跟第一种方式是一样的，只不过将返回的实例保存在变量中。

```sol
import './OtherContract.sol';

contract Test {
  function CallOtherContractFn(address _address) external view returns(uint x) {
    OtherContract oc = OtherContract(_address);
    x = oc.fn();
  }
}
```

### 调用合约并发送 ETH
如果目标合约的函数是 [[Function Type#payable|payable]]，那么我们可以通过调用它来给合约转账：`_ContractName(_Address).f{value: _Value}()`，其中 `f` 是函数名，`_Value` 是转账的 ETH 数额（单位是[[ETH uint#wei|wei]]）。

```sol
function SetOtherContractFn(address otherContract, uint256 x) payable external {
  OtherContract(otherContract).setX{value: msg.value}(x);
}
```
