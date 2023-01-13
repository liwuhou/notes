在 Solidity 中，有一些变量类型属于引用类型，其中最重要的两种引用类型为， 数组（`Array`）和结构体（`struct`）。


## Array
数组是 Solidity 中的一种常用的变量类型，用来存储一组数据（整数、字节、地址等等）。
数组分为固定长度数组与可变长度数组：

- 固定长度数组：在声明的时候就指定数组的长度，用 `T[K]` 的格式声明，其中 `T` 为数组类型，`K`为数组长度，如：

```sol
uint[8] array1;
bytes1[5] array2;
address[100] array3;
```

- 可变长度数组（动态数组）：在声明时不指定数组的长度，用 `T[]` 的格式声明，其中 `T` 是元素的类型，如：

```solidity
uint[] array4;
bytes1[] array5;
address[] array6;
bytes array7;
```

>  `bytes` 比较特殊，他是数组，但是不用加 `[]` 。另外不能用 `byte[]` 声明单字节数组，而是用 `bytes` 或 `bytes1[]`。并且在 *gas* 上，`bytes` 比 `bytes1[]` 便宜。因为 `bytes1[]` 在 `memory` 中要增加 31 个字节进行填充，会产生额外的 *gas*。但在 `storage` 中，由于内在紧密打包，不会存在字节填充的情况。

### Array 创建的规则
- 在`memory`中声明动态数组时，可以使用 `new` 操作符，但是必须声明长度，并且声明后，长度不能改变。

```sol
// memory dynamic array
uint[] memory array8 = new uint[](5);
bytes memory array9 = new bytes(9);
```

- 数组字面量声明，用方括号包着来初始化数组的一种方式，并且每个元素的 type 都是以数组中第一个元素的类型为准，例如 `[1, 2, 3]` 里的所有元素就都是 `uint8` 类型。因为在 Solidity 中，如果一个引用类型的值没有指定类型，默认就是最小单位的 type 为该引用类型的 type。

```sol
function foo() public pure {
  // bar([1, 2, 3]); error
  bar([uint(1), 2, 3]);
}

function bar(uint[3] memory array) {
  // ...
}
```

如果创建的是动态数组，就需要一个一个元素进行赋值

```sol
function test() {
  uint memory x = new uint[](3);
  x[0] = 1;
  x[1] = 2;
  x[2] = 3;
}
```

### 数组成员

- `length`：数组中有一个包含元素数量的属性，`memory`数组的长度在创建后是固定的。
- `push`：动态数组和`bytes`拥有`push`方法，可以在数组最后添加一个元素。如果方法中没有传值，则会在数组末尾追加一个 `0` 元素。
- `pop`：动态数组和`bytes`拥有`pop`方法，可以移除数组末尾最后一个元素。

## Struct
Solidity 支持通过构造结构体的形式定义新的类型，创建结构体的方法：

```sol
struct Person {
  string name;
  uint8 age;
}

Person william; // 初始化一个 Person 的结构体
```

结构体的赋值的方法

```sol
Person william;
william.name = "william";
william.age = 18;

Person abby = Person("abby", 18);

Person skye = Person({name: "skye", age: 18}); // better
```