[Loader](https://webpack.js.org/concepts/#loaders)

>  Out of the box, Webpack only understands JavaScript and JSON files. **Loaders** allow Webpack to process other types of files and convert them into valid [modules](https://webpack.js.org/concepts/modules) that can be consumed by your application and added to the dependency graph.

由于 Webpack 只能理解 Javascript 和 JSON 文件，这是 Webpack 开箱即有的功能。如果要让 Webpack 去处理其它类型的文件，那就需要对应的 loader，现在我们来往 mini-webpack 里实现一个loader

### loader 的实现原理
在 Webpack 中使用 loader，需要先在 `webpack.config.js` 中配置，配置的格式如下：

```javascript
// webpack.config.js

module.exports = {
  modules: {
    rules: [
      {
        test: /\.json$/,
        use: 'some-json-loader'
      }
    ]
  }
}
```

然后在 `createAsset` 方法之中，在读取完文件之后，经 babel 生成抽象语法树之前，遍历一遍配置对象中的 `modules.rules` 数组，匹配到正则 `test` 字段的则使用相应 `use` 字段给的 loaders(use 可以是一个数组 )，这样对文件的内容做一层处理，后续接着走打包的逻辑，实现了 `all in js` 的思想

接着再改造下 `createAsset` 的方法

```javascript
function createAsset(filePath) {
  let source = fs.readFileSync(filePath, {
    encording: 'utf-8'
  })

  // 抽象语法树前，执行 loader
  if (Array.isArray(config.modules.rules)) {
    config.modules.rules.forEach(({ test, use }) => {
      if (test.test(filePath) && use) { // test 匹配上了就过一遍 use 数组里的 loader
        (Array.isArray(use) ? [...use].reverse() : [use]) // 按 wepback 中的实现，use 数组的 loader 顺序是倒序
          .reduce((handledSource, currLoader) => currLoader(handledSource), source)
      }
    })
  }
}
```

这里先写一下咱这 Webpack 的 loader 配置

```javascript
// mini-wepback.config.js
import jsonLoader from './json_loader.js'

export default {
  modules: {
    rules: [
      {
        test: /\.json$/,
        use: jsonLoader
      }
    ]
  }
}
```

再实现下读取 json 的 loader

```javascript
// json_loader.js
export default (source) => {
  return `export default ${JSON.stringify(source)}`
}
```

最后在 js 文件中引入一个 json 测试一下，完美。