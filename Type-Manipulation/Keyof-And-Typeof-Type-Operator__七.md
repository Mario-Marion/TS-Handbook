---
highlight: vs2015
---
## Keyof 操作符

`keyof` 操作符接受一个对象类型，并生成其键的字符串或数值的联合类型。以下类型 `P` 为联合类型 "x" | "y"。
```ts
type Point = { x: number; y: number };
type P = keyof Point;
```
如果对象有字符串或数字索引签名，`keyof` 将返回这些类型:
```ts
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
// type A = number

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
// type M = string | number
```
注意，在例子中，`M` 是 `string | number`，这是因为 JavaScript 对象的键总是强转为字符串，允许`object[0]` 这种写法，最终都会被 JS 转为 `obj["0"]`。

当与映射类型结合使用时，`Keyof` 类型变得特别有用（可参考另一篇文章：[映射类型](https://juejin.cn/editor/drafts/7207299650323300411)）。

## Typeof 操作符

JavaScript 已经有一个 `typeof` 操作符，你可以在表达式上下文中使用:
```ts
// Prints "string"
console.log(typeof "Hello world");
```
TypeScript 也添加了 `typeof` 操作符，你可以在类型上下文使用它来引用变量或属性的类型：
```ts
let s = "hello";
let n: typeof s;
```
`typeof` 对于基本类型用处不大，但是结合其他类型操作符，可以方便地表示许多模式。

例如：Ts 内置工具类型 `ReturnType<T>`，它接收一个函数类型并产生它的返回类型：
```ts
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;
// type k = boolean
```
如果我们参数用函数名去使用 `ReturnType`，我们会得到一个报错：
```ts
function f() {
    return { x: 10, y: 3 };
}
type P = ReturnType<f>;
// error：'f' refers to a value, but is being used as a type here. Did you mean 'typeof f'?
```
报错意思是：函数 f 是一个值，但是却被当作类型使用，你是否打算使用 typeof f。

记住，值和类型不是一回事。要引用函数 f 的类型，得使用 `typeof`：
```ts
function f() {
    return { x: 10, y: 3 };
}
type P = ReturnType<typeof f>;
// p 类型为
// type p {
//     x: number;
//     y: number;
// }
```
### 限制
TypeScript 限制了使用 `typeof` 表达式的种类。

只有在标识符（即变量名）或它们的属性上使用 `typeof` 才是合法的。这有利于避免编写，你以为有执行但却没有执行的代码：
```ts
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
// Error
```
这代码思路是：想调用 `msgbox` 函数，然后返回值用 `typeof` 返回它的类型。

正确写法： `let shouldContinue: ReturnType<typeof msgbox>;`



感谢观看，如有错误，望指正

>官网地址：
>- keyof：<https://www.typescriptlang.org/docs/handbook/2/keyof-types.html>
>- typeof：<https://www.typescriptlang.org/docs/handbook/2/typeof-types.html>
>
>
>github 资料： <https://github.com/Mario-Marion/TS-Handbook>



