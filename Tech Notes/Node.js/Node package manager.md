Node.js 使用模块化来管理应用，可以使得每个模块彼此独立，代码不会相互影响。在早先，Node.js 诞生之初，Javascript 还没有标准的模块化机制，所以 Node.js 一直采用 [[CommonJS]]规范。而在随后，Javascript 标准的模块制 [[ES Modules]] 诞生了，浏览器厂商也纷纷开始支持 `ES Modules`。Node.js 从 `v13.2.0` 开始引入 `ES Modules`规范，同时也兼容早期的 `CommonJS`。

而在 Node.js 支持 `ES Modules`之前，就有 `Babel` 这样的编译工具和 `webpack` 这类的打包机，可以将 `ES Modules`模块规范的代码转为 `CommonJS`模块规范的代码了。到了现在，Node.js 自身对 `ES Modules` 的支持也越发成熟。

> [[实现打包功能|实现 webpack 的打包]]

  ![](http://cdn.liwuhou.cn/tmp/20230120231400.png)

### 两种规范的异同

| -        | CJS                                              | ESM                                              |
| -------- | ------------------------------------------------ | ------------------------------------------------ |
| 语法类型 | 动态                                             | 静态                                             |
| 关键声明 | `require`                                        | `export` & `import`                              |
| 加载方式 | 运行时加载                                       | 编译时加载                                       |
| 加载行为 | 同步加载                                         | 异步加载                                         |
| 书写位置 | 任意                                             | 顶层位置                                         |
| 指针指向 | `this` 指向 `当前模块`                           | `this` 指向 `undefined`                          |
| 执行顺序 | 首次引用时 **加载模块**，再次引用时 **读取缓存** | 引用时生成 `只读引用`，执行时才是正式取值        |
| 属性引用 | 基本类型属于复制不共享，引用类型属于浅拷贝且共享 | 所有类型属于动态只读引用                         |
| 属性改动 | 工作空间可修改引用的值                           | 工作空间不可修改引用的值，但可通过引用的方法修改 |

### 在 Node.js 中使用模块化的三种方式
1. 仍然使用旧的 `CommonJS` 规范，预计未来很长的一段时间，Node.js 依然会同时兼容 `ES Modules` 和 `CommonJS`两种规范。
2. 项目中采用 `ES Modules`，通过 `babel` 编译为 `CommonJS` 规范的代码。
3. 直接用最新的 `ES Modules`：这种方式在 `v13.2.0` 以后的版本中可用。但是使用会有条件。

### 在 Node.js 中使用 ES Modules 规范
在 Node.js的开发中，一般使用 `CommonJS` 的规范会比较多，但在 Web 的开发中，我们都习惯使用 `ESM` 规范编码。

而在真实的开发中，其实完全可以在 Node.js 开发中使用 `ESM`开发，因为 Web 和 Node.js 其实本质上都是一种 JS 的运行环境，所以只有本质上运行环境不同而已。所以我们需要一个工具来抹平这两种环境的差异，让 Node 开发变得像 Web 开发一样丝滑。

#### 让 Node 支持 ESM
在 `V13.2.0`之后，通过在 `package.json` 文件中，更改字段 `type` 为 `module`|`commonjs` 就可以让项目默认使用 `ESM` 还是 `CommonJS`规范来解析 js 模块。同时，如果想在 `ESM` 规范或者 `CJS` 规范中收入不同规范的代码，就需要更改文件的后缀为 `.cjs` 或者 `.mjs`。

#### 部署 Node.js 的 ESM 开发环境
上述的方案虽好，但也有因为 Node.js 高低版本差异带来的兼容性问题，这个时候就可以通过引入 `babel` 的方式，让开发代码中的 `ESM` 代码，编译为 `CJS` 的代码。另外，很多 Npm 模块都使用的 CJS 编译，项目中同时使用 `require` 和 `export/import` 会报错，这个时候，将代码从 `ESM` 转换为 `CJS` 就是最稳定的做法了。这个 Node 编译部署的方案，适用于 Node 的任何版本，当属 Node 支持 ESM 最稳定的方案，且无之一。

使用 `babel` 编译，就要用到相应的工具链。

```bash
npm i @babel/cli @babel/core @babel/node @babel/preset-env -D
```

这四个 `babel` 子包，对于 部署 Node 编译至关重要。

- `@babel/cli`：提供支持 `@babel/core` 的命令运行环境
- `@babel/core`： 提供转译函数
- `@babel/node`： 提供支持 `ESM` 的命令运行环境
- `@babel/preset-env`： 提供预设语法转换集成环境

安装完毕后，在 `package.json` 中指定 `babel` 的相关配置（或者单独配置 `.babelrc.json` 或 `babel.config.json`）。

详细配置见[babel 官网文档](https://babeljs.io/docs/en/configuration)

