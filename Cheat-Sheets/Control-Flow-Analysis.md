> 建议把 Handbook 部分（[官网链接](https://www.typescriptlang.org/docs/handbook/intro.html) 或 [掘金链接](https://juejin.cn/post/7207743145998925861)）全看完，再看本章节
# 控制流分析（Control Flow Analysis）

## 描述

控制流分析（Control Flow Analysis 简称：CFA） 几乎总是采用联合类型，并且基于你的代码逻辑去减少联合类型里面的类型数量。

大多数时候，CFA 在 JavaScript 布尔逻辑中工作，但是有一些方法可以定义你的函数，这些函数会影响 TypeScript 缩窄类型的方式。

**简单说就是：根据代码上下文可以推断出当前变量（或属性，等）的类型。**

## if 语句

大多数缩窄来自 if 语句中的表达式，用不同的类型操作符，在新作用域内进行窄化

### typeof (用于判断原始类型)

```ts
const input = getUserInput()
input // string | number
if (typeof input === "string") {
  input // string
}
```

### instanceof (用于判断类)

```ts
const input = getUserInput()
input // string | number[]
if (input instanceof Array) {
  input // number[]
}
```

### "property" in object (判断属性是否属于对象)

```ts
const input = getUserInput()
input // string | {error: ...}
if ("error" in input) {
  input // {error: ...}
}
```

### 类型守卫函数（type-grad function） (用于任何东西)

```ts
const input = getUserInput()
input // number | number[]
if (Array.isArray(input)) {
  input // number[]
}
```

## 表达式（Expressions）

当进行布尔运算时，缩窄也发生在代码的同一行

```ts
const input = getUserInput()
input // string | number
const inputLength = (typeof input === "string" && input.length) || input
　　　　　　　　　　　　　　　　　　　　　　  // && input: string
```

## 识别联合类型（Discriminated Unions ）

```ts
type JSONResponse = {status: 200, data: any}
| {status: 300, to: string}
| {status: 400, error: Error}
```

所有联合成员都有相同属性名称，CFA (Control Flow Analysis) 能识别对待

```ts
const response = getResponse()
response // JSONResponse

switch(response.status) {
  case 200: response.data
  case 400: redirect(response.to)
  case 500: response.error
}
```

## 类型守卫（Type Guards）之类型谓词（type predicates）

函数返回类型是类型谓词，可以当成类型守卫，用于 CFA

下面例子中，`isErrorResponse` 就是类型守卫（以上例子，检查 `typeof`、`instanceof` 等的返回值，都是类型守卫）
```ts
function isErrorResponse(obj: Response): obj is APIErrorResponse {
    return obj instanceof APIErrorResponse
}
```
用法：
```
const response = getResponse()
response // Response | APIErrorResponse

switch(isErrorResponse(response)) {
    response // APIErrorResponse
}
```

##  断言函数（Assertion Functions）

上边例子中，是根据自定义类型守卫函数返回的返回值，在新作用域中表示特定类型

而断言函数是抛出，不是返回值，从而改变当前作用域表示特定类型

```ts
function assertResponse(obj: any): asserts obj is ErrorResponse {
  if (!(obj instanceof JSONResponse)) {
    throw new Error("Not a success!")
  }
}
const res = getResponse();
res // SuccessResponse | ErrorResponse
assertResponse(res) // 断言函数更改当前作用域
res // ErrorResponse
```

## 赋值（Assignment）

使用 "as const" 缩窄类型

对象中的属性值被视为可变的，在赋值过程中，类型将被“拓宽”为非字面量类型。使用 "as const" 将会把所有属性值类型锁定为它们的字面量类型。

```ts
const data1 = { name: "Zagreus" }
const data2 = { name: "Zagreus" } as const
// data1 类型：data1: {name: string}
// data2 类型：data2: { readonly name: "Zagreus"}
```

跟踪相关变量

```ts
class SuccessResponse { }
const response = getResponse()
const isSuccessResponse = response instanceof SuccessResponse
if (isSuccessResponse) {
  response // SuccessResponse
}
```

重新赋值更新类型

```ts
let data: string | number = Math.random() ? "asd" : 123
data // string | number
data = "hello"
data // string
data = 123
data // number
data = {} // Error：type '{}' is not assignable to type 'string | number'
data // string | number
```
赋值错误类型，`data` 的类型又变回了 `string | number`

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/static/TypeScript%20Control%20Flow%20Analysis-8a549253ad8470850b77c4c5c351d457.png>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
