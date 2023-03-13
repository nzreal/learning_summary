# never, any, unknown, void

## any

any 肯定是大家一开始写 ts 最喜欢用的类型了，百无限制
不过使用了 any 后，用 ts 和使用 js 也就没有了差别，在生产代码中尽量不要使用 any

any 既是顶层类型，又是底部类型，可以理解为一个大区间，包含了所有类型，所有类型既是它的父类型，又是它的子类型

```ts
type test = number extends any ? true : false; // true
type test2 = any extends number ? true : false; // boolean 此处待细究
```

因此 any 类型可以被赋值给任意类型，任意类型也可以赋值给 any

```ts
JSON.parse(x: string): any
```

在 unknown 出来之前就定义了其返回值类型，不然可以是 unknown

## unknown

unknown 同样也是顶部类型，所有类型都是它的子类型

```ts
type test = number extends unknown ? true : false; // true
type test2 = unknown extends number ? true : false; // false
```

相比起 any 来说更加得类型安全，任何值都可以被赋值给 unknown, 但 unknown 不能赋值给除自己以外的其他值

而一般 unknown 的使用需要配合 as 进行断言

```ts
let a: unknown = 1;
a.toFixed(1); // 不进行断言会报错 Property 'toFixed' does not exist on type 'unknown'
(a as number).toFixed(2);
```

## never

never 是底部类型，即 never 是所有类型的子类型，never 没有自己的子类型

```ts
type test = number extends never ? true : false; // false
type test2 = unknown extends never ? true : false; // false
```

很多人不清楚 never 的具体用处

对于其定义是 never 表示的是那些永不存在的值的类型

其一般用处如下

1. 总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型。

2. 变量也可能是 Never 类型，当它们被永不为真的类型保护所约束时。

```ts
function catchError(msg: string): never {
  throw new Error(msg);
}

function loop(): never {
  while (true) {}
}
```

也可用于 exhaustive check，穷尽检查

```ts
interface Foo {
  type: 'foo';
}

interface Bar {
  type: 'bar';
}

interface Baz {
  type: 'baz';
}

type All = Foo | Bar | Baz;

function handleValue(val: All) {
  switch (val.type) {
    case 'foo':
      // 这里 val 被收窄为 Foo
      break;
    case 'bar':
      // val 在这里是 Bar
      break;
    default:
      // val 在这里是 Baz
      const exhaustiveCheck: never = val; // Type 'Baz' is not assignable to type 'never'.
      break;
  }
}

handleValue({ type: 'baz' });
```

检测上述情况中 是否穷举了 All 内部的类型

## void

void 可以说与 any 正好相反，无任何类型，一般使用在函数无返回值时使用

```js
function hello(): void {
    console.log("no return value");
}
```

同时也可以为变量定义 void 类型，不过只能为它赋予 undefined 、 null （注意，"strictNullChecks": true 时会报错）和 void 类型的值

```js
let void1: void
let null1: null = null
let und1: undefined = undefined
let void2: void

void1 = void2
void1 = und1 
void1 = null1 // Type 'null' is not assignable to type 'void'.
```
