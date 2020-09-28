# Webpack 样式文件模块导入

之前小学webpack大概到了自己能配置webpack的入门程度, 牛刀小试自己写了个配置文件进行项目搭建
结果遇到了一个问题: 样式无法生效, 不多bb上代码

ts文件

```tsx
import React from "react"
import styles from './basicStyle.less'

const SideMenu: React.FC<{}> = ((props) => {
    return (
        <div className={styles.sideMenu} >
            123
        </div>
    );
});
```
less文件
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

改了下less文件导入及引用的写法

```tsx
import React from "react"
import './basicStyle.less'

const SideMenu: React.FC<{}> = ((props) => {
    return (
        <div className="sideMenu">
            123
        </div>
    );
});
```
这样就能成功渲染导入的样式了

这时刚入门webpack且对模块一无所知的我黑人问号脸???之前在用antdp框架写的时候可不是介个样子的啊

好的, 我们先来思考为何同样是className一种可以另一种却不可以, 我们来观察一下, 首先他们的写法不同(这不是废话吗),
 我们要知道这个写法不同为何会导致结果的分歧, 这时我们从打包出来的js文件看起

首先是打包出来的css部分(此处开发模式打包没有使用MiniCssExtractPlugins), 两者皆为一致

```js
var ___CSS_LOADER_API_IMPORT___ = __webpack_require__(/*! ../../../node_modules/css-loader/dist/runtime/api.js */ "./node_modules/css-loader/dist/runtime/api.js");
exports = ___CSS_LOADER_API_IMPORT___(false);
// Module
exports.push([module.i, ".sideMenu {\n  height: 100vh;\n  width: 256px;\n  background-color: #00a67c;\n}\n", ""]);
// Exports
module.exports = exports;
```
然后我们来看{style.sideMenu}对象传入className的打包部分

```js
var react_1 = __importDefault(__webpack_require__(/*! react */ "./node_modules/react/index.js"));
var basicStyle_less_1 = __importDefault(__webpack_require__(/*! ./basicStyle.less */ "./src/component/SideMenu/basicStyle.less"));

var TitleBar = (function (props) {
    return (react_1.default.createElement("div", null));
});
var SideMenu = (function (props) {
    return (react_1.default.createElement("div", { className: basicStyle_less_1.default.sideMenu },
        react_1.default.createElement(TitleBar, null),
        "123"));
});
exports.default = SideMenu;
```
再来看下字符串传入className的打包部分

```js
var react_1 = __importDefault(__webpack_require__(/*! react */ "./node_modules/react/index.js"));
__webpack_require__(/*! ./basicStyle.less */ "./src/component/SideMenu/basicStyle.less");

var TitleBar = (function (props) {
    return (react_1.default.createElement("div", null));
});
var SideMenu = (function (props) {
    return (react_1.default.createElement("div", { className: "sideMenu" },
        react_1.default.createElement(TitleBar, null),
        "123"));
});
exports.default = SideMenu;
```
好, 这时候就要开始大家一起来找茬环节, 很快就发现了不同之处

前者是以esModule模块的形式导入, 将其直接传给了className; 后者是直接导入less文件, 以传统html class 字符串的方式引入样式

好的字面意义上的写法不同, 这里产生的问题就是模块导入没有解析, 那么只要我们在webpack的配置中加上样式导入模块的解析就ok了吧

原来的样式解析(xixi图简便, 写的比较捞, 能用就行)
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
之后改了改将每个use抽离为getStyleLoader的公共函数

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
  ].filter(Boolean);
  if(preProcessor) {
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
  return loaders;
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
var SideMenu = (function (props) {
    return (react_1.default.createElement("div", { className: basicStyle_less_1.default.sideMenu },
        react_1.default.createElement(TitleBar, null),
        "123"));
});
```
SideMenu部分仍然是老老实实的导入模块的sideMenu
```js
var ___CSS_LOADER_API_IMPORT___ = __webpack_require__(/*! ../../../node_modules/css-loader/dist/runtime/api.js */ "./node_modules/css-loader/dist/runtime/api.js");
exports = ___CSS_LOADER_API_IMPORT___(false);
// Module
exports.push([module.i, ".Rn5PPvSCZzNGXHWaa1hPM {\n  height: 100vh;\n  width: 256px;\n  background-color: #00a67c;\n}\n", ""]);
// Exports
exports.locals = {
	"sideMenu": "Rn5PPvSCZzNGXHWaa1hPM"
};
module.exports = exports;
```
我们再来对比下样式导入的部分, 发现 诶, 多了行exports.locals, 而其上的exports的模块内容也发生了改变, 类名变成了Rn5PPvSCZzNGXHWaa1hPM

哈哈美滋滋, 但是这时再把它改为之前的字符串传入, 样式又不再生效了, 啊这...

虽然能用但是不够优雅

未完待续...还会继续改
