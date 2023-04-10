---
highlight: vs2015
---
> 建议把 Handbook 部分（[官网链接](https://www.typescriptlang.org/docs/handbook/intro.html) 或 [掘金链接](https://juejin.cn/post/7207743145998925861)）全看完，再看本章节
# Types

## 描述

全称叫做 '类型别名'，为类型字面量提供名称。比 Interface 支持更丰富的类型系统特性。

## Type 与 Interface 区别

-   Interface 只能描述对象的形状，而 Type 不止
-   Interface 能多次声明进行扩展，而 Type 不行
-   在性能关键类型中，"接口比较检查" 可以更快。

## 把类型别名想象成变量

和变量语义类似，可以在不同的作用域中创建具有相同的名称。

## 工具类型（Utility Types）

TypeScript 内置了许多全局类型，将帮助你在类型系统中完成常用的任务。

## 对象字面量语法（ Object Literal Syntax ）

```ts
type JsonRsponse = {
  version: number;
  outOfStock?: boolean; // 可选属性
  readonly body: string; // 只读属性
  /** In bytes */        // 编辑器注释提示
  payloadSize: number;   //
  update: (retryTimes: number) => void; // 箭头函数方法
  update2(retryTimes: number): void; // 声明函数方法
  (): JsonRsponse; // 可调用类型（函数）
  [key: string]: number; // 索引签名，接受字符串索引，值为number
  new (s: string): JSONResponse; // 构造函数类型
}
```

这些对象字面量的语法和 Interface 的没有区别，详情解释可参考：[接口 Interfaces-TypeScript 官网Cheat Sheets](https://juejin.cn/post/7202576202288414778)

## 原始类型（ Primitive Type ）
主要对文档编写有用
```ts
type SanitizedIput = string
type MissingNo = 404
```
## 对象字面量类型
```ts
type Location = {
    x: number;
    y: number;
}
```
## 元组类型（Tuple Type）

元组是明确知道索引元素的类型和数组长度的特殊数组

```ts
type Data = [
  location: Location,
  timestamp: string
]
```

## 联合类型（Union Type）

描述一个类型是众多选项之一，例如已知字符串列表：

```ts
type Size = "small" | "medium" | "large"
```

## 交叉类型（Intersection Types）

一种 扩展/合并 的方法

```ts
type local = { x: number } & { y: number }
// local { x: number; y: number }
```

## 类型索引（Type Indexing）

提取一个类型的子集并重新命名

```ts
type Res = { data: { x: string } }
type Content = Res["data"]
// Conent {x:string}
```

## 类型来自值（Type from Value）

通过 `typeof` 运算符，重用存在 JavaScript 运行时值的类型

```ts
const data = { x: 123 }
type Content2 = typeof data
// Content2 = { x: number }
```

## 类型来自方法返回值（Type from Func Return）

重用方法的返回值类型

```ts
const createFixtures = () => 123
type Fixtures = ReturnType<typeof createFixtures>
// Fixtures = number
```
`ReturnType` 是 TypeScript 内置工具类型

## 类型来自模块（Type from Module）

```ts
// data.ts 文件
interface Data {
  (s: string): boolean
}
export {
  Data
}
```
```ts
// playground.ts 文件
const data: import("./data").Data = (n: string) => true
```



> 下面介绍的这些特性对于构建库、描述现有 JavaScript 代码非常有用，你可能会发现在大多数 TypeScript 应用程序中很少用到它们。

## 映射类型（Mapped Types）

行为类似于类型系统的 map 语句，允许使用输入类型更改产生新的类型结构。

```ts
type Artist = {name: string,bio: string}
type Subscriber<Type> = {
  // 循环遍历泛型参数 Type 的每一个字段
  [Property in keyof Type]:
  (nv: Type[Property]) => void
  //设置类型为一个函数，原始类型为参数
}
type ArtisSub = Subscriber<Artist>
// { name: (nv: string)=>void, bio: (nv:string)=>void }
```

## 条件类型（Conditional Types）

行为类似于类型系统的 if 语句，通过泛型创建，通常用于减少联合类型中的选项数量。

```ts
type HasFourLegs<Animal> = Animal extends { legs: 4 } ? Animal : never
type Bird = { cry: '唧唧喳喳', legs: 2 }
type Dog = { cry: '汪汪汪', legs: 4 }
type Ant = { cry: '...', legs: 6 }
type Wolf = { cry: '嗷呜~', legs: 4 }
type Animals = Bird | Dog | Ant | Wolf
type FourLegs = HasFourLegs<Animals>
// FourLegs = Dog | Wolf
```

## 模板联合类型（Template Union Types）

字符串模板可以用来组合和操纵类型系统中的文本

```ts
type SupportedLangs = "en" | "pt" | "zh";
type FooterLocaleIDs = "header" | "footer";

type AllLocaleIDs = `${SupportedLangs}_${FooterLocaleIDs}_id`;
// "en_header_id" | "en_footer_id" 
// | "pt_header_id" | "pt_footer_id" 
// | "zh_header_id" | "zh_footer_id"
```


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/static/TypeScript%20Types-ae199d69aeecf7d4a2704a528d0fd3f9.png>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>


