# Webpack 样式文件模块导入

之前小学 webpack 大概到了自己能配置 webpack 的入门程度, 牛刀小试自己写了个配置文件进行 react 项目搭建
结果遇到了一个问题: 样式无法生效, 不多 bb 上代码

**组件**

```tsx
import React from 'react'
import styles from './basicStyle.less'

const SideMenu: React.FC<{}> = (props) => {
  return <div className={styles.sideMenu}>123</div>
}
```

**样式文件**

```less
.sideMenu {
  height: 100vh;
  width: 256px;
  background-color: #00a67c;

  .titleBar {
  }
}
```

结果样式没有生效, 为啥呢?

改了下 less 文件导入及引用的写法

```tsx
import React from 'react'
import './basicStyle.less'

const SideMenu: React.FC<{}> = (props) => {
  return <div className="sideMenu">123</div>
}
```

这样就能成功渲染导入的样式了

这时刚入门 webpack 且对模块一无所知的我黑人问号脸???怎么肥四

好的，我们先来思考为何同样是 className 前者 {style.sideMenu} 导入不行但后者直接字符串导入可以, 我们来观察一下, 首先他们的写法不同(这不是废话吗)，我们要知道这个写法不同为何会导致结果的分歧, 这时我们从打包出来的 js 文件看起

首先是打包出来的 css 部分(此处开发模式打包没有使用 MiniCssExtractPlugins), 以字符串传入和对象传入的都为一致

**css 打包部分**

```js
var ___CSS_LOADER_API_IMPORT___ = __webpack_require__(
  /*! ../../../node_modules/css-loader/dist/runtime/api.js */ './node_modules/css-loader/dist/runtime/api.js'
)
exports = ___CSS_LOADER_API_IMPORT___(false)
// Module
exports.push([
  module.i,
  '.sideMenu {\n  height: 100vh;\n  width: 256px;\n  background-color: #00a67c;\n}\n',
  '',
])
// Exports
module.exports = exports
```

**className 对象传入的打包部分**

```js
var react_1 = __importDefault(
  __webpack_require__(/*! react */ './node_modules/react/index.js')
)
var basicStyle_less_1 = __importDefault(
  __webpack_require__(
    /*! ./basicStyle.less */ './src/component/SideMenu/basicStyle.less'
  )
)

var TitleBar = function (props) {
  return react_1.default.createElement('div', null)
}
var SideMenu = function (props) {
  return react_1.default.createElement(
    'div',
    { className: basicStyle_less_1.default.sideMenu },
    react_1.default.createElement(TitleBar, null),
    '123'
  )
}
exports.default = SideMenu
```

**className 字符串传入的打包部分**

```js
var react_1 = __importDefault(
  __webpack_require__(/*! react */ './node_modules/react/index.js')
)
__webpack_require__(
  /*! ./basicStyle.less */ './src/component/SideMenu/basicStyle.less'
)

var TitleBar = function (props) {
  return react_1.default.createElement('div', null)
}
var SideMenu = function (props) {
  return react_1.default.createElement(
    'div',
    { className: 'sideMenu' },
    react_1.default.createElement(TitleBar, null),
    '123'
  )
}
exports.default = SideMenu
```

好, 这时候就要开始大家一起来找茬环节, 很快就发现了不同之处

**less 文件的导入不同以及 className 处的传值不同**

其实前文所述的以对象传入 className 说白了就是 css module 以模块导入类名，js 早就有模块化了，我大 css 不也得跟上？（其实并没有）

css 本身是没有模块化这一说的，但是多亏了 webpack 的 loader ，才诞生出了伪 css 模块
webpack 的 loader 我个人总结，说直白点就是字符串替换，包括就好比 css 的预处理语法 less，sass，stylus 等最终都会由 loader 转化为 css，同样的 css-loader 也可以将本来没有模块化功能的 css 伪造成对于开发者拥有模块化

原来的样式解析(xixi 图简便, 写的比较捞, 能用就行)

```js
{
   test: /\.css$/,
   exclude: /\.module\.css$/,
   use: [
          'style-loader',
          'css-loader',
          'postcss-loader'
   ]
},
{
    test: /\.less$/,
    use: [
      'style-loader',
      'css-loader',
      'less-loader'
    ]
},
```

参照框架中的 webpack 配置
之后改了改将每个 use 抽离为 getStyleLoader 的公共函数

```js
const getStyleLoaders = (cssOptions, preProcessor) => {
  const loaders = [
    isEnvProduction && MiniCssExtractPlugin.loader,
    isEnvDevelopment && require.resolve('style-loader'),
    {
      loader: require.resolve('css-loader'),
      options: cssOptions,
    },
    {
      loader: require.resolve('postcss-loader'),
      options: {
        ident: 'postcss',
        plugins: () => [
          require('postcss-flexbugs-fixes'),
          require('postcss-preset-env')({
            autoprefixer: {
              flexbox: 'no-2009',
            },
            stage: 3,
          }),
          postcssNormalize(),
        ],
        sourceMap: false,
      },
    },
  ].filter(Boolean)
  if (preProcessor) {
    loaders.push(
      {
        loader: require.resolve('resolve-url-loader'),
        options: {
          sourceMap: false,
        },
      },
      {
        loader: require.resolve(preProcessor),
        options: {
          sourceMap: true,
        },
      }
    )
  }
  return loaders
}
```

```js
{
        // oneOf 只匹配一种规则
        oneOf: [
          {
            test: cssRegex,
            use: getStyleLoaders({
              importLoaders: 1,
              sourceMap: false,
              // 在此处modules开启模块化即可
              modules: true,
            }),
          },
          {
            test: lessRegex,
            use: getStyleLoaders({
                importLoaders: 3,
                sourceMap: false,
                modules: true,
              },
              'less-loader'
            ),
          },
        ]
      },
```

感天动地, 样式它千呼万唤始出来, 然后我们再来瞅瞅打包文件这娃

```js
var SideMenu = function (props) {
  return react_1.default.createElement(
    'div',
    { className: basicStyle_less_1.default.sideMenu },
    react_1.default.createElement(TitleBar, null),
    '123'
  )
}
```

SideMenu 部分仍然是老老实实的导入模块的 sideMenu

```js
var ___CSS_LOADER_API_IMPORT___ = __webpack_require__(
  /*! ../../../node_modules/css-loader/dist/runtime/api.js */ './node_modules/css-loader/dist/runtime/api.js'
)
exports = ___CSS_LOADER_API_IMPORT___(false)
// Module
exports.push([
  module.i,
  '.Rn5PPvSCZzNGXHWaa1hPM {\n  height: 100vh;\n  width: 256px;\n  background-color: #00a67c;\n}\n',
  '',
])
// Exports
exports.locals = {
  sideMenu: 'Rn5PPvSCZzNGXHWaa1hPM',
}
module.exports = exports
```

我们再来对比下样式导入的部分, 发现 诶, 多了行 exports.locals, 而其上的 exports 的模块内容也发生了改变, 类名变成了 Rn5PPvSCZzNGXHWaa1hPM

哈哈美滋滋, 但是这时再把它改为之前的字符串传入, 样式又不再生效了, 啊这...

虽然能用但是不够优雅

未完待续...还会继续改
