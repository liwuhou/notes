当我们调用智能合约的时候，本质上是向目标合约发送了一段 `calldata`，在 [remix](remix.ethereum.org)上发送一次交易的时候，可以在详细信息中看见 `input`即为此次交易的 `calldata`

发送的 `calldata` 中前 4 个字节是 `selector` （函数选择器）。

### `msg.data`
`msg.data` 是 Solidity 中的一个全局变量，值为完整的`calldata`（调用函数时传入的数据）。

可以通过 `Log` 事件来输出调用 `mint` 函数的 `calldata`

```sol
event Log(bytes data);

function mint(address to) external {
  emit Log(msg.data)
}
```

当参数为`0x2c44b726ADF1963cA47Af88B284C06f30380fC78`时，输出的`calldata`为

```
0x6a6278420000000000000000000000002c44b726adf1963ca47af88b284c06f30380fc78
```

这段很乱的字节码可以分成两部分：

```
前4个字节为函数选择器selector：0x6a627842后面32个字节为输入的参数：0x0000000000000000000000002c44b726adf1963ca47af88b284c06f30380fc78
```

其实`calldata`就是告诉智能合约，我要调用哪个函数，以及参数是什么。

### `method id` & `selector`
`method id`定义为[[Function Type#函数签名]]的`Keccak`哈希后的前 4 个字节，当`selector`与`method id`相匹配时，即表示调用该函数。

### 使用`selector`
可以利用 `selector` 来调用目标函数，只需要利用[abi.encodeWithSelector](https://docs.soliditylang.org/en/v0.8.17/units-and-global-variables.html?highlight=abi.encodeWithSelector#abi-encoding-and-decoding-functions)方法，传入函数的`method id`, 然后第二位开始传入调用函数的参数，传给 [call](https://docs.soliditylang.org/en/v0.8.17/types.html#members-of-addresses)`call`函数：

```sol
    function callWithSignature() external returns(bool, bytes memory){
        (bool success, bytes memory data) = address(this).call(abi.encodeWithSelector(0x6a627842, "0x2c44b726ADF1963cA47Af88B284C06f30380fC78"));
        return(success, data);
    }
```

