## 概述

在浏览器的 js 中，采用的是单线程模型，也就是说，所有任务都只能在一个线程上完成。更要命的时，主线程中页面的渲染和 js 的执行是互斥的，这就导致在处理一些耗时较长的 js 执行的时候会导致页面响应的卡顿。而且随着电脑计算能力的增强，多核 cpu 的出现，单线程无法完全发挥计算机的计算能力。

Web Worker 的作用就在于为 js 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行，两者之间互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程，这样一些计算密集或高延迟的任务就会被 Worker 线程负担，而主线程（通常是负责 UI 交互的部分）就会很流畅，不会阻塞或拖慢。

Web Worker 虽好，但也有几点需要注意：

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
work.onmessage = function (event) {
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
self.addEventListener(
  'message',
  (e) => {
    self.postMessage('You send: ', e.data)
  },
  false
)
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
self.addEventListener('message', ({ data }) => {
  switch (data.cmd) {
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
worker.addEventListener('error', () => {
  /* ... */
})
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

主线程与 Worker 线程通信的内容可以是文本、对象，甚至是二进制数据。不过需要注意的是，这种通信是拷贝关系，即传值而不是传址，Worker 对通信内容的修改不会影响到主线程。事实上，浏览器内部的运行机制就是，先将通信内容串行化，然后将串行化后的字符串发给 Worker，后者再将它还原。

由于通信之间可以发生二进制数据，但是拷贝方式改善二进制数据会造成性能问题。比如发送一个大文件的时候，浏览器都会生成一个原文件的副本，如果文件过大的话，浏览器就会花太多时间在生成副本上导致卡顿。为了解决这个问题，js 允许主线程把二进制数据直接转移给子线程，只是一旦转移，主线程就无法再使用这些二进制数据了，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移的方式称为 [Transferable Objects](https://www.w3.org/html/wg/drafts/html/master/infrastructure.html#transferable-objects) 。这使得主线程可以快速地把数据交给 Worker，对于影像处理、声音处理、3D 运算等就很方便了，不用担心会产生性能负担

如果要转移数据的控制权， 就要使用下面的写法

```js
// Transferable Object 格式
worker.postMessage(arrayBuffer, [arrayBuffer])

// example
var ab = new ArrayBuffer(1)
worker.postMessage(ab, [ab])
```

### 同页面的 Web Worker

通常情况下，Worker 载入的是一个单独的 JavaScript 脚本文件，但是也可以载入与主线程在同一个网页的代码。

```html

<!DOCTYPE html>
  <body>
    <script id="worker" type="app/worker">
      addEventListener('message', function () {
        postMessage('some message');
      }, false);
    </script>
  </body>
 </html>
```

上面是一段嵌入网页的脚本，注意必须指定`<script>`标签的`type`属性是一个浏览器不认识的值，上例是`app/worker`。

然后，读取这一段嵌入页面的脚本，用 Worker 来处理。

```js
var blob = new Blob([document.querySelector('#worker').textContent])
var url = window.URL.createObjectURL(blob)
var worker = new Worker(url)

worker.onmessage = function (e) {
  // e.data === 'some message'
}
```

上面代码中，先将嵌入网页的脚本代码，转成一个二进制对象，然后为这个二进制对象生成 URL，再让 Worker 加载这个 URL。这样就做到了，主线程和 Worker 的代码都在同一个网页上面。

%% ASY: 如果我不需要上面的 html 代码，直接就从 js 中生成，能有用吗？ %%

## 实践

### Worker 线程完成轮询

一些轮询的接口也可以放到 worker 里面

```js
function createWork(f) {
  const blob = new Blob(['(' + f.toString() + ')()'])
  const url = window.URL.createObjectURL(blob)
  const worker = new Worker(url)
  return worker
}

const pollingWorker = createWorker(function(e) {
  let cache
  const compare = (new, old) => new === old

  setInterval(() => {
    fetch('/deploy_time').then((res) => {
      const data = res.json()

      if (!compare(data, cache)) {
        cache = data
        self.postMessage(data)
      }
    })
  }, 1000)
})

pollingWorker.onmessage = function() {
  // receive data
}

pollingWorker.postMessage('init')
```

上面代码中，Worker 每秒钟轮询一次数据，然后跟缓存做比较。如果不一致，就说明服务端有了新的变化，因此就要通知主线程。

# 七、API

### 7.1 主线程

浏览器原生提供`Worker()`构造函数，用来供主线程生成 Worker 线程。

```javascript
var myWorker = new Worker(jsUrl, options)
```

`Worker()`构造函数，可以接受两个参数。第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，且只能加载 JS 脚本，否则会报错。第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称，用来区分多个 Worker 线程。

```javascript
// 主线程
var myWorker = new Worker('worker.js', { name: 'myWorker' })

// Worker 线程
self.name // myWorker
```

`Worker()`构造函数返回一个 Worker 线程对象，用来供主线程操作 Worker。Worker 线程对象的属性和方法如下。

- Worker.onerror：指定 error 事件的监听函数。
- Worker.onmessage：指定 message 事件的监听函数，发送过来的数据在`Event.data`属性中。
- Worker.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- Worker.postMessage()：向 Worker 线程发送消息。
- Worker.terminate()：立即终止 Worker 线程。

### 7.2 Worker 线程

Web Worker 有自己的全局对象，不是主线程的`window`，而是一个专门为 Worker 定制的全局对象。因此定义在`window`上面的对象和方法不是全部都可以使用。

Worker 线程有一些自己的全局属性和方法。

- self.name： Worker 的名字。该属性只读，由构造函数指定。
- self.onmessage：指定`message`事件的监听函数。
- self.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- self.close()：关闭 Worker 线程。
- self.postMessage()：向产生这个 Worker 线程发送消息。
- self.importScripts()：加载 JS 脚本。
