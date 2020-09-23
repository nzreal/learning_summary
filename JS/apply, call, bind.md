# apply, call, bind

apply，call，bind 这三兄弟大家应该都不陌生，都可以改变函数的执行上下文(Execution context)，也可以说改变函数的 this 指向

这么一听还有点复杂，啥？执行上下文？哈？this 指向我也还不太懂

我们先来瞅瞅其应用场景

如

```js
// 我常用的检验类型
const checkType = (target) =>
  Object.prototype.toString.call(target).slice(8, -1);

// 遍历伪数组HTMLCollection
const elements = document.getElementsByClassName('xixi');
Array.prototype.forEach.call(elements, () => {});
```

不难发现相当多的 call，apply 应用于 prototype 之上，因为定义在原型上的函数可以被实例继承，实例调用函数时 this 便指向该实例(对原型链还抱有疑问，可查看我的原型链图片哦)，因此巨大多数的 prototype 上定义的函数内部实现都是使用 this，因此使用 call 改变 this 指向也就是将 this 指向你传参进入的 target，及用 target 去调用该函数，换句话说也是将该函数定义在 targe 上调用，好了这下原理也懂了

好的预习完毕，我们就来讲一讲他们的区别和源码实现

首先我们将其一分为二，兄弟分家，其心必异

## apply，call

apply 和 call 皆为即时调用函数，但是传参上有一点小小的区别

```js
function fn(a, b) {
  console.log(a, b);
}

fn.call(null, a, b);
fn.apply(null, [a, b]);
```

非常明显的 call 一个一个接受参数，而 apply 接收一个数组(伪数组?)

一开始我想当容易记混索性记了个口诀，we should call somebody one by one，我们打电话需要一个一个来，一个一个传参，xixi

那么 call 内部实现就是传参...args 或者从 arguments 中获取

结合上面所叙将函数定义在 target 上调用，由此我们可以模仿其功能的写出源码

```js
Function.prototype.newCall = function (context, ...args) {
  // 函数内的this指向调用call的函数即我们要使用的函数
  // context为传入对象即要改变的this指向
  //  不传context的时候便是window调用该函数与直接运行无异
  const ctx = context || window;
  // 添加uuid防止与其prototype上其他函数冲突造成覆盖
  const fnName = (this.name || 'function') + UUID();
  ctx[fnName] = this;

  ctx[fnName](...args);
  // 运行完后删去对象上的函数
  delete ctx[fnName];
};
// apply同理可得，然后我们再加上一些健壮性判断
Function.prototype.newApply = function (context, array) {
  if (typeof context !== 'object') {
    throw new Error(context + ' is not an object!');
  }
  // 判断 this 是否是函数
  if (typeof this !== 'function') {
    throw new Error(this + ' is not a function!');
  }
  // 判断第二个参数是否是一个数组
  // arguments[1].__proto__ === Array.prototype
  if (array && !array instanceof Array) {
    throw new Error('the Args should be an Array');
  }

  const ctx = context || window;
  const fnName = (this.name || 'function') + UUID();
  ctx[fnName] = this;
  ctx[fnName](array ? ...array: undefined);
  delete ctx[fnName];
};
```

## bind

bind 为啥要分家呢，因为他的兄弟俩都过于急躁刚改变了 this 就要调用，bind 看不太下去，他可是个沉稳的小伙子因为他改变了 this 指向后只是静候佳音，等待调用

react 中就经常有用到 bind 进行绑定

```js
class Foo extend PureComponent {
  constructor(props) {
    super(props);
    this.state = {}
    // bind将fn执行上下文绑定于当前class实例 箭头函数也可以
    this.fn = this.fn.bind(this)
  }
  fn() {}
  render() {
    return <div></div>
  }
}
```

因此 bind 的改变 this 指向是永久性的
因此关于其源码实现我们可以理解为 bind 后返回一个函数，函数包裹的是返回该函数调用 call

```js
Function.prototype.newBind = function (context) {
   const ctx = context || window
   const fn = this
   // bind 可接收预定义参数
   const bindArgs = Array.from(arguments).slice(1)
   return function (...args) {
      return args ? fn.call(ctx, ...bindArgs, ...args) : fn.call(ctx)
   }
}
```

当该 bind 后的函数作为构造函数时，会微妙的出现一些问题

这个时候可以参考我的另一篇文章https://github.com/nzreal/learning_summary/blob/master/JS/class%2C%20new%E6%93%8D%E4%BD%9C%E7%AC%A6.md

因为作为 new 的构造函数时，会把实例的隐式原型链 __proto__ 指向 new 操作符后的构造函数 fn 的原型 prototype ，而 fn 的 prototype 和其指向的函数的 prototype 是不一样的，所以需要把prototype也继承过来

这是《JavaScript Web Application》一书中对 bind()的实现：通过设置一个中转构造函数 fNOP，使绑定后的函数与调用 bind()的函数处于同一原型链上，用 new 操作符调用绑定后的函数，返回的对象也能正常使用 instanceof，因此这是最严谨的 bind()实现。

```js
Function.prototype.newBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError(
      'Function.prototype.bind - what is trying to be bound is not callable'
    );
  }

  const aArgs = Array.from(arguments).slice(1),
    fToBind = this,
    // 将原型进行中转的中转站构造函数
    fNOP = function () {},
    fBound = function () {
      // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
      return fToBind.apply(
        this instanceof fBound ? this : context,
        // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
        aArgs.concat(Array.prototype.slice.call(arguments))
      );
    };
  // 维护原型关系
  if (this.prototype) {
    // 当执行Function.prototype.bind()时, this为Function.prototype
    // this.prototype(即Function.prototype.prototype)为undefined
    fNOP.prototype = this.prototype;
  }
  // 下行的代码使fBound.prototype是fNOP的实例,因此
  // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
  fBound.prototype = new fNOP();
  return fBound;
};
```
