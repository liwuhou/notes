服务器向浏览器推送信息，除了 `Websocket`，还有一种方法，就是`Server-Sent Event` （简称 SSE）

### SSE 本质
严格地说，HTTP 协议无法做到服务器主动推送信息。但是有一种变通的方法，就是服务器向客户端声明，接下来要发送的是流信息（Streaming）

也就是说，发送过来的不是一个一次性的数据包，而是一个数据流，会连续不断地发送过来。这时客户端不会关闭连接，会一直等着服务器发过来的数据流，视频播放就是这样的例子，本质上，这种通信就是以流信息的方式，完成一次用时很长的下载。

SSE 就是利用这种机制，使用流信息向浏览器推送信息。它基于 http 协议，目前除了 IE/老 Edge，其它的浏览器都支持。

### SSE 的特点
SSE 与 Websocket 一样都是建立在服务器与客户端之间的连接，但总体来说，Websocket 更灵活，因为它是全双工通道，可以双向通信。而 SSE 是单向通道，只能服务器发送数据给客户端。

![](http://cdn.liwuhou.cn/tmp/20220315215537.png)

但是 SSE 也有自己的优点

> - SSE 使用 HTTP 协议，现有的服务器软件都支持；Websocket 是一个独立的协议
> - SSE 属于轻量级，使用简单；Websocket 协议相对复杂
> - SSE 默认支持断线重连；Websocket 需要自己实现
> - SSE 一般只用来传送文本，二进制数据需要编码后传送；Websocket 默认支持二进制数据
> - SSE 支持自定义发送的消息类型

### 客户端 API
**EventSource**对象

SSE 客户端的 API 在 `EventSource`对象上。下面的代码可以检测当前浏览器是否支持 SSE

```js
if ('EventSource' in window) {
  // Do something
}
```

使用 SSE 时，浏览器首先会生成一个 `EventSource` 实例，向服务器发起连接

```js
const source = new EventSource('url')
```

上面的`url`可以与当前网址同域，也可以跨域。跨域时，可以指定第二个参数，打开`withCredentials`属性，表示是否一起发送 Cookie。

```javascript
const source = new EventSource(url, { withCredentials: true });
```

`EventSource` 实例的 `readyState` 属性，表明连接的当前状态。该属性只读，可以取以下值

> -   0：相当于常量`EventSource.CONNECTING`，表示连接还未建立，或者断线正在重连。
> -   1：相当于常量`EventSource.OPEN`，表示连接已经建立，可以接受数据。
> -   2：相当于常量`EventSource.CLOSED`，表示连接已断，且不会重连。

### 基础用法
连接一旦建立，就会触发 `open` 事件，可以在 `onopen` 属性定义回调函数

```js
source.onopen = function(event) {}

// 也可以这样监听
source.addEventListener('open', function(event) {}, false)
```

客户端在收到服务器发来的数据时，会触发 `message` 事件，可以在 `onmessage` 属性放置回调函数

```js
source.onmessage = function(event) {
  const { data } = event
  // hadle data
}

// or

source.addEventListener('message', function(event) {
  const { data } = event
}, false)
```
上面的数据就是服务器传回的数据（文本格式）
如果通信发生错误（比如连接中断），这会触发 `error` 事件，可以在 onerror 属性定义回调

```js
source.onerror = function(event) {}

// or

source.addEventListener('error', function(event) {})
```

`close` 方法可以用来关闭 SSE 连接

```js
source.close();
```

### 自定义事件
默认情况下，服务器发来的数据，总是触发浏览器的 `EventSource` 实例的 `message` 事件。开发者还可以自定义 SSE 事件，这样发送回来的数据就不会触发 `message` 事件了

```js
source.addEventListener('foo', function(event) {
  const { data } = event
  
}, false)
```

上面是浏览器 SSE 对 `foo` 事件的监听，服务器发送对应 `event` 字段是 `foo` 事件的包即可

