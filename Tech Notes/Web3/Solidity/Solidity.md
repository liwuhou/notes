
Solidity 中的常用的变量有几种类型：
1. [[Value Type|数值类型(Value Type)]]: 包括布尔类型，整型等等，这类变量赋值时直接传递数值。[文档🔗](https://docs.soliditylang.org/en/latest/types.html#value-types)
2. [[Function Type 1|函数类型(Function Type)]]: [Solidity 文档里](https://docs.soliditylang.org/en/latest/types.html#function-types)将函数照到数值类型，但 Solidity 里函数的复杂性，我觉得单独开个类型会更好梳理，同时函数也支持[[Function Type#函数重载]]的特性
3. [[Reference Type|引用类型(Reference Type)]]: 包括数组和结构体，这类变量占用的空间大，在赋值时会直接传递地址（指针）
4. [[Mapping Type|映射类型(Mapping Type)]]: **Solidity**里的哈希表

Solidity 中的变量一旦以某种类型声明，就会赋上[[Base Value|初始值(Base Value)]]，不同的类型会有不同的初始值。并且，被[[Base Value#delete 操作符|delete]]操作符操作过的变量不会被删除，变量依然在，只是回归了默认值。

Solidity 也可以声明[[Constant & Immutable|常量(Constant)]]，与其它编程语言不同的是。Solidity 中还有 [[Constant & Immutable|不可变量(immutable)]]的概念。

Solidity 中的[[Event]]是 EVM 上日志的抽象，它不仅响应方便还很经济。链上存储的变量至少需要消耗 `20,000`个[[Gas]]，而 EVM 上的事件，每个大概才消耗 `2,000` 个 *gas*。

在面向对象的语言中，继承是很重要的特性，它可以显著地减少重复代码。如果把合约看成是对象的话，Solidity 也是一门支持面向对象的语言，也是支持[[Inheritance|继承]]的。同时也提供了[[Abstract & Interface#abstract|抽象合约(abstract)]]和[[Abstract & Interface#interface|接口(interface)]]来约束和描述一个合约类型。

写智能合约经常会出 bug，Solidity 中的[[Error]]命令可以帮助我们 debug。

Solidity 中有两种特殊的回调函数，一种是用来[[transfer() & send() & call()#接收ETH]] 的 [[receive & fallback#接收 ETH 的函数 —— receive|receive]]，一种是用来处理合约中不存在的函数调用（代理合约 proxy contract）的[[receive & fallback#回退函数 —— fallback|fallback]]。