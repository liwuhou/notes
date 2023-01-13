在 Solidity 中，以声明了某个变量的时候，即使后续没有赋值动作，Solidity 都会为它赋上默认值。

不同在类型，初始值都不同，基本类型变量的默认值为：
-   `boolean`: `false`
-   `string`: `""`
-   `int`: `0`
-   `uint`: `0`
-   `enum`: 枚举中的第一个元素
-   `address`: `0x0000000000000000000000000000000000000000` (或 `address(0)`)
-   `function`
    -   `internal`: 空白方程
    -   `external`: 空白方程

引用类型的默认值为：
-   映射`mapping`: 所有元素都为其默认值的`mapping`
-   结构体`struct`: 所有成员设为其默认值的结构体
-   数组`array`
    -   动态数组: `[]`
    -   静态数组（定长）: 所有成员设为其默认值的静态数组

```sol
// Reference Types  
uint[8] public _staticArray; // 所有成员设为其默认值的静态数组[0,0,0,0,0,0,0,0]  
uint[] public _dynamicArray; // `[]`  
mapping(uint => address) public _mapping; // 所有元素都为其默认值的mapping  
// 所有成员设为其默认值的结构体 0, 0  
struct Student{  
  uint256 id;  
  uint256 score;  
}  
Student public student;
```


### delete 操作符
Solidity 中提供了一个操作符，可以删除变量中的赋值，让其回归初始值。

```sol
bool public _bool = true;

function delBool() public {
  delete _bool; // 删除之后，_bool 变为默认的 false
}
```