---
highlight: vs2015
---
> 建议把 Handbook 部分（[官网链接](https://www.typescriptlang.org/docs/handbook/intro.html) 或 [掘金链接](https://juejin.cn/post/7207743145998925861)）全看完，再看本章节
# Interface

## 描述

用来描述对象的形状，能够被扩展。

JavaScript 中几乎所有的东西都是一个对象，而接口的构建是为了匹配它们的运行时行为。

## 常用语法 ( Common Syntax )

### 1. 描述普通对象

```ts
interface JsonResponse {
  version:number;
  outOfStock?: boolean; // 可选属性
  readonly body: string; // 只读属性
  update: (retryTimes: number) => void; // 箭头函数方法
  update2(retryTimes: number):void // 函数声明方法
}
interface JsonResponse2 { [key: string]: number } // 索引签名，接受字符串索引，值为 number
```

### 2. 描述函数

Interface 也可以直接来描述一个函数。因为在 JS 中，一切皆是对象，函数在 JS 中也是对象，可以拥有属性，并且可以被调用。

```ts
interface JsonResponse {
  (): string;
  toFn: string
}
const fn: JsonResponse = () => {
  return 'str'
}
fn.toFn = 'content'
```

### 3. 描述构造函数
一般用于描述具体类，因为抽象类没有构造函数（可参考：[抽象构造签名](https://juejin.cn/post/7202898170613907517/#heading-40)）
```ts
interface ConstructorFn { new(s: string): any }

function fn(S: ConstructorFn) {
  new S('jhjh')
}

class JsonResponse { }
fn(JsonResponse) // OK

function ConstructorFn2() { }
fn(ConstructorFn2)
// Error:
// Argument of type '() => void' is not assignable to parameter of type 'ConstructorFn'.
// Type '() => void' provides no match for the signature 'new (s: string): any'.'
```
例子中，`fn` 传入普通函数 `ConstructorFn2` 会报错
### 4. 附加注释，鼠标移入时编辑器会有附加注释

```ts
interface JsonResponse {
  version: number
}
interface JsonResponse {
  /** In bytes */
  payloadSize: number
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/657c1d0035e346d4aa28211f6eba4922~tplv-k3u1fbpfcp-zoom-1.image)

 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a224f30d87934c37903f6d31e7c464e7~tplv-k3u1fbpfcp-zoom-1.image)

### 5. 扩展

```ts
// 接口扩展接口
interface X { x: number }
interface Point extends X { y: number }
// Point {x:numer;y:number}

// 后续扩展，同属性必须声明同一类型，或是子类型，否则报错！
// 错误，接口 Point 错误的扩展接口 X，类型属性 x 是不兼容的，string 类型不能分配给 number 类型
interface X { x: number }
interface Point extends X { x: string }
// 正确，456 为 number 的子类型
interface X { x: number }
interface Point extends X { x: 456 }

// 也可扩展类型别名，并且可同时扩展多个类型别名和接口
type Y = { y: number }
interface Point2 extends X, Y { z: number }
// Point2 {x:numer;y:number;z:number}

```

##  泛型（ Generics ）

```ts
interface APICall<R> {
  data: R
}
// 使用
interface JsonResponse { content: string };
const api: APICall<JsonResponse> = { data: { content: 'xxx' } }
api.data.content
```

使用 extends 约束泛型参数

```ts
// 意味着要有 status 属性的类型才能使用
interface APICall<R extends { status: number }> {
  data: R
}
// 使用
interface JsonResponse { content: string, status: number };
const api: APICall<JsonResponse> = { data: { content: 'xxx', status: 200 } }
api.data.status
```

##  重载（ Overloads ）
可调用的接口（函数/方法），也可以有重载写法
```ts
interface Expect {
  (input: boolean): string;
  (input: string): boolean;
}

const expect: Expect = (input: boolean | string): any => {
  if (typeof input === "boolean") {
    return input ? "Passed" : "Failed";
  } else if (typeof input === "string") {
    return input === "Passed";
  }
};
```

## 属性修饰符 Getters & Setters

可以描述对象属性修饰符 getters 与 setters

```ts
interface Ruler {
  get size(): number
  set size(value: number | string)
}

const ruler:Ruler = {
  size: 123
}
ruler.size = 456
ruler.size = '456'
ruler.size  // 类型为 number
ruler.size = false // Error
```
如果 setter 参数没有指定类型，那么会根据 getter 的返回值进行推断

## 通过合并扩展（Extension via merging）

多个同名接口是可以合并扩展的

```ts
interface Legged {
  numberOfLegs: number;
}
interface Legged {
  numberOfLegs: 123;
}
// 报错，numberOfLegs 必须都为 number，就算 123 是 number 的子类型都不行
```
在相同 namespace 中同名接口也会合并扩展：
```
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
}
namespace Animals {
  export interface Legged {
    numberOfHands: number;
  }
}
// 合并为
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
    numberOfHands: number;
  }
}
```

## 一致性类 （ Class conformance ）

可通过 `implements` 关键字来确保类的一致性

```ts
interface Syncable { sync(): void }
class Account implements Syncable {
  sync() { }
}
// 必须实现 sync 方法
```

## 内置原始类型

`boolean`、`string`、`number`、`undefined`、`null`、`any`、`unknown`、`never`、`void`、`bigint`、`symbol`

## 类型字面量

**Object：**`{ fild: string }`

**Function：**`(arg: number)=> string`

**Arrays：**`string[]` or `Array<string>`

**Tuple：**`[string, number]`

## 避免使用类型

`Object`、`String`、`Number`、`Boolean`

## 常用内置 JS 对象

`Date`、`Error`、`Array`、`Map`、`Set`、`Regexp`、`Promise`


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/static/TypeScript%20Interfaces-34f1ad12132fb463bd1dfe5b85c5b2e6.png>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

