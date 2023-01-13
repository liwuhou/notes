Solidity 中有两个重要的关键字——`constant` 和 `immutable`，用这两个关键字声明的变量，不能在部署合约后更改数值，同时也会节省一些[[Gas]]。

### Constant
`constant` 变量**必须在声明的时候就初始化**，之后再也不能改变，尝试改变的话，编译不会通过。

```sol
// constant 变量必须在声明的时候就要初始化并赋值，并在之后不能被改变
uint256 constant CONATANT_NUM = 10;
string constant CONSTANT_STRING = '0xAA';
bytes constant CONSTANT_BYTES = "william";
address constant CONSTANT_ADDRESS = 0x0000000000000000000000000000000000000000;
```

### Immutable
而 `immutable` **变量可以在声明时或[[Constructor & Modifier#Constructor|构造函数]]中初始化**，之后就再也不能改变。否则也是会报错。与 `constant` 不同的地方在于，`immutable` 可以通过函数的返回值为初始化。

```sol
uint256 constant CONSTANT_NUM = 1; // correct
// uint256 constant CONSTANT_NUM = getNum(); // incorrect
// adress constant CONSTANT_ADDRESS = address(this); // incorrect

uint256 public immutable IMMUTABLE_NUM = geetNum(); // correct
address public immutable IMMUTABLE_ADDRESS = address(this); // correct

function getNum() public pure returns(uint256 _num) {
  _num = 10;
}
```