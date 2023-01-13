### Constructor

在 Solidity 中存在着这类特殊的函数，像 构造函数（`Constructor`)在每个合约中可以定义一个，且只有在部署合约的时候会自动运行一次。可以用它来初始化一些参数，例如初始化合约的 `owner`地址

```sol
address owner;

constructor() {
  owner = msg.sender;
}
```

像 [[Constant & Immutable#Constant|constant]] 声明的变量就不可在构造函数中初始化，但  [[Constant & Immutable#Immutable|immutable]] 声明的变量可以。

```sol
constructor() {
  uint immutable _number = 0; // work
  uint constant _num = 1; // throw an error
}
```

需要注意的是，在旧版本的 Solidity(`<=0.4.22`)中，构造函数不以 `constructor` 来命名，而是使用与当前合约同名的函数作为构造函数。

```sol
pragma solidity =0.4.21;  
contract Parents {  
  // 与合约名Parents同名的函数就是构造函数  
  function Parents () public {  
  }  
}
```


### Modifier
`modifier` 是 Solidity 特有的语法，类似于面向对象编程中的 `decorator` 概念，声明函数拥有的特性，并减少冗余。`modifier`主要的使用场景是运行函数前的检查，诸如地址、变量、余额等。

以下是一个定义只有合约 Owner 才能调用的 `modifier`。

```sol
modifier onlyOwner {
  require(msg.sender == owner, "You can't change Owner!!!!!"); // 检查调用者是否为 owner 地址
  _; // 如果是的话就继续运行函数主体，否则报错并 revert 交易
}
```

带有 `onlyOwner` 修饰符的函数只能被 owner 地址调用，比如：

```sol
function changeOwner(address _newOwner) external onlyOwner {
  owner = _newOwner; // 只有 owner 地址运行这个函数，并改变 owner
}
```

当非合约的 Owner 调用函数的时候，就会因为没有权限而被 revert。

![](http://cdn.liwuhou.cn/tmp/20221226180858.png)

### 额外资料
- [OppenZepplin实现的Ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol)