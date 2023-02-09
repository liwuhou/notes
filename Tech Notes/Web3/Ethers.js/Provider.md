`Provider` 类是对以太坊网络连接的抽象单元，能为标准以太坊节点功能提供简洁、一致的接口。在[[Ethers]]中，`Provider` 不接触用户私钥，只对链上数据进行读取，而不能写入，因此比[web3.js](https://web3js.readthedocs.io)要安全。