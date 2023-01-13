**Proof of Work** 简称PoW，翻译为工作量证明，是一个用来描述区块链网络中某台矿机的工作量的机制。在了解区块链中的工作量证明里，先了解了下 [[Hash]]和[[Blockchain#Puzzle Friendly|Puzzle Friendly]]。

当一个矿机（miner）将最近产生的交易打包成区块追加到区块链的尾部时，

比特币中的工作量证明，由于**Puzzle friendly** 的特性，区块链网络中的所有矿机为了找到这么一个满足条件的 nonce，只能通过穷举。再加上找到这个数虽然很不容易，但却很容易验证，其它矿机只要验证上述的等式成立就可以了（这也是挖矿中的 Difficult to solve, but easy to verify 的特性）。

