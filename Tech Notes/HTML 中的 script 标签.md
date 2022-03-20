时至今日，将 js 代码引入 html 中的主要方法还是利用 `<script>` 元素，甚至是动态加载脚本也是利用 DOM api 挂载一个`<script>`标签的形式。这个元素最早是由网景公司创造出来的，最早在  **Netscape Navigator 2** 中实现的。后来，这个元素被正式加入到HTML规范。
`<script>` 元素有下列8个可选属性

- `charset`: 指定 `src` 属性使用的代码字符集，这个属性很少用，因为大多数浏览器都不在乎它的值
- `async`： 表示马上开始下载脚本，但不能阻塞其它页面动作，比如下载其它资源和等待其它脚本加载。只对外部文件有效。
- `defer`： 表示脚本可以延迟到文档完全被解析之后再执行，只对外部脚本有效。在 IE7 及更早的版本中，对行内脚本也有效
- `crossorigin`： 配置相关请求的 CROS（跨资源共享），默认不使用 CORS。`crossorigin="anonymous"`，配置文件请求不必设置凭据标志，`crossorigin="use-credentials"` 设置凭据标志，意味着出站请求会包含凭据。
- `integrity`允许比对接收到的资源和指定的加密签名以验证子资源完整性（SRI，Subresource Integrity）。如果接收到的资源的签名与这个属性指定的签名不匹配，则页面会报错，脚本不会执行。这个属性可以用于确保内容分发网络（CDN，Content Delivery Network）不会提供恶意内容。
- `language`： 废弃。最初用于表示代码块中的脚本语言（如"JavaScript"、"JavaScript 1.2"或"VBScript"）。大多数浏览器都会忽略这个属性，不应该再使用它。
- `src`： 表示包含要执行的代码的外部文件。
- `type`：可选。代替 `language`，表示代码块中脚本语言的内容类型（也称MIME类型） 。按照惯例，这个值始终都是`"text/javascript"`，尽管`"text/javascript"`和`"text/ecmascript"`都已经废弃了。JavaScript文件的MIME类型通常是`"application/x-javascript"`，不过给type属性这个值有可能导致脚本被忽略。在非IE的浏览器中有效的其他值还有`"application/javascript"`和`"application/ecmascript"`。**如果这个值是`module`，则代码会被当成ES6模块，而且只有这时候代码中才能出现import和export关键字。**

使用 `<script>` 有两种方式，一种是直接在 `<script>` 标签内写入代码，另一种方式则是通过 `src` 引入外部的脚本文件。

要嵌入 Javascript 代码，直接放入 `<script>` 内即可

```html
<script>
  function sayHi() {
    alert('hi');
  }
</script>
```

  > 在使用行内代码时，要注意代码中不能出现字符串 `'</script>'` ，因为浏览器解析脚本的时候会在看到 `</script>` 时，把它当成 `script`元素的结束标签，想要解决这个问题，要用 `\` 对 `/` 转义。

```html
<script>
  function sayHi() {
    alert('<\/script>')
  }
</script>
```

另一种方式即是通过 `src` 来引入外部的脚本文件，这个也是绝大多数的使用场景

```html
<script src="./example.js"></script>
```

这里页面引入了一个 `example.js` 的外部脚本，与解析行内脚本一样，在解析外部 js 文件的时候，页面也是会阻塞（阻塞的时间也包含了下载这个外部 js 文件的时间）。

> 值得注意的是，在 HTML 中，不能忽略 `script` 的结束标签，即 `<script src="./exaple.js" />` 因为它是无效的 HTML，这在某些浏览器中将不能正常处理。

在 `script` 引入外部 js 的时候，浏览器也不会真的去检查引入的文件扩展名是不是 `.js`。另外使用了 `src` 属性的 `<script>` 的元素也不会再去执行元素内的代码。

### 标签位置
一般来说，`<script>` 标签有两个位置可以插入，一个是在`<Head>` 元素内，一个是在`<body>`元素的结束标签之前。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Example HTML Page</titie>
    <script src="./example.js"></script>
    <body>
      <!-- 页面内容 -->
      <script src="./example.js"></script>
    </body>
  </head>
</html>
```

第一种做法的主要目的是为了让外部的 CSS 和 Javascript 文件都集中放在一起。不过这样的做法也就意味着浏览器必须将所有的 js 代码都下载、解析和解释完成后，才会开始渲染页面（页面在浏览器解析到 `<body>` 的起始标签时开始渲染）。这就将会导致页面渲染的明显延迟，因为在此期间浏览器窗口完全空白。
为了解决这一种做法中页面空白时间可能过长的问题，将 js 代码引用放置在 `<body>` 元素中页面内容的后面，让浏览器先行解析页面元素，这样就有效降低了页面的空白时间。

### 推迟执行脚本
在 HTML4.01中，`<script>` 元素定义了一个叫 `defer`的属性，这个属性能让 js 脚本在整个页面被解析完毕后再运行，因此能让浏览器立即下载外部 js 文件，但延迟执行。HTML5 规范要求两个脚本应该按照它们出现的顺序执行，因此第一个推迟了的脚本会在第二个推迟的脚本之前执行，并且两个脚本都是会在 `DOMContentLoaded`事件之前执行，不过在实际中，推迟脚本不一定总会按顺序执行或者在 `DOMContentLoaded` 事件之前执行，因此一个页面最好只包含一个这样的脚本。

### 异步执行脚本
HTML5 为 `<script>`元素定义了 `async` 属性，告诉浏览器立即开始下载，不必等脚本下载和执行完后再加载页面，整个外部 js 文件的内容与页面的下载和解析并行。同样的，该异步脚本也不会等待其它异步脚本的下载和执行后再加载，标记为 async 的脚本并不能保存执行的次序会按照他们出现的顺序。使用 `async` 也会告诉页面你不会使用 `document.write`，不过好的 Web 开发实践也不推荐使用这种方式。


### 动态加载脚本
除了 `<script>` 标签，还可以通过 js 使用DOM api，向 DOM 中动态地添加 `<script>` 元素加载指定脚本，这也是很多懒加载的实现原理。

```js
const script = document.createElement('script')
script.src = 'example.js'
script.async = false // *
document.head.appendChild(script)
```

在将 `script` 元素添加到 DOM 之前都不会发送请求。默认情况下，以这种方式创建 `<script>` 元素都是以异步方式加载的，相当于添加了 `async` 属性，但是并不是所有浏览器都支持 `async` 属性的，因此如果要统一动态脚本的加载行为，就需要明确将其设置为同步加载： `script.async = false`。

以这种方式获取的资源对浏览器预加载器是不可见的。这会严重影响它们在资源获取队列中的优先级。根据应用程序的工作方式以及怎么使用，这种方式可能会严重影响性能。要想让预加载器知道这些动态请求文件的存在，可以在文档头部显式地声明它们

```html
<head>
  <link ref="preload" href="example.js">
</head>
```
