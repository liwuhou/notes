## 概述
在浏览器的 js 中，采用的是单线程模型，也就是说，所有任务都只能在一个线程上完成。更要命的时，主线程中页面的渲染和 js 的执行是互斥的，这就导致在处理一些耗时较长的 js 执行的时候会导致页面响应的卡顿。而且随着电脑计算能力的增强，多核 cpu 的出现，单线程无法完全发挥计算机的计算能力。

Web Worker 的作用就在于为 js 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行，两者之间互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程，这样一些计算密集或高延迟的任务就会被 Worker 线程负担，而主线程（通常是负责 UI 交互的部分）就会很流畅，不会阻塞或拖慢。

Web Workd 虽好，但也有几点需要注意：

1. 同源限制：分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源
2. DOM 限制：Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用 `document`、`window` 和 `parent` 这些对象。但是，Worker 线程可以访问 `navigator` 对象和 `location` 对象。
3. 通信联系：Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成通信。
4. 脚本限制：Worker 线程不能执行 `alert()` 方法和 `confirm()` 方法，但可以使用 `XMLHttpRequest` 对象发出 Ajax 请求。
5. 文件限制：Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络

## 基本用法
主线程采用 `new` 命令，调用 `Worker()` 构造方法，新建一个 Worker 线程

```js
var worker = new Worker('url')
```

`Worker()` 构造函数的参数是一个脚本文件的 url 地址，该文件就是 `Worker` 线程要执行的任务，这个脚本必须来自网络。如果下载没有成功（比如 404 错误），这个 Worker 就会默默地失败。

然后主线程调用 `work.postMessage()` 方法，向 worker 发消息

```js
worker.postMessage('hello world')
worker.postMessage({ method: 'echo', args: ['Work'] })
```

`worker.postMessage()` 方法的参数，就是主线程传给 Worker 的数据。它可以是各种数据类型，甚至是二进制数据。
另外，主线程这边还可以通过 `work.onmessage` 指定监听函数，接收子线程返回的消息

```js
work.onmessage = function(event) {
  doSomething(event.data)
}

function doSomething(data) {
  console.log('received message:', data)
  worker.postMessage('Work done')
}
```

Worker 完成任务以后，主线程可以将其关闭：

```js
worker.terminate()
```

### Worker 线程
另一边，Worker 线程内部有一个监听函数，可以用来监听 `message` 事件。

```js
self.addEventListeren('message', (e) => {
  self.postMessage('You send: ', e.data)
}, false)
```

`self` 代表的就是子线程自身，即子线程的全局对象。因此，等同于下面两种写法

```js
// 写法一
this.addEventListener('message', ...)
// 写法二
addEventListener('message', ...)
```

`self.addEventListener()` 指定监听对象，也可以使用 `self.onmessage = function ...` 来指定回调函数。监听函数的参数是一个事件对象，它的 `data` 属性包含主线程发来的数据。
`self.postMessage()` 方法用来向主线程发送消息。

经过一些封装，可以实现一个根据主线程发来的数据，Worker 调用不同的方法：

```js
self.addEventlistener('message', ({data}) => {
  switch(data.cmd) {
    case 'start':
      self.postMessage('WORKER STARTED:' + data.msg)
      break
    case 'stop':
      self.postMessage('WORKER STOPED:' + data.msg)
      self.close()
    default:
      self.postMessage('Unknown command: ' + data.msg)
  }
})
```

`self.close()` 用于在 Worker 中关闭自身

### Worker 加载脚本
 Worker 内部如果要加载其它脚本的话，有一个专门的方法 `importScripts()`。

```js
importScripts('./script.js')
```

是的，`import scripts`，该方法可以加载多个脚本

```js
importScripts('./script1.js', './script2.js')
```

### 错误处理
主线程可以监听 Worker 是否发生错误。如果发生错误，Worker 会触发主线程的 `error` 事件。

```js
worker.onerror = function (event) {
  console.log(['ERROR: Line', e.lineno, ' in ', e.filename, ': ', e.message].join(''))
}

// or
worker.addEventListener('error', () => {/* ... */})
```

Worker 内部也可以监听 `error` 事件

### 关闭 Worker

使用完毕之后，为了节省系统资源，必须关闭 Worker

```js
// 主线程中
worker.terminate()

// worker 线程中
self.close()
```

## 数据通信
