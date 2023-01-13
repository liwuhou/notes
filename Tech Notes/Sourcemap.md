sourcemap 是一个用来映射源码与转换后代码的位置的文件。简单来说，它其实就是一个位置文件，里面储存着位置信息

sourcemap 不仅可以用来调试压缩后的代码，也可以用来调试编译后的代码。如果没有 sourcemap，那么开发人员调试的代码都转换后的代码，而不是可读性更好的原始代码。

![](http://cdn.liwuhou.cn/tmp/20221114231647.png)

js 中的`eval` 可以动态地执行代码，但是 `eval` 中的代码却打不了断点。
为了解决这个问题，浏览器中引入了一种特性，只要在 `eval` 代码中加上 `//#sourceURL=xxx` 就可以以 `xxx` 为名字的代码加到 sources 里，就可以正常打断点了。

```js
eval(`
  function add(a, b) {
    return a + b;
  }     
//# sourceURL=xxx.js`)
```

webpack 中生成 sourcemap 的配置比较丰富，工具也会根据 `^(inline-|hidden-|eval-)?(nosources-)?(cheap-(modules-)?)?source-map$` 的正则规律来组合。

- `eval`： 浏览器 `devtool`支持通过 sourceURL 来把 eval 的内容单独生成文件，可以进一步通过 sourceMappingUrl 来映射回源码，webpack 利用这个特性来简化 sourcemap 的处理，可以直接从模块开始映射，不用从 bundle 级别
- `cheap`：只映射到源代码的某一行，不精确到列，可以提升 sourcemap 的生成速度
- `source-map`：生成 sourcemap 文件，配置 inline，会以 dataURL 的方式内联，可以配置 hidden，只生成 sourcemap，不和生成的文件关联
- `nosources`：不生成 sourceContent 内容，可以减小 sourcemap 文件的大小
- `module`：sourcemap 生成时会关联第一步 loader 生成的 sourcemap，可以映射回最初的源码

如果需要更细致的对 webpack 中的 sourcemap 进行配置，可以将 devtool 选项设置为 `false`，然后引入 [SourceMapDevToolPlugin](https://webpack.js.org/plugins/source-map-dev-tool-plugin/) 插件来配置。