# Webpack 学习梳理

**概念**

本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

**特性**

- 打包 <a href="https://www.oschina.net/p/commonjs" target="_blank">CommonJs</a> 和 <a href="https://github.com/amdjs/amdjs-api/wiki/AMD" target="_blank">AMD</a> 模块（以及绑定）
- 可创建单个或多个按需加载的块，以减少初始加载时间
- 在编译期间会解决依赖关系，减少了运行时的大小
- 加载器可以在编译时预处理文件，如 jsx 到 js，less 到 css

**官方文档概念讲到这，来说句人话，根据自己的理解说说**
**webpack、gulp 以及新出的 vite 等打包工具可以说是奠定了前端这个领域的基石，没有他们就没有现今的三大框架和预处理语言，也没有体系如此巨大的前端工程化**

#### webpack 四大**核心概念**：

- 入口(entry)
- 输出(output)
- loader
- 插件(plugins)

一个标准 webpack.config.js 配置文件模板

```js
module.export = {
  mode: '', // 打包模式，会启用webpack不同打包策略，分为development开发环境和production 生产环境
  entry: '',
  output: {},
  plugins: [],
  module: {
    rules: [
      // loader处
    ],
  },
}
```

## 1. entry 入口

入口起点(entry point)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

**举个例子就是 spa react 的 index.js 和 vue 的 main.vue 的初始化整个 app 的入口文件**

每个依赖项随即被处理，最后输出到称之为 bundles 的文件中。

可以通过在 webpack 配置中配置 entry 属性，来指定一个入口起点（或多个入口起点）。默认值为 ./src。

#### 单个入口

```js
module.export = {
  entry: './src/index.js',
}
```

#### 多入口

接收一个对象

如区分主入口文件和库文件

```js
module.export = {
  entry: {
    main: './src/index.js',
    vendor: './src/vender.js',
  },
}
```

或者多页面应用
可以理解为 2 个入口文件来构建内部依赖图
此处可以通过配置 CommonChunkPlugin 来复用入口文件的代码

```js
module.export = {
  entry: {
    page1: './src/page1.js',
    page2: './src/page2.js',
  },
}
```

## 2. output 出口

output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist。基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。你可以通过在配置中指定一个 output 字段，来配置这些处理过程

**说人话，出口文件就是进入 html 需要加载的 js 文件**

**单出口**

```js
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'index.bundle.js',
  },
}
```

**多出口**
如果配置创建了多个单独的 "chunk"（例如，使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用占位符(substitutions)来确保每个文件具有唯一的名称

```js
module.export = {
  entry: {
    page1: './src/page1.js',
    page2: './src/page2.js',
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    // 可以保险起见为文件名加上 hash戳，冒号后代表截取hash字符数
    // filename: '[name].bundle.[hash: 6].js'
    filename: '[name].bundle.js',
  },
}
```

## 3. loader 文件预处理器

loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。

本质上，webpack loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。

又是一大堆官腔，说人话
**loader 就是根据文件名匹配规则的预处理器，根据个人理解 loader 所做的说白了就是**

### 字符串替换（划重点）

**读入.less .jsx .xxx 等等文件，loader 一视同仁都是字符串按照 loader 内部的规则进行替换生成浏览器能读懂的 js，css**

比如 jsx 语法读入会将 html 标签一律转化为 react 的 \_createElement 函数

在 webpack 的配置中 loader 将定义在 module 的 rules 中，而 loader 有两个基础属性：

**test** 属性，用于标识出应该被对应的 loader 进行转换的某个或某些文件。
**use** 属性，表示进行转换时，应该使用哪个 loader。

```js
const config = {
  output: {
    filename: 'index.js',
  },
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
}
```

所以无论是 ts，jsx 亦或是 less，sass 甚至是以模块导入的 css 都是由 loader 来进行处理的，

如果在 loader 中想要进一步配置，loader 的 use 也能接收一个对象或者一个数组去多重转化

```js
const config = {
  output: {
    filename: 'index.js',
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: 'style-loader',
            options: {
              // 具体配置
            },
          },
        ],
        include: pathLib.resolve(__dirname, 'src'), // 用 test 去匹配的位置
        exclude: '/node_modules/', // 不去匹配的位置
      },
    ],
  },
}
```

## 4. plugin 插件

loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 new 操作符来创建它的一个实例。

**说人话，plugin 功能较多较全，不像 loader 那般专注于预处理文件**

```js
const HtmlWebpackPlugin = require('html-webpack-plugin') // 通过 npm 安装
const webpack = require('webpack') // 用于访问内置插件

module.export = {
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
  // hwp 可以直接生成一个html模板或者自己指定模板html的位置，然后会将output的出口文件插入html模板
  plugins: [new HtmlWebpackPlugin({ template: './src/index.html' })],
}
```

## 3. mode 模式

|            选项             |                                                                                                           描述                                                                                                            |
| :-------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| development<br>（开发模式） |                                                              会将 process.env.NODE_ENV 的值设为 development。启用 NamedChunksPlugin 和 NamedModulesPlugin。                                                               |
| production<br>（生产模式）  | 会将 process.env.NODE_ENV 的值设为 production。启用 FlagDependencyUsagePlugin, FlagIncludedChunksPlugin, ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin 和 UglifyJsPlugin. |

### mode: development

相当于

```js
module.exports = {
+ mode: 'development'
- plugins: [
  // 当开启 HMR(热模块更新) 的时候使用该插件会显示模块的相对路径
-   new webpack.NamedModulesPlugin(),
  // node 进程注入属性
-   new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
- ]
}
```

### mode: production

相当于

```js
module.exports = {
+  mode: 'production',
-  plugins: [
-    new UglifyJsPlugin(/* ... */), // 顾名思义丑化 js，在plugin中具体讲
-    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
-    new webpack.optimize.ModuleConcatenationPlugin(), // 对 js 预处理，进行变量提升加快运行速度
-    new webpack.NoEmitOnErrorsPlugin()
 // 在编译出现错误时，使用 NoEmitOnErrorsPlugin 来跳过输出阶段。这样可以确保输出资源不会包含错误。对于所有资源，
-  ]
}
```

## 5. devServer 本地服务

顾名思义 development-server

webpack 将项目打包存于内存中（也可以配置存于 dist 使用），node 开启本地服务访问项目

```js
module.export = {
  // devServer 一些常用配置
  devServer: {
    port: 3300,
    progress: true, // 打包启动时控制台显示进度
    contentBase: './dist', // 基于这个目录下开启服务
    open: true, // 启动完毕后自动打开页面
    compress: true, //gzip压缩
    hot: true, // 开启热模块更新，自动在plugin添加 webpack.HotModuleReplacementPlugin
    // 在所有响应中添加首部内容：
    headers: {
      'X-Custom-Foo': 'bar',
    },
  },
}
```

### devServer.proxy 设置反向代理

### devServer.historyApiFallback

当使用 HTML5 History API 时，任意的 404 响应都可能需要被替代为 index.html。
服务器部署时需于服务器配置文件上配置，如 nginx.conf

```js
module.export = {
  historyApiFallback: true,
}
```

通过传入一个对象，比如使用 rewrites 这个选项，此行为可进一步地控制：

```js
module.export = {
  historyApiFallback: {
    rewrites: [
      { from: /^\/$/, to: '/views/landing.html' },
      { from: /^\/subpage/, to: '/views/subpage.html' },
      { from: /./, to: '/views/404.html' },
    ],
  },
}
```

当路径中使用点(dot)（常见于 Angular），你可能需要使用 disableDotRule：

```js
module.export = {
  historyApiFallback: {
    disableDotRule: true,
  },
}
```

cli 开启

webpack-dev-server --history-api-fallback

待续...
