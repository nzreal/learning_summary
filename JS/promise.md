# Promise

记录一下自己写的简易版 promise 源码

promise 的主要特性，初始状态为 pending，resolve 后为 fulfilled，reject 后为 rejected

resolve 后执行 then

```js
const PENDING = 'pending';
const FULFILLED = 'fullFilled';
const REJECTED = 'rejected';
const isFunction = (variable) => typeof variable === 'function';

class EasyPromise {
  constructor(executor) {
    if (!isFunction(executor)) {
      throw new Error('Promise must accept a function as a parameter');
    }

    this.status = PENDING;
    this.value = undefined;
    this.onResolveCallbacks = [];
    this.onRejectedCallbacks = [];

    const resolve = (value) => {
      if (this.status === PENDING) {
        this.status = FULFILLED;
        this.value = value;
        this.onResolveCallbacks.forEach((fn) => fn(this.value));
      }
    };

    const rejected = (value) => {
      if (this.status === PENDING) {
        this.status = REJECTED;
        this.value = value;
        this.onRejectedCallbacks.forEach((fn) => fn(this.value));
      }
    };

    try {
      executor(resolve, rejected);
    } catch (err) {
      rejected(err);
    }
  }

  then(onFulFilled, onRejected) {
    onFulFilled = isFunction(onFulFilled) ? onFulFilled : (val) => val;

    onRejected = isFunction(onRejected)
      ? onRejected
      : (error) => {
          throw error;
        };

    if (this.status === FULFILLED) {
      onFulFilled(this.value);
    } else if (this.status === REJECTED) {
      onRejected(this.value);
    } else if (this.status === PENDING) {
      // 函数返回的是其结果, 所以then内传入的函数需要再次用函数包裹
      // push(() => onFulFilled(this.value))
      this.onResolveCallbacks.push(onFulFilled);
      this.onRejectedCallbacks.push(onRejected);
    }
  }
}
```

简易版 Promise 完成了，但是 Promise 的链式调用 then 算是 Promise 的精髓之一，以后再来实现，还有 finally 和 Promise.all 或 Promise.race 待实现
