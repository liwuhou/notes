ABI 全称 Application Binary Interface，顾名思义就是一种在以太坊系统中描述某个合约属性和方法的接口。透过合约中暴露的 ABI，我们可以与合约进行交互。任何合约部署的时候都会生成对应的 ABI。

### 与 Interface 等价
合约 ABI 与[[Abstract & Interface#interface|interface]]等价，利用[abi-to-sol工具](https://gnidan.github.io/abi-to-sol/)也可以将`ABI json`文件转换为`接口sol`文件。
