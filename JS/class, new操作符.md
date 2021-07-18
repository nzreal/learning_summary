# class, new 操作符

这俩玩意没什么特别好说的了

class 其实就是构造函数的语法糖，其源码实现也浮现于脑中

```js
var Hello = (function () {
  function Hello(x) {
    this.x = x;
  }

  Hello.prototype.greet = function () {
    console.log(this.x);
  };

  return Hello;
})();
// 等价于
class Hello {
  constructor(x) {
    this.x = x;
  }
  greet() {
    console.log(this.x);
  }
}
```

那么思考一哈 new 操作符就是接受一个构造函数和其参数将此构造函数实例化，并继承其原型链上的函数和属性

```js
function myNew(func) {
  return function (...args) {
    // 创建一个新对象且将其隐式原型指向构造函数原型
    let obj = {
      __proto__: func.prototype,
    };
    // 执行构造函数
    func.call(obj, ...args);
    // 返回该对象
    return obj;
  };
}
```
