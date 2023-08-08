---
title: 十分钟——带你了解webpack的主要配置
date: 2020-01-07
tags: 
    - Javascript
    - NodeJs
    - TenMinutes
summary: reduce是个功能很强大，也很有意思的数组方法。以前没深入学习的时候，只是让他来做做累加，处理一些简单数据的合计功能。当我深入挖掘的时候才发现他可以做的东西还有很多，比如拼接字符串、拼接数组、过滤和映射数组、整合对象等骚操作。
---

### 简单说几句

webpack本质上是一个打包工具，它会根据代码的内容解析模块依赖，帮助我们把多个模块的代码打包。借用webpack官网的图片。

![webpack](http://cdn.liwuhou.cn/blog/20200306224904.png)


<!-- more -->

### 入口

多个代码模块中会有一个起始的`.js`文件，这个便是webpack构建的入口。webpack会读取这个文件，并从他开始解析依赖，然后进行打包。你热播的入口文件就是`./src/index.js`。

常见的项目中，多是单页面应用，那么可能入口只有一个;如果是多页面应用，那么就是多个页面对应多个构建入口了。
入口可以使用`entry`来进行配置，webpack支持配置多个入口来进行构建：
```JavaScript
// 单个入口
module.exports = {
    entry: './src/index.js'
}

// 上述配置等同于
module.exports = {
    entry: {
        main: './stc/index.js'
    }
}

// 或者配置多个入口
module.exports = {
    entry: {
        foo: './src/page-foo.js',
        bar: './src/page-bar.js',
        // ...
    }
}

// 使用数组来对多个文件进行打包也可
// 也可以理解为多个文件作为一个入口，webpack会解析两个文件的依赖后进行打包
module.exports = {
    entry: {
        main: [
            './src/foo.js',
            './src/bar.js'
        ]
    }
}
```

### loader
webpack中提供了一种处理多种文件格式的机制，便是使用loader。我们可以把loader理解为一个转换器，负责把某种文件格式的内容转换为 webpack 可以支持打包的模块。

举个例子，在没有添加额外插件的情况下，webpack会默认把所有依赖打包为js文件，入口文件依赖一个.hbs的模板文件以及一个.css的样式文件，那么我们需要`handlebars-loader`来处理.hbs文件，需要`css-loader`来处理.css文件（这里还需要`style-loader`，后续详解），最终把不同格式的文件都解析成js代码，以便打包在浏览器中运行。

当我们需要不同的loader来解析处理不同类型的文件时，我们可以在`modul.rules`字段下来配置相关的规则，例如使用Babel来处理.js文件

```JavaScript
module.exports = {
    module: {
        // ...
        rules: [
            {
                test: /\.jsx?/,  // 匹配文件路径的正则表达式，通常用来匹配文件后缀名
                include: [
                    path.resolve(__dirname, 'src') // 指定那些路径下的文件需要禁用loader的处理
                ],
                use: 'babel-loader' // 指定使用的loader
            }
        ]
    }
}
```

loader是webpack中比较复杂的一块内容，它支撑着webpack来处理文件的多样性，后续我们还会介绍如何更好地使用loader以及如何开发loader。

### plugin
在webpack的构建流程中，plugin用于处理更多其他的一些构建任务。可以这么理解，模块代码转换的工作由loader来处理，除此之外的其他任何工作都可以交由plugin来完成。
通过添加我们需要的plugin，可以满足更多构建中的特殊需求。例如，要使用压缩js代码的`uglifyjs-webpack-plugin`插件，只需在配置中通过`plugins`字段添加新的plugin即可：

```js
import UglifyPlugin = require('uglifyjs-webpack-plugin');
module.exports = {
    // ...
    plugins: [
        new UglifyPlugin()
    ]
}
```

 除了压缩js代码的`uglifyjs-webpack-plugin`，常用的还有定义环境变量的`DefinePlugin`，生成css文件的`ExtractTextWebpackPlugin`等。

 plugin理论上可以干涉webpack整个构建流程，可以在流程的每一个步骤中定制自己的构建需求。

 ### 输出
 webpack的输出即指webpack最终构建出来的静态文件，可以看看上面的webpack官方图片右侧的那些文件。当然，构建结果的文件名、路径等都是可以配置的，使用`output`字段。

 ```JavaScript
const path = require('path');
module.exports = {
    // ...
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
}

// 或者多个入口生成不同的文件
module.exports = {
    entry: {
        foo: './src/foo.js',
        bar: './src/bar.js'
    },
    output: {
        filename: '[name].js',
        path: __dirname + '/dist/'
    }
}


// 在路径中使用hash，每次构建时会有一个不同hash值，避免发布新版本时线上使用浏览器缓存
module.exports = {
    // ...
    output: {
        filename: '[name].js',
        path: __dirname + '/dist/[hash]'
    }
}
 ```

 我们一开始直接使用webpack构建时，默认创建的输出内容就是`./dist/main.js`。

 ### 开发环境和生产环境配置的差异
 一般在前端开发中，都会有两套的构建环境：一套开发时使用，构建结果用于本地调试，不进行代码压缩，打印debug信息，包含sourcemap文件;另一台构建后的结果应用于生产环境，代码需要经过压缩，运行时不打印debug信息，静态文件不包括sourcemap的。有的时候还需要一套测试环境，在运行时直接请求mock等工作。

 ### 在配置文件中区分mode
 webpack的配置文件都是直接暴露一个js对象，这种方式暂时没有办法获取到webpack的mode参数，所以需要一些曲线救国的方式。特别是在webpack 3.x版本中，只能依靠运行环境的node提供的机制来给webpack传递环境变量，控制不同环境下的构建行为，例如：我们在npm中的`scripts`字段添加一个用于生产环境的构建命令：

 ``` json
"scripts": {
    "build": "NODE_ENV=production webpack",
    "develop": "NODE_ENV=development webpack-dev-server"
}
 ```

 然后在`webpack.config.js`中通过`process.env.NODE_ENV`来获取命令传入的环境变量

 ```javascript
const config = {
    // webpack配置
}

if(process.env.NODE_ENV === 'production'){
    // 生产环境需要做的事，比如使用代码压缩插件等
    config.plugins.push(new UglifyJsPlugin())
}

module.exports = config;
 ```

在webpack4.x的做法略有不同，不需要这么曲线救国的方式获取环境变量了，根据官方文档[多种配置类型](https://webpack.docschina.org/configuration/configuration-types/)，配置文件可以对外暴露一个函数，因此我们可以这么做：

 ```JavaScript
module.exports = (env, argv) => ({
    // ... 其他配置
    optimization: {
        minimize: false,
        // 使用argv.mode来获取环境参数
        minimizer: argv.mode === 'production' ? [
            new UglifyJsPlugin({}),
            // 其他需要在产生环境配置的plugin
        ] : [
            // 开发环境
        ]
    }
})
 ```

这样获取到mode之后，就可以根据不同的环境来配置不同的loader和plugin了。

4.x也可以通过设置[mode](https://webpack.docschina.org/concepts/mode/)，或者从CLI参数中传递 `webpack --mode=production`

|     选项      | 描述                                                                                                                                                                                                                                                       |
| :-----------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `development` | 会将`DefinePlugin`中的`process.env.NODE_ENV`的值设置为`development`。启用`NamedChunksPlugin`和`NameModulesPlugin`。                                                                                                                                        |
| `production`  | 会将`DefinePlugin`中的`process.env.NODE_ENV`的值设置为`production`。启用`FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` 和 `TerserPlugin`。 |
|    `none`     | 退出任何默认优化选项                                                                                                                                                                                                                                       |



> 如果没有设置，webpack会将`mode`默认值设置为`production`
> 记住，设置`NODE_ENV`并不会自动地设置`mode`。


### 运行时的环境变量
在webpack4.x中，可以直接在项目运行时访问`process.env.NODE_ENV`来访问环境变量，而在3.x中就需要我们使用`DefinePlugin`插件手动的声明环境变量了。

```JavaScript
const webpack = require('webpack');
module.exports = {
    // webpack配置
    plugins: [
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)
        })
    ]
}
```

 ### 常见的环境差异配置

 - 生产环境可能需要分离css成单独的文件，以便多个页面共享同一个css文件
 - 生产环境需要压缩html/css/js代码
 - 生产环境需要压缩图片
 - 开发环境需要生成sourcemap文件
 - 开发环境需要打印debug信息
 - 开发环境需要live reload或者hot reload功能

 以上便是常见的环境需求差异，可能更加复杂的项目中会有更多的构建需求（如划分静态域名等），但是我们都可以通过判断环境变量来实现这些有环境差异的构建需求。

 webpack4.x已经提供了上述差异配置的大部分功能，mode为production时默认使用js代码压缩，而mode为development时默认启用hot reload功能等等。这样让我们配置更为简洁，只需要针对特别使用的loader和pulugin就可以了。

 webpack3.x版本则还是只能自己手动修改配置来满足大部分差异需求。

 ### 拆分配置
 前面列出的几个环境差异的配置，可能这些构件需求就已经有点多了，会让整个webpack的配置变得复杂，尤其是有这大量环境变量判断的配置。我们可以把webpack的配置按照不同的环境拆分为多个文件，运行时直接根据环境变量加载对应的配置即可。基本的划分如下：

 - `webpack.base.js`: 基础部分，即多个文件中共享的配置
 - `webpack.development.js`: 开发环境使用的配置
 - `webpack.prodution.js`: 生成环境使用的配置
 - `webpack.test.js`: 测试环境使用的配置

首先我们知道，对于webpack的配置，其实本质上就是一个js对象，所以对于这个对象，我们可以使用js代码来修改：

```JavaScript
const config = {
    // ... webpack配置

}
// 我们可以修改这个config来调整配置，例如添加一个新的插件
config.plugins.push(new YourPlugin());

module.exports = config;
```

我们可以使用`webpack-merge`工具，它会通过判断环境变量，然后比较智能地合并多个配置对象，这样就可以很轻松的拆分webapck配置了。

```bash

# 首先需要安装 webpack-merge
npm install -D webpack-merge
# or
yarn add -D webpack-merge

```

然后配置webapck.base.js，大概是这样：

```JavaScript
// webpack.base.js
module.exports = {
    entry: '...',
    output: {
        // ...
    },
    resolve: {
        // ...
    },
    module: {
        // 这里是一个简单的例子
        rules: [
            {
                test: /\.js$/,
                use: ['babel']
            }
        ]
    },
    plugins: [
        // ...
    ]
}
```

然后webpack.development.js需要添加开发环境的loader和plugin，就可以使用webpack-merge的api了：

```JavaScript
// webpack.development.js
const {smart} = require('webpack-merge');
const webpack = require('webpack');
const base = require('./webpack.base.js');

// 用smart api，rules中的匹配规则相同，use又都是数组的时候，smart会识别后处理
module.exports = smart(base, {
    module: {
        rules: [
            {
                test: /\.js$/,
                use: ['coffee']
            }
            // 和上述base配置合并后，这里是{test: /\.js$/, use: ['babel', 'coffee']}
            // 如果use的值用的是字符串或者对象的话，那么就会替换掉原本的规则use的值
        ]
    },
    plugins: [
        // plugins这里的数组会和base中的plugins数组合并
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)
        })
    ]
})
```

可见webpack-merge插件提供的smart方法，可以帮助我们更加轻松的处理loader配置的合并。
webpack-merge还有其他api可以用于自定义合并行为，这里就不详细介绍了，可以查阅[官方文档](https://github.com/survivejs/webpack-merge) 

> 每次花个十分钟，懂一个前端知识点，走得虽慢，但坚持走下去，足以致千里。

关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)
