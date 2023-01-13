
### abstract
可以使用`abstract` 关键字来声明一个抽象的智能合约，该合约中至少要包含一个未实现的函数，即某个函数缺少主体`{}`中的内容，并且未实现的函数还需要加[[Inheritance#规则|virtual]]关键字，以便子合约重写，否则都会在编译的时候报错。

```sol
abstract contract InsertionSort {
  function insertionSort(uint[] memory array) public pure virtual returns(uint[] memory);
}
```

注意看方法的结果没有`{}`，而是直接以`;`结束。
与接口不同的是，抽象合约中的未实现的方法必须加关键字 `virtual`

值得注意的是，如果一个合约继承了一个抽象合约，但是没有完全实现抽象合约中定义的方法，那么这个合约也要变成一个抽象合约。也就是说，抽象合约具有传染性。

### interface
接口类似于 TS 中的接口，也类似抽象合约，但它不实现任何功能。接口的规则：
1. 不能包含状态变量
2. 不能包含构造函数
3. 不能创建任何描述符
4. 不能继承除接口外的其它合约
5. 所有函数都必须是[[Function Type#internal VS external|external]]，**且不能有函数体**
6. 继承接口的合约必须实现接口定义的所有功能

```sol
interface Token {
    enum TokenType { Fungible, NonFungible }
    struct Coin { string obverse; string reverse; }
    function transfer(address recipient, uint amount) external;
}
```

接口是一类用来描述合约属性和行为的依据。一个合约继承了某个接口之后，如果不实现接口中定义的方法，就会报错，除非合约是抽象合约（抽象合约可以不给某个方法的实现）。

```sol
abstract contract AbInterface is Token {} // work

contract Test is Token {
  function transfer(address recipient, uint amount) public override {
    // 具体实现
  }
}
```

与抽象合约不同的是，接口中定义的方法无须加关键字[[Inheritance#规则|virtual]]关键字，但继承的合约，实现方法的时候要加[[Inheritance#规则|override]]

### 利用 interface 与合约交互

如果我们知道一个合约实现了某个接口的实现，那么使用的时候，可以直接用接口+合约地址生成一个具有对应接口规则的 [[ABI]] 合约。然后调用这个合约中接口约束的变量或方法。
比如我们知道无聊猿`BAYC`  是属于 `IERC721` 的代币，也就是它实现了 `IERC721`接口的功能。我们不需要知道它的源代码，只需知道它的合约地址(`0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D`)，就可以用 `IERC721` 接口来跟它交互。

```sol
contract interactBAYCWithInterface {
  // 利用BAYC地址创建接口合约变量（ETH主网）  
  IERC721 BAYC = IERC721(0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D);
  
  // 通过接口调用BAYC的balanceOf()查询持仓量  
  function balanceOfBAYC(address owner) external view returns (uint256 balance){  
    return BAYC.balanceOf(owner);  
}  
  
  // 通过接口调用BAYC的safeTransferFrom()安全转账  
  function safeTransferFromBAYC(address from, address to, uint256 tokenId) external{  
    BAYC.safeTransferFrom(from, to, tokenId);  
  }
}
```