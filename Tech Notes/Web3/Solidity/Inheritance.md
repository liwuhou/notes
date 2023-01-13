Solidity 中的继承比较多花样，包括简单继承、多重继承，以及[[Constructor & Modifier#Modifier|修饰器(modifier)]]继承和[[Constructor & Modifier#Constructor|构造器(constructor)]]继承。

### 规则
- `virtual`: 父合约中的函数，如果希望子合约重写，需要加上`virtual` 关键字
- `override`: 在子合约中重写了父合约中的函数时，需要加上`override` 关键字
注意：用`override`修饰`public`变量时，会重写与变量同名的`getter` 函数。

```sol
mapping(address => uint256) public override balanceOf;
```

### 简单继承
在 Solidity 中，子合约想要继承父合约，在定义合约的时候，要使用`is`关键字来指明继承的父合约。

```sol
contract Sup {
  event Log(string msg);
  
  function foo public virtual { // 添加 virtual 关键字，让子合约重写
    emit Log('Sup');
  }
}
contract Sub is Sup { // 继承 Sup 合约
  function foo() public override { // 加 override 关键字，指明重写父合约中的方法
    emit Log('Sub');
  }
}
```

### 多重继承
Solidity 中的合约是可以继承多个合约的，但有如下几个规则：
1. 继承时要按辈分由高到低的顺序排列，
  ```sol
  constract Sup {}
  constract Base is Sup {}
  constract Sub is Sup, Base {}
  ```
2. 如果某一个函数在多个继承的合约中都存在，那么在子合约里必须要重写，否则会报错
3. 重写在多个父合约中都重名的函数时，`override`关键字后面要加上所有父合约的名字，例如 `override(Sup, Base)`

e.g:
```sol
contract Sup {
  Event Log(string msg);

  function foo() public virtual {
    Emit Log('Sup');
  }
}

contract Base is Sup {
  function foo() public virtual override {
    Emit Log('Base');
  }
}

contract Sub is Sup, Base {
  function foo() public override(Sup, Base) { // 必须重写
    Emit Log('sub');
  }
}
```

### 修饰器的继承
Solidity 中的[[Constructor & Modifier#Modifier|修饰器(modifier)]]也是可以继承到的，使用的时候，用法与函数继承类似，在相应的地方加`virtual` 和 `override`即可

```sol
contract Base1 {
    modifier exactDividedBy2And3(uint _a) virtual {
        require(_a % 2 == 0 && _a % 3 == 0);
        _;
    }
}

contract Identifier is Base1 {

    //计算一个数分别被2除和被3除的值，但是传入的参数必须是2和3的倍数
    function getExactDividedBy2And3(uint _dividend) public exactDividedBy2And3(_dividend) pure returns(uint, uint) {
        return getExactDividedBy2And3WithoutModifier(_dividend);
    }

    //计算一个数分别被2除和被3除的值
    function getExactDividedBy2And3WithoutModifier(uint _dividend) public pure returns(uint, uint){
        uint div2 = _dividend / 2;
        uint div3 = _dividend / 3;
        return (div2, div3);
    }
}
```

`Identifier`合约可以直接在代码中使用父合约中的`exactDividedBy2And3`修饰器，也可以利用`override`关键字重写修饰器：

```sol
modifier exactDividedBy2And3(uint _a) override {  
  _;  
  require(_a % 2 == 0 && _a % 3 == 0);  
}
```

### 构造函数的继承
子合约中有两种方法继承父合约中的[[Constructor & Modifier#Constructor|构造器(constructor)]]。

假如父合约`Sup`中有一个状态变量 `sup`，由构造函数传参的形式来初始化赋值，

```sol
abstract contract Sup {
  uint public sup;

  constructor(uint _sup) {
    sup = _sup;
  }
}
``` 

这里使用了[[Abstract & Interface#abstract|abstract]]关键字，用来说明这个智能合约是一个抽象合约。
那么继承的时候有这么两种方法：

1. 在声明继承父合约时，声明父构造函数的参数
  ```sol
  constract Sub is Sup(1) {}
  ```
2. 在子合约的构造函数中声明构造函数的参数
  ```sol
  constract Sub is Sup {
    constructor(uint _num) Sup(_num) { // 直接这里调用
    
    }
  }
  ```

### 调用父合约中的函数
子合约有两种方式调用父合约中的函数，直接调用和利用`super` 关键字
1. 直接调用：子合约可以直接用`父合约名.函数名()`的方式调用父合约的函数
  ```sol
  function callParent() public {
    Sup.foo();
  }
  ```
2. `super`关键字：子合约利用`super.函数名()`来调用**最近的父合约函数**，Solidity 继承关系的亲密程序是按声明时从右到左的顺序，所以最近的父类就是声明时最右的父类。
  ```sol
  constract Sub is A, B, C, D {
    function callSuper() public {
      super.foo(); // 调用的是 D 中的 foo 方法。
    }
  }
  ```

### 钻石继承
在面向对象编程中，钻石继承（菱形继承）指一个派生类同时有两个或两个以上的基类。

在多重+菱形继承链条上使用`super`关键字时，需要注意的是使用`super`会调用继承链条上的每一个合约的相关函数，而不是只调用最近的父合约。

我们先写一个合约`God`，再写`Adam`和`Eve`两个合约继承`God`合约，最后让创建合约`people`继承自`Adam`和`Eve`，每个合约都有`foo`和`bar`两个函数。

```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

/* 继承树：
  God
 /  \
Adam Eve
 \  /
people
*/

contract God {
    event Log(string message);

    function foo() public virtual {
        emit Log("God.foo called");
    }

    function bar() public virtual {
        emit Log("God.bar called");
    }
}

contract Adam is God {
    function foo() public virtual override {
        emit Log("Adam.foo called");
        Adam.foo();
    }

    function bar() public virtual override {
        emit Log("Adam.bar called");
        super.bar();
    }
}

contract Eve is God {
    function foo() public virtual override {
        emit Log("Eve.foo called");
        Eve.foo();
    }

    function bar() public virtual override {
        emit Log("Eve.bar called");
        super.bar();
    }
}

contract people is Adam, Eve {
    function foo() public override(Adam, Eve) {
        super.foo();
    }

    function bar() public override(Adam, Eve) {
        super.bar();
    }
}

```

在这个例子中，调用合约`people`中的`super.bar()`会依次调用`Eve`、`Adam`，最后是`God`合约。

虽然`Eve`、`Adam`都是`God`的子合约，但整个过程中`God`合约只会被调用一次。原因是Solidity借鉴了Python的方式，强制一个由基类构成的DAG（有向无环图）使其保证一个特定的顺序。更多细节你可以查阅[Solidity的官方文档](https://solidity-cn.readthedocs.io/zh/develop/contracts.html?highlight=%E7%BB%A7%E6%89%BF#index-16)。