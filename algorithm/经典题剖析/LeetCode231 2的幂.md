# LeetCode 231. 2 的幂

给定一个整数，编写一个函数来判断它是否是 2 的幂次方。

示例  1:

> 输入: 1
>
> 输出: true
>
> 解释: 20 = 1

示例 2:

> 输入: 16
>
> 输出: true
>
> 解释: 24 = 16

示例 3:

> 输入: 218
>
> 输出: false

这是一道比较经典的算法题, 看到的第一眼哈哈直接暴力, 奥利给没毛病

### 1. 暴力解法

```js
var isPowerOfTwo = function (n) {
  if (n <= 0) return false;
  if (n == 1) return true;
  var flag = true;
  while (n > 1) {
    if (n % 2) {
      flag = false;
    }
    n /= 2;
  }
  return flag;
};
```

### 2. 按位与算法

```js
var isPowerOfTwo = function (n) {
  return n > 0 && (n & (n - 1)) == 0;
};
```

&号是将前后的数各转为二进制数相与，2 的次方根转为二进制时首位为 1 后续都是 0，10xxx00，减一后退一位第一位为 0 后续皆为 1，01xxx11，则相与为 0
