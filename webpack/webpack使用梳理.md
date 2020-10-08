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
};
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
};
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
};
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
};
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
};
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
};
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
};
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
};
```

## 4. plugin 插件

loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 new 操作符来创建它的一个实例。

**说人话，plugin 功能较多较全，不像 loader 那般专注于预处理文件**

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

module.export = {
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
  // hwp 可以直接生成一个html模板或者自己指定模板html的位置，然后会将output的出口文件插入html模板
  plugins: [new HtmlWebpackPlugin({ template: './src/index.html' })],
};
```

## 5. mode 模式

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

## 6. devServer 本地服务

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
    overlay: true, // 开启全局语法报错
    noInfo: true, // 启动时和每次保存之后，那些显示的 webpack 包(bundle)信息的消息将被隐藏。错误和警告仍然会显示。
    // 也可以接收一个对象 { warnings: false, errors: true }
    // 在所有响应中添加首部内容：
    headers: {
      'X-Custom-Foo': 'bar',
    },
  },
};
```

### devServer.proxy 设置反向代理

如果你有单独的后端开发服务器 API，并且希望在同域名下发送 API 请求 ，那么代理某些 URL 会很有用。

dev-server 使用了非常强大的 http-proxy-middleware 包。

```js
module.export = {
  proxy: {
    '/api': {
      target: 'http://localhost:3000',
      // 替换 /api
      pathRewrite: { '^/api': '' },
    },
  },
};
```

有时你不想代理所有的请求。可以基于一个函数的返回值绕过代理。

在函数中你可以访问请求体、响应体和代理选项。必须返回 false 或路径，来跳过代理请求。

例如：对于浏览器请求，你想要提供一个 HTML 页面，但是对于 API 请求则保持代理。你可以这样做：

```js
module.export = {
  proxy: {
    '/api': {
      target: 'http://localhost:3000',
      bypass: function (req, res, proxyOptions) {
        if (req.headers.accept.indexOf('html') !== -1) {
          console.log('Skipping proxy for browser request.');
          return '/index.html';
        }
      },
    },
  },
};
```

也可以配置多个代理

```js
module.export = {
  proxy: {
    '/api': {
      target: 'http://localhost:3000',
    },
    '/hehe': {
      target: 'http://localhost:8080',
    },
  },
};
```

或者以 context 来接收需转发的请求路径

```js
module.export = {
  proxy: [
    {
      context: ['/auth', '/api'],
      target: 'http://localhost:3000',
    },
  ],
};
```

默认情况下，不接受运行在 HTTPS 上，且使用了无效证书的后端服务器。如果你想要接受，修改配置如下：

```js
proxy: {
  "/api": {
    target: "https://other-server.example.com",
    secure: false
  }
}
```

### devServer.historyApiFallback history router 设置

当使用 HTML5 History API 时，任意的 404 响应都可能需要被替代为 index.html。
服务器部署时需于服务器配置文件上配置，如 nginx.conf

```js

  historyApiFallback: true,

```

通过传入一个对象，比如使用 rewrites 这个选项，此行为可进一步地控制：

```js
  historyApiFallback: {
    rewrites: [
      { from: /^\/$/, to: '/views/landing.html' },
      { from: /^\/subpage/, to: '/views/subpage.html' },
      { from: /./, to: '/views/404.html' },
    ],
  },
```

当路径中使用点(dot)（常见于 Angular），你可能需要使用 disableDotRule：

```js
module.export = {
  historyApiFallback: {
    disableDotRule: true,
  },
};
```

cli 开启

webpack-dev-server --history-api-fallback

### devServer.https https 服务

默认情况下，dev-server 通过 HTTP 提供服务。也可以选择带有 HTTPS 的 HTTP/2 提供服务：

```js
https: true;
```

以上设置使用了自签名证书，但是你可以提供自己的：

```js
https: {
  key: fs.readFileSync("/path/to/server.key"),
  cert: fs.readFileSync("/path/to/server.crt"),
  ca: fs.readFileSync("/path/to/ca.pem"),
}
```

## 7. externals 外部扩展

externals 配置选项提供了「从输出的 bundle 中排除依赖」的方法。相反，所创建的 bundle 依赖于那些存在于用户环境(consumer's environment)中的依赖。此功能通常对 library 开发人员来说是最有用的，然而也会有各种各样的应用程序用到它。

简单点，这样做的目的就是将不怎么需要更新的第三方库脱离 webpack 打包，不被打入 bundle 中，从而减少打包时间，但又不影响运用第三方库的方式，例如 import 方式等。

防止将某些 import 的包(package)打包到 bundle 中，而是在运行时(runtime)再去从外部获取这些扩展依赖(external dependencies)。
例如，从 CDN 引入 jQuery，而不是把它打包：

在模板页面引入

```html
<script
  src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous"
></script>
```

**webpack.config.js**

属性名称是 jquery，表示应该排除 import \$ from 'jquery' 中的 jquery 模块。为了替换这个模块，jQuery 的值将被用来检索一个全局的 jQuery 变量。换句话说，当设置为一个字符串时，它将被视为全局的（定义在上面和下面）。

```js
module.export = {
  externals: {
    // 前面为暴露出来的模块名
    // 后者为去全局搜索的变量
    jquery: 'jQuery';
  }
}
```

具有外部依赖(external dependency)的 bundle 可以在各种模块上下文(module context)中使用，例如 CommonJS, AMD, 全局变量和 ES2015 模块。外部 library 可能是以下任何一种形式：

- root：可以通过一个全局变量访问 library（例如，通过 script 标签）。
- commonjs：可以将 library 作为一个 CommonJS 模块访问。
- commonjs2：和上面的类似，但导出的是 module.exports.default.
- amd：类似于 commonjs，但使用 AMD 模块系统。

还可以接受以下语法：

#### 字符串数组

```js
module.exports = {
  externals: {
    subtract: ['./math', 'subtract'],
  },
};
```

subtract: ['./math', 'subtract'] 转换为父子结构，其中 ./math 是父模块，而 bundle 只引用 subtract 变量下的子集。该例子会编译成 require('./math').subtract;

#### 对象

<div style="background-color: #FFFFCC; padding: 15px;font-style:italic">
一个形如 { root, amd, commonjs, ... } 的对象仅允许用于 libraryTarget: 'umd' 这样的配置.它不被允许 用于其它的 library targets 配置值.（ libraryTarget 位于 output 中）
</div>
<br>

```js
module.exports = {
  externals: {
    lodash: {
      commonjs: 'lodash',
      amd: 'lodash',
      root: '_', // 指向全局变量
    },
  },
};
```

此语法用于描述外部 library 所有可用的访问方式。这里 lodash 这个外部 library 可以在 AMD 和 CommonJS 模块系统中通过 lodash 访问，但在全局变量形式下用 \_ 访问。subtract 可以通过全局 math 对象下的属性 subtract 访问（例如 window['math']['subtract']）。

详细参考：https://webpack.docschina.org/configuration/externals/
