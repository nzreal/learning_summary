# Ch4

这一章算是对之后具体大章的先导介绍吧

## 1. Scope and Closure

作用域(Scope)是绝大多数语言所具有的的一种特性，JS拥有两种作用域：**块级作用域** 和 **函数作用域**。

作用域帮助我们更好的划分代码界限，抽象、独立代码功能。在作用域中，代码可以访问同级及向上访问作用域中的变量，但无法向下。  

像大多数语言一样，JS 是词法作用域，也是静态作用域，作用域在编译的那一刻就已经定好，但 JS 也有自己标志性的两大特性：**变量提升** 和 **闭包**。

### 暂时性死区（Temporal Dead Zone）

```js
console.log(a) // undefined
var a
```

懂得都懂，变量提升不再赘述

```js
console.log(a) // Uncaught ReferenceError: b is not defined
let a
```
在 `let` / `const` 变量定义前引用变量会报错，这就是 **暂时性死区**（以下简称 `TDZ` ）

 ES6 标准中对 let/const 声明中的解释 [第13章](https://link.segmentfault.com/?url=http%3A%2F%2Fwww.ecma-international.org%2Fecma-262%2F6.0%2F%23sec-let-and-const-declarations)，有如下一段文字
 > The variables are created when their containing Lexical Environment is instantiated but may not be accessed in any way until the variable’s LexicalBinding is evaluated.

 翻译一下：

 > 当JS程序在 module function 或 block 作用域进行实例化时，在此作用域中用 let / const 声明的变量会先在作用域中被创建，但因此时还未进行词法绑定，所以无法访问。

 因此，在这运行流程进入作用域创建变量，到变量可以被访问之间的这一段时间，就称之为 TDZ。

 在 TDZ 的影响下，我们需要在变量声明之后再引用变量（虽然本该如此...）

#### TODO

- [ ] 探究 JS 文件具体解析执行过程
- [ ] 探究为何要设计变量提升这一特性

## Prototype

JS 可以在定义一个 Object structure 前去实例化一个对象，这在计算机语言中是十分罕见的，这个特性也是由于原型以及原型链。

在 ES6 的 `class` 特性出来前，开发者们通常使用 `原型链继承`，或是用函数模拟 `class`。 现在的 `class` 的继承特性仍然是基于原型链，详细请参考 [class, new](../../class&new.md)。

## Types & Coercion

即使使用 TS / Flow 等静态类型，也应该去理解 JS 自身的类型与隐式转换，详细请参考 [js隐式转化](../../js隐式转化.md)。