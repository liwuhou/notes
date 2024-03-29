两个对等的端在建立 WebRTC 通话的时候，需要彼此知道对方端的信息。而信令服务器就是串起这两个端的媒介，本质上就是一个运行着即时通讯协议的服务器，可以通过一个 `Websocket` 服务器来完成信令服务器的使命。

下面就是针对 WebRTC 会话过程中所需的转发逻辑，倒推出来服务器应该具备的哪些功能。

![](http://cdn.liwuhou.cn/blog/202302080817332.png)

如果要实现一对多的会话，就需要服务器维护一些会话者之间的集体信息，如房间 ID 。由此，可以采用一些数据结构来关联集体信息与个人信息，这里用到了 Redis 的一种数据结构 Hash，存放的大体结构如下：

![](http://cdn.liwuhou.cn/blog/202302080836693.png)

### 使用 Node.js 来实现信令服务器

