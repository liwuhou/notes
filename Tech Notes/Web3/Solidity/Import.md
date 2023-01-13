Solidity 支持模块化，提供了`import`关键字来导入其它源代码中的合约。
`import` 的位置在声明Solidity版本号之后，在其余代码之前。

### import 语法
借鉴了 js 和 python，可以收入某个文件的全局符号

```sol
import './other.sol';
```

也可以像 js 一样局部引入和改名

```sol
import Other from './other.sol';
import * as Another from './other.sol';
import { Foo } from './other.sol';
import { Foo as Bar } from './other.sol';
```

### 引入相对位置的文件
可以通过文件的相对位置，导入

```sol
import './other.sol';
```

### 引入源文件 URL

```sol
// 通过网址引用
import 'https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol';
```

### 通过 npm 导入

```sol
import '@openzeppelin/contracts/access/Ownable.sol';
```

### 通过全局符号导入特定合约

```sol
import { SpecialContract } from './specialContract.sol';
```