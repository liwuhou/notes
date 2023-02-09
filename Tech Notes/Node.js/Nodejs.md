Node 是一个基于 Chrome V8 引擎开发的 Javascript 运行时环境，但 Node.js 的运行环境跟浏览器的运行环境还是很不一样的。

![](http://cdn.liwuhou.cn/tmp/20230118105650.png)

首先 Node.js 不是浏览器，因此它不具有 DOM API，诸如 `Window`对象、`Document`对象等。再然后，Nodejs 也提供了自己特有的 API，比如全局的 `global`对象，也提供了当前进程信息的 `Process`对象，操作文件的 `fs`模块，以及创建 web 服务的`http` 模块等等。这些 API 能够让我们使用 Javascript 操作计算机，所以可以用 Node.js 平台来开发 web 服务器。

![](http://cdn.liwuhou.cn/tmp/20230118141826.png)

Node.js 的基本架构
Node.js 由 V8 Javascript 引擎、libUV 库、c-ares、llhttp/http-parser、open-ssl、zlib，以及一些 C/C++ 库等构成，运行于操作系统之上，这就是 Node.js 的基本架构。

其中，libUV 库负责处理事件循环，c-ares、llhttp/http-parser、open-ssl、zlib 等库提供 DNS 解析、HTTP 协议和文件压缩等功能。

而在这些模块的上一层是中间层，包括 `Node.js Bindings`、`Node.js Standard Library` 以及 `C/C++ AddOns`。`Node.js Bindings`层的作用是将底层的那些用 C/C++ 写的库接口暴露给 JS 环境，而 `Node.js Standard Bibrary` 是 Node.js 本身的核心模块。至于 `C/C++ AddOns`，它可以让用户自己的 C/C++ 模块通过桥接方式提供给 Node.js。

中间层之上就是 [[Node's API|Node.js 的 API 层]]了，开发者使用 Node.js 开发应用，主要就是跟这一层打交道，所以 Node.js 的应用最终都会运行在 Node.js 层的 API 层之上。


