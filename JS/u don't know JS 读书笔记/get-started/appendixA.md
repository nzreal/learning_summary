# Appendix A

## Values vs. References

基础值 `primitive` 和引用值 `references`，懂得都懂，不赘述了。

## So Many Function Forms

基础的不说了，小小拓展一下

### prototype properties

#### arguments

函数实际传参的类数组，在函数运行时创建，可在函数内部上下文直接获取，在箭头函数中无此属性。

在日常业务开发中使用参数解构会更为直观。

```js
function a(test, ...restParams) {
  // restParams 更为直观
}
```


#### name

返回函数实例的名称，可分为函数被声明时赋值的 name，还有匿名函数表达式推断而来的 name。

顺带一提，bind 之后的 name 为 'bound xxx(原函数名)'。

```js
function a() {}
a.name // a

const b = {}
const c = a.bind(b)

c.name // bound a
```

#### length

函数形参列表，与之相对上面的 arguments.length 是实参列表

#### caller

在函数运行时创建，指向调用函数的作用域（函数），若为全局作用域则指向 Null（非标准，尽量不要在生产中使用）

```js
function a() {
  console.log(caller)
}

a() // null

function b() {
  a()
}

b() // f b() { a() }
```

#### toSource

返回函数的源代码的字符串表示（非标准）

### 多种函数声明方式

#### 函数声明

```js
function a() {}
a.name // a
```

#### 函数表达式(匿名)

```js
const a = function() {}
a.name // a
```

#### 函数表达式

有意思的是这次 `a.name` 就不是推断而来的 `a` 而是 `b`

```js
const a = function b() {}
a.name // b
```

#### 其他

```js
// IIFE 即执行函数
(function(){ .. })();

// async await
async function a() { await ... }

// generator
function *b() { yield ...; }
```