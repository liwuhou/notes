Node.js 是运行在操作系统中的 Javascript 运行时环境，封装了一系列操作系统的 API，通过它们我们可以执行操作系统指令、文件 IO 和建立网络请求等操作系统中的服务。

其中 Node 内置的模块比较丰富，但是常用的就这几个：

- File System 模块：也就是常说的 `fs`模块，提供操作系统中操作目录和文件的模块，操作包括读、写、创建、删除和权限设置等。
- Net 模块：提供网络套接字 socket，用来创建 TCP 连接，TCP 连接可以用来访问后台数据库和其它持久化服务。
- HTTP 模块：提供创建 HTTP 连接的能力，常用来创建 Web 服务，是 Node.js 中最常用的核心模块
- URL 模块：用来处理客户端请求的 URL 信息的辅助模块，可以用来解析 URL 字符串。
- Path 模块：用来处理文件路径信息的辅助模块，抹平了系统差异，常用来解析文件路径的字符串。
- Process 模块：用来获取进程信息。
- Buffer 模块：用来处理二进制数据。
- Console 模块：控制台模块，同浏览器的 Console 模块，用来输出信息到控制台。
- Crypto 加密解密模块：用来处理需要用户授权的服务。
- Events 模块：用来监听和派发事件

以上模块只是日常开发中比较常用的模块，欲了解更多，可以访问官网文档： [Node.js latest api doc](https://nodejs.org/dist/latest-v18.x/docs/api/)。

除此之外，Node.js 的社区生态也非常活跃，有大量的第三方模块可以使用，可以通过 [npm 仓库](https://npmjs.com) 检索，然后通过 [[Node package manager|npm]]包管理工具进行安装。