# CH2

每个.js 文件可以被视为一个独立的程序而不是代码片段，在于 js 的错误处理方式。

## 变量引用值与原始值

1. null 与 undefined
2. 变量隐式转化 [JS 隐式转化](https://github.com/nzreal/learning_summary/blob/master/JS/js%E9%9A%90%E5%BC%8F%E8%BD%AC%E5%8C%96.md)

## Function

函数声明式（function declaration）和函数表达式（function expression）

声明具有函数提升，函数提升 > 变量提升

## ===

Object.is 可以视为 “====” 严格中的严格比较

```js
Object.is(+0, -0); // false
Object.is(NaN, NaN); // true
```

## ==

a == b 比较的双方若类型不一致则会进行类型隐式转化
[JS 隐式转化](https://github.com/nzreal/learning_summary/blob/master/JS/js%E9%9A%90%E5%BC%8F%E8%BD%AC%E5%8C%96.md)

## class

面向对象：封装，继承，多态

扩展：[class & new 原理](https://github.com/nzreal/learning_summary/blob/master/JS/class%2C%20new%E6%93%8D%E4%BD%9C%E7%AC%A6.md)

## Modules & ESM

ESM 模块再被导入实例化之后，任何地方再次 import 引入的都是这一单例
