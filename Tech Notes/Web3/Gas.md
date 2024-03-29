由于在链中存储的空间有限，为了让你的交易被写在某个区块中，你需要支付手续费，由此产生*gas*。
当我们发起一笔交易的时候，会支持一小部分以太坊或者其实区块链的代币，以补偿矿工（ miners）和验证者（validators）他们提供的算力。自然地，这些奖励会激励人们运行节点，而这些人的收入就由 gas 的使用量来决定。

而 *gas* 就是一个这样的计量单位，用来计量一笔交易所耗费的计算机资源，当你的交易使用到越多的资源，就需要支付更多的*gas*。

### 常见的消耗*gas*的操作
- 转移代币，消耗的*gas*较少
- 铸造 NFT，消耗的*gas*多
- 向某个 Defi 存钱，消耗*gas*多

每个区块链都有计算*gas*的方式，我们一般只关心*gas*的价格（Gas Price）和交易的手续费(Transaction Fee）。
当区块链越繁忙的时候，说明没有足够的资源（运算节点）去处理所有的交易，因此每个操作花费的*gas*就会越多。

### Gas Brunt & Txn Saving Fees
在 Gas Fees 中有 Base、Max、 和 Max Priority。
Base Fee 是每个*gas*消耗掉的基础费用，这个 base 乘以*gas*的使用数量就是 Gas Brunt。这部分代币会被“烧掉”，永远无法在链上流通。
Max 是我们愿意为这次操作所付的*gas*的最大费用。
Max Priority 是给矿工的小费，能促使矿工更快的打包区块，记录交易。

也就是说，每次发起交易的时候，都要支付一部分手续费，而手续费中有一部分是被烧掉了，另一部分则是直接给了矿工。
故付给矿工的费用等于 `Transaction Fee - Brunt Fee`。

