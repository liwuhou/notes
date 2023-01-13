在 Solidity 中，有一类类型可以通过键（key）来查询对应的值（value），比如通过一个用户的 id 来查询他的钱包地址。这个类型本质上就是 hash map。
Mapping type 只能在于[[Layout & Scope#数据位置(Layout)|storage]]中

### 声明的规则
使用 `mapping(Key type => Value type)` 格式来定义映射关系。

```sol
mapping(uint => address) public id2Addr; // 用户 id 到钱包地址的映射
mapping(address => address) public swapPair; // 币对的映射，地址到地址
```

### 映射的规则
1. 映射的 `Key type` 只能使用 Solidity 中默认的类型，比如`uint`，`address` 等，不能使用自定义的结构体。而 `Value type` 可以使用自定义的结构体。
   ```sol 
     struct Person {
       string name;
       uint8 age;
     }
     mapping(Student => uint) public testVal; // Error
   ```
2. 映射的存储位置必须是 `storage`，因此可以用于合约的状态变量，函数中的 `storage` 变量，和 **library** 函数的参数。不能用于 `public`函数的参数或返回结果中，因为 `mapping`记录的是一种关系（key-value pair）
3. 如果映射声明为 `public`，那么 Solidity 会自动为你创建一个 `getter` 函数，可以通过 `key` 来查询对应的 `Value`
4. 给映射新增的键值对的语法为 `_Map[_Key] = _Value;`，其中 `_Map` 是映射变量名，`_Key` 和 `_Value` 对应新增的键值对。
```sol
function writeMap(uint _key, address _value) public {
  id2Addr[_key] = _value;
}
```

### 映射的原理
1. 映射不会储存任何的键信息，也没有 length 信息
2. 映射使用 `keccak256(key)` 哈希算法的结果当做 offset 来存取 value
3. 因为 Ethereum 会定义所有未使用的空间为 0，所以未赋值（Value）的键（Key）初始化为 0