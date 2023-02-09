![](http://cdn.liwuhou.cn/blog/202302052216977.png)

`PeerConnection` 是 WebRTC 中的核心对象，也是一场 WebRTC 通话中的核心载体。

```js
// 尽可能多的兼容不同的浏览器
const PeerConnection = window.RTCPeerConnection ||
      window.mozRTCPeerConnection ||
      window.webkitRTCPeerConnection

```

### PeerConnection 中的核心方法

-   `addIceCandidate()`： 保存 ICE 候选信息，即双方协商信息，持续整个建立通信过程，直到没有更多候选信息。
-   `addTrack()` ：添加音频或者视频轨道。
-   `createAnswer()` ：创建应答信令。
-   `createDataChannel()`： 创建消息通道，建立`WebRTC`通信之后，就可以 `p2p` 的直接发送文本消息，无需中转服务器。
-   `createOffer()`： 创建初始信令。
-   `setRemoteDescription()`： 保存远端发送给自己的信令。
-   `setLocalDescription()` ：保存自己端创建的信令。

以上就是`PeerConnection`这个载体核心驱动的主要方法了，除了这些核心方法之外，还有一些**事件监听函数**，这些监听函数用于监听远程发送过来的消息。

假如 A 和 B 建立连接，如果 A 作为主动方即呼叫端，则需要调用的就是上述**核心方法**去创建建立连接的信息，而 B 则在另一端使用上述**部分核心方法**创建信息再发送给 A，A 则调用**事件监听函数**去保存这些信息。常用的事件监听函数如下：

-   `ondatachannel`： 创建`datachannel`后监听回调以及 `p2p`消息监听。
-   `ontrack` ：监听远程媒体轨道即远端音视频信息。
-   `onicecandidate`： ICE 候选监听。

