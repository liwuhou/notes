`webpack` 是一个现代 `JavaScript` 应用程序的静态模块打包器，当 `webpack` 处理应用程序时，会递归构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将这些模块打包成一个或多个 `bundle`。

### 概念

- entry： 入口
- output： 输出
- loader： 模块转换器，用于把模块原内容按照需求转换为新内容
- plugins： 扩展插件，在 webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或产生一些副作用

`webpack4` 开始，支持开箱即用，在不引入任何配置文件的情况下就可以使用，例如默认的入口是 `./src`，而默认的打包出口是 `./dist/main.js`。更多的配置可以见 `node_modules/webpack/lib/WebpackOptionDefaulter.js` 这个文件

### Entry
`entry` 用来指示 webpack 打包模块的入口，

### loader
loader 是 webpack 中，用于将源代码进行转换的工具。

`loader` 需要配置在 `webpack.config.js` 中的 `module.rules` 数组中
并且 `loader` 的格式为：

```js
{
  test: /\.jsx?$/, // 匹配规则
  use: 'babel-loader'
}
// or
{
  test: /\.jsx?$/,
  loader: 'babel-loader',
  options: {
    // ...
  }
}
// or
{
  test: /\.jsx?$/,
  use: {
    loader: 'babel-loader',
    options: {
      // ...
    }
  }
}
```

`test` 字段是匹配规则，针对符合规则的文件进行处理

`use`字段有几种写法，如上

将 js 代码向低版本转换：

首先需要用到[babel-loader](https://npmjs.org/package/babel-loader)，除此之外，还需要对 `babel` 进行配置，为此我们安装一下以下依赖：

```bash
npm install @babel/core @babel/preset-env @babel/plugin-transform-runtime -D
npm install @babel/runtime @babel/runtime-corejs3
```

然后通过新建一个 webpack 的配置

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /.jsx?$/,
        use: ['babel-loader'],
        exclude: /node_modules/ 
      }
    ]
  }
}
```

通过对 `loader` 指定 `include` 或是 `exclude`，指定其中一个即可，因为 `node_modules` 目录通常不需要去编译，这样排除后，有利于编译效率的提升

接着可以在 `.babelrc` 中编写 `babel` 配置，也可以在 `webpack.config.js` 中进行配置。

1. 创建一个 `.babelrc`

```json
{
  "presets": ["@babel/preset-env"],
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": 3
      }
    ]
  ]
}
```

2. 在 webpack 中配置 babel

```js
//webpack.config.js 
module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
            plugins: [
              [
                '@babel/plugin-transform-runtime',
                {
                  corejs: 3
                }
              ]
            ]
          }
        },
        exclude: /node_modules/
      }
    ]
  }
}
```

在没有配置 `mode`字段的时候，可以通过命令行的方式指定 webpack 的 mode，

```bash
npx webpack --mode=development
```

也可以在 `webpack.config.js` 配置，两种都支持以下配置

- `development`: 将 `process.env.NODE_ENV` 的值设置为 `development`，启用 `NamedChunksPlugin` 和 `NamedModulesPlugin`
- `production`: 将 `process.env.NODE_ENV` 的值设置为 `production`，启用 `FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` 和 `UglifyJsPlugin`

然后直接运行 `npx webpack` 即可。

### plugin
