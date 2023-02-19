在 WebRTC 会话中，浏览器以 [[WebRTC PeerConnection]] 为载体，建立起 `P2P`连接，而两个浏览器是如何建立关联关系就涉及到了与信令服务器的交互。大致上两个不同的浏览器是这么建立连接的。

![](http://cdn.liwuhou.cn/blog/202302052228185.png)

图中 A 为呼叫端（Caller），B 为被呼叫端（Callee）。

1. 首先 A 呼叫 B，这个呼叫的过程，一般使用 `WebSocket` 通信，让对方收到信息。
2. B 接受应答，A 和 B 均开始初始化 [[WebRTC build connection|PeerConnection]] 实例，用来关联 A 和 B 的 SDP 会话信息。
3. A 调用 `createOff` 创建信令，同时通过 `setLocalDescription` 方法在本地 `PeerConnection` 实例中储存一份（即图中流程 ①）
4. 然后调用信令服务器将 A 的 SDP 转发给 B（图中流程 ②）
5. B 接收到 A 的 SDP 后调用 `setRemoteDescription`，将其储存在初始化完成的 `PeerConnection` 实例中（图中流程③）
6. B 同时调用 `createAnswer` 创建应答 SDP，并调用 `setLocalDescription` 储存在自己本地的 `PeerConnection` 实例中（图中流程④）
7. B 继续将自己创建的应答 SDP 通过服务器转发给 A（图中流程⑤）
8. A 调用 `setRemoteDescription` 将 B 的 SDP 储存在本地 `PeerConnection` 实例中（图中流程⑥）
9. 在会话的同时，图中有 `ice candiate`，这个信息就是 ice 候选信息，A 发给 B 的 B 储存，B 发给 A 的 A 储存，直到候选完成。

> SDP实际上就是 WebRTC 会话的信令。
> 完成了以上过程就相当于建立了 WebRTC 的会话基础，可以利用这个桥梁去添加和监听双方的音视频信息。



