区块链是一个不断增长的全网总账本，每个完全节点都拥有完整的区块链，并且节点总是信任最长的区块链，假如想要伪造篡改区块链的话，就要拥有全网超过 51%的算力才行 。

区块链由一个一个的区块构成，每一个区块上都记录了一系列的交易，并且每个区块都指向前一个区块，从而形成了一个链条：
![](http://cdn.liwuhou.cn/blog/20221206234509.png)

每一个区块中，都有一个唯一的哈希标识，被称为区块哈希，同时，区块通过记录上一个区块的哈希来指向上一个区块：
![](http://cdn.liwuhou.cn/blog/20221206234555.png)

**每一个区块中还有一个 `Merkle` 哈希来确保该区块的所有交易记录无法被篡改**。这就保证了区块链很难被篡改，想要成功地篡改掉区块链，就要获取全网 51%以上的算力，让这 51%中的每个节点都信任同一个最长被篡改的区块链。


### 区块链的不可篡改性

区块链是一个不断增长的全网总账本，每个完全节点都拥有完整的区块链，并且节点总是信任最长的区块链，假如想要伪造篡改区块链的话，就要有全网超过 51%的算力才行 。
区块链的不可篡改特性正是由哈希算法来保证的。

区块链中的主要数据就是一系列的交易，通常第一条交易都是 `Coinbase`交易，也就是矿工的挖矿奖励，后续的交易就都是用户交易了。

### 哈希算法
哈希算法又称散列算法，指的是通过一个单向函数，可以把任意长度的输入数据转化为固定长度的输出：
$$ h = H(x) $$

```js
H("morning") = c7c3169c21f1d92e9577871831d067c8
H("bitcoin") = cd5b1e4947e304476c788cd474fb579a
```

> 通常使用十六进制来表示哈希输出

 由于哈希是一个单向的函数，要设计一个安全的哈希算法就要满足：通过输入可以很容易地计算出输出，但反过来是无法通过输出来反推输入的，只能通过暴力穷举。
 
```js
H("???????") = c7c3169c21f1d92e9577871831d067c8
H("???????") = cd5b1e4947e304476c788cd474fb579a
```

另引出一个问题，设计一个安全的哈希算法还需要满足另一个条件：碰撞率低。

### Puzzle Friendly
指为了得到某一个 hash 映射后的数值的话，只能通过暴力穷举的方式，没有其它的途径。

### 哈希碰撞

哈希碰撞是指两个输入的数据不同，却恰好计算出了相同的哈希值，这种现象就称为碰撞。
本质上，发生因为输入的集合是无限的，通过一定的映射（哈希算法）之后输出了有限的集合，根据戈农原理，很显然，发生碰撞是必然的。从上述的定义中也可以确定，一个哈希算法的安全程度，跟他输出的位数有关，位数超高，输出的集合就越大，碰撞的概率就越低，该哈希算法就越安全。

安全的哈希算法不是说碰撞不存在，而是没有什么高效的方法去人为地制造哈希碰撞。对于 $2^{256}$位的哈希值去人为地制造碰撞的话，这个计算量是非常地巨大的。

### 利用哈希算法快速确定数据是否被篡改

假定我们相信一个安全的哈希算法，如果两个输入得到了一个相同的哈希值，那么我们就可以说这两个输入是相同的。在上传文件的场景中，我们经常会遇到判断用户上传的文件是否跟服务器中的文件相同。我们就可以通过两个文件的哈希值是否一样来判断是否属于同一个文件。我们从网站上下载一个耗时非常大的文件时，也可以通过比较下载下来文件的哈希，跟下载网站上提供的文件哈希做对比，确定下载途中是否被篡改。

和文件类似，两个数据的哈希相同，即可认定两者是相同的。比特币使用哈希算法来保证所有交易都是不可修改的，就是计算并记录交易的哈希，如果交易被篡改，那么哈希验证将无法通过，这个区块就是无效的。

### 常用的哈希算法

|哈希算法|输出长度（bit）|输出长度（字节）|
|---|:---|:---|
|MD5|128| 16 |
| RipeMD160| 160 | 20|
|SHA-1|160|20|
|SHA-256|256|32|
|SHA-512|512|64|

比特币使用的哈希算法种类有两种：`SHA-256` 和 `RipeMD160`

其中，`SHA-256`的理论碰撞概率是：尝试 $2^{130}$ 次的输入，有 99.8%的概率碰撞。$2^{130}$ 是一个非常大的数字，以现有的计算机的计算能力，是不可能在短期内破解的。

### 比特币使用的两种哈希算法方案

1. 对数据进行两次 `SHA-256` 计算 
这种算法在比特币协议中通常被称为 **hash256** 或者 **dhash**

```js
const bitcoin = require('bitcoinjs-lib')
const createHash = require('create-hash')

const standardHash = (name, data) => {
  let h = createHash(name, data)
  return h.update(data).digest()
}

const hash256 = (data) => {
  return standardHash('sha256', standardHash('sha256', data))
}
```

2. 对数据先计算一次 `SHA-256` 再计算一次 `RipeMD160`，这种算法在比特币协议中被称为 **hash160**

```js
const hash160(data) {
  return standardHash('ripemd160', standardHash('sha256', data))
}
```

### Merkle Hash
在区块的头部，有一个 `Merkle Hash` 字段，它记录了本区所有交易的 `Merkle Hash`：

![](http://cdn.liwuhou.cn/blog/20221207073744.png)

`Merkle Hash` 是把一系列的数据的**哈希**根据一个简单算法变成一个汇总的哈希
假设区块内有 4 个交易，对每个交易的数据做 dhash 处理，会得到 4 个哈希值 `a1`，`a2`，`a3`和`a4`：

```js
a1 = dhash(tx1)
a2 = dhash(tx2)
a3 = dhash(tx3)
a4 = dhash(tx4)
```

因为哈希值也可以看成是数据，所以可以把 `a1` 和 `a2` 拼起来，`a3` 和 `a4`  拼起来，再计算出两个哈希值 `b1`和 `b2`

```js
b1 = dhash(a1 + a2)
b2 = dhash(a3 + a4)
```

![](http://cdn.liwuhou.cn/blog/202212072219833.png)

再把 `b1` 和 `b2` 这两个哈希值拼起来，再计算一遍 dhash 得出最终的 hash，这个 hash 就是 Merkle Hash：

![](http://cdn.liwuhou.cn/blog/202212072221115.png)

总而言之凑成 Merkle Hash 是需要有最终的两个哈希拼起来后，再过一遍 dhash 的。如果有单数，就把最后一份数据复制一次去计算。

例如只有三个交易的话，就是这样

```js
b1 = dhash(a1 + a2)
b2 = dhash(a3 + a3) // a3 自己拼接了自己
```

五个也是同理的

```js
b1 = dhash(a1 + a2)
b2 = dhash(a3 + a4)
b3 = dhash(a5 + a5)

c1 = dhash(b1 + b2)
c2 = dhash(b3 + b3)

MerkleHash = dhash(c1 + c2)
```

所以从 Merkle Hash 的计算方法可以看出来，修改任意一个交易，哪怕是一个字节，或者交换两个交易的顺序，都会导致 Merkle Hash 验证失败，也就会导致这个区块本身无效，所以，Merkle Hash 记录在区块的头部，它的作用就是保证交易记录是永远不会被修改的。

### Block Hash
区块本身用 Block Hash——也就是区块哈希来标识。但是一个区块自己的区块哈希并没有记录在区块头部，而是通过计算区块头部的哈希得到的。

![](http://cdn.liwuhou.cn/blog/202212072230127.png)

区块头部的 Prev Hash 记录了上一个区块的 Block Hash，这样可以通过 Prev Hash 来追踪上一个区块。区块链就是通过这样的链条串联起来形成的。

区块链的第一个区块也称为创世区块，它并没有上一个区块，因此他的 Prev Hash 被设置为 `000000……0000`（256 位 0）。

可以看到，由于区块链的特殊性，修改一个区块的成本是相当高的，当有攻击者修改了一个区块之后，会导致这个区块的 Block Hash 变化，然后后续所有的区块都要全部重新设计才能伪造出来，才能修改整个区块链。随着时间的推移，区块链的区块会不断的增加，修改一个区块的难度也会随着时间不断的增加。另外攻击者也要掌握全网 51%以上的算力，这些难度的叠加让区块链被伪造显得格外的难。

已经产生的区块内的交易记录是固定不变的，新产生的交易记录会在新区块进行打包，所以，也就是说，通过校验的区块（最终能访问到创世区块的区块）的 Merkle Hash 和 Block Hash 是不会改变的。

### 总结
区块链的安全建立在哈希算法基础上，通过哈希算法保证了所有的区块数据都不可更改。
交易数据依靠 Merkle Hash 来确保不会被修改，整个区块链依靠 Block Hash 确保区块无法修改
工作量证明机制（挖矿）保证修改区块链的难度非常地巨大从而无法实现被攻击。
