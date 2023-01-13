[ethers.js](https://docs.ethers.org) 库是一个完整而紧凑的库，旨在提供与以太坊区块链及其生态系统交互能力的库。跟[web3.js](https://web3js.readthedocs.io)相比，具有以下优势：
1. 代码更加紧凑：`ethers.js` 大小为 116.5kb，而`web3.js`为 590.6kb。
2. 更加安全：`web3.js`认为用户会在本地部署以太坊节点，私钥和网络连接状态由这个节点管理；而`ethers.js`中，[[Provider]]提供器类管理网络连接状态，`Wallet`钱包类管理密钥，安全且灵活。
3. 原生支持`ENS`。

![](http://cdn.liwuhou.cn/blog/202301092141400.png)
