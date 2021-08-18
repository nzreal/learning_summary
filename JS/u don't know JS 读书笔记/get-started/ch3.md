# CH3

## 1. 循环 & 迭代器

也许我们日常开发中接触不到迭代器，但是只要是涉及循环的地方就有迭代器存在
无论是 Array、Map、Set、[TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 甚至是 String 中都是内置迭代器（还有 function 中的 arguments 和 DOM 操作获取的 node list），可以通过访问 [Symbol.iterator](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator) (详见[迭代协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols))进行访问

### 内置迭代器的对象

一句话概括，只要是能用 `for ... of` 循环的便都是可迭代对象

> 小小扩展一下：for ... in 与其区别是循环对象内可枚举对象

包括 `...args` 展开运算符其内部实现也使用了同样的迭代协议

- [ ] object 没有迭代器为何可以使用 ...

#### String

```js
'nzreal'[Symbol.iterator]();
// StringIterator {}
```

#### Array

```js
['nzreal', 'handsome'][Symbol.iterator]();
// Array Iterator {}
```

#### TypedArray

```js
new Int8Array(8)[Symbol.iterator]();
// Array Iterator {}[[Prototype]]: Array Iterator
```

#### Set

```js
new Set(['nzreal', 'handsome'])[Symbol.iterator]();
// SetIterator {"nzreal", "handsome"}
//  [[Entries]]
//    0: "nzreal"
//    1: "handsome"
//  [[Prototype]]: Set Iterator
//  [[IteratorHasMore]]: true
//  [[IteratorIndex]]: 0
//  [[IteratorKind]]: "values"
```

Set 的默认迭代器方式是 `values`，默认迭代值是其 value 值，当然也可以通过 `entries` 来迭代 key-value

#### Map

```js
new Map([['nzreal', 'handsome']])[Symbol.iterator]();
// MapIterator {"nzreal" => "handsome"} 展开
//  [[Entries]]
//    0: {"nzreal" => "handsome"}
//  [[Prototype]]: Map Iterator
//  [[IteratorHasMore]]: true
//  [[IteratorIndex]]: 0
//  [[IteratorKind]]: "entries"
```

Map 的默认迭代器是 `entries`, 默认迭代 key-value

### 自定义迭代器

同时我们也可以为没有内置迭代器的对象自定义迭代器或者覆盖已有的迭代器
大致格式如下:

#### 迭代器协议

| 属性 | 值                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| next | 一个无参数函数，返回一个应当拥有以下两个属性的对象： <br><br>done（boolean）<br>如果迭代器可以产生序列中的下一个值，则为 false。（这等价于没有指定 done 这个属性。）<br><br>如果迭代器已将序列迭代完毕，则为 true。这种情况下，value 是可选的，如果它依然存在，即为迭代结束之后的默认返回值。<br><br>value <br><br>迭代器返回的任何 JavaScript 值。done 为 true 时可省略。<br><br>next() 方法必须返回一个对象，该对象应当有两个属性： done 和 value，如果返回了一个非对象值（比如 false 或 undefined），则会抛出一个 TypeError 异常（"iterator.next() returned a non-object value"）。 |

#### 可迭代协议

要成为可迭代对象，并且有默认的迭代行为，一个对象必须实现 @@iterator 方法。

| 属性              | 值                                                     |
| ----------------- | ------------------------------------------------------ |
| [Symbol.iterator] | 一个无参数的函数，其返回值为一个符合迭代器协议的对象。 |

---

综上两种协议，实现自己的迭代器格式如下：

```js
const myIterator = {
  next: function () {
    // ...
  },
  [Symbol.iterator]: function () {
    return this;
  },
};
```

> 备注：不可能判断一个特定的对象是否实现了迭代器协议，然而，创造一个同时满足迭代器协议和可迭代协议的对象是很容易的（如上面的示例中所示）。
>
> 这样做允许一个迭代器能被各种需要可迭代对象的语法所使用。因此，很少会只实现迭代器协议，而不实现可迭代协议

## 2. 闭包

TODO

- [ ] 可以单拎一篇出来讲

## 3. this

有一个误区是 this 指向 function 自身，其实不然
this 可以简单理解为调用 function 的对象，严格意义上 this 指向 function 执行时的执行上下文

```js
function a() {
  console.log(this)
}

a() // window
```

为什么需要 this 呢，因为 JS 的作用域是静态作用域，在定义函数时便已经决定好其作用域，为了让 function 能够访问到作用域外的变量，便有了 this

使用 apply

```js
function a() {
  console.log(this.name)
}

function b() {
  const test = {
    name: 'hehe',
  }
  a.apply(test)
}

b() // hehe
```

>The benefit of this-aware functions—and their dynamic context—is the ability to more flexibly re-use a single function with data from different objects. A function that closes over a scope can never reference a different scope or set of variables. But a function that has dynamic this context awareness can be quite helpful for certain tasks

这种带有 `this` 的函数及其动态上下文的好处是更加灵活，同一个函数可以复用来自不同对象的数据。在一个作用域上关闭的函数无法引用另一个作用域内的变量或变量集。但是一个具有动态上下文的 `this` 函数在某些场景下将会十分有用。

 **TODO**

- [ ] 需要再深入学习下执行上下文

## 4. 原型 prototype

[原型链](../原型链.md)

### Object Linkage

可以通过 `Object.create()` 来创建对象之间的原型引用

```js
const people = {
  walk: 1
}

const Mike = Object.create(people)
// {}
//  [[Prototype]]: Object
//    walk: 1
//    [[Prototype]]: Object
```

同样也可以使用 `Object.create(null)` 来创建没有原型的对象，在某些情况下会使得对象更加安全

### `this` revisited

```js
const feat = {
  call() {
    console.log(`say my name: ${this.name}`)
  }
}

const a = Object.create(feat)
const b = Object.create(feat)
a.name = 'a'
b.name = 'b'

a.call() // a
b.call() // b
```
当 `this` 与 `link` 搭配就会产生奇妙的化学反应，`a` 和 `b` 以`feat`为原型创建，换句话说，`feat`作为抽象类被 `a` 和 `b` 继承，js class 的继承也便是使用原型链的方式，或者是以上的方式实现。