---
highlight: vs2015
---
# Conditional Types

在大多数程序的核心，我们必须根据输入做出决定。JavaScript 程序也不例外，但考虑到值实际上可以很容易的进行自检，而且也是基于输入的类型据决定的。所以条件类型有助于描述输入和输出类型之间的关系。

```ts
interface Animal {
    live(): void;
}
interface Dog extends Animal {
    woof(): void;
}

type Example1 = Dog extends Animal ? number : string;
// Example1类型：type Example1 = number
type Example2 = RegExp extends Animal ? number : string;
// Example2 类型：type Example2 = string
```
条件类型看起来有点像 JavaScript 中的条件表达式（`condition ? trueExpression : falseExpression`）
```ts
SomeType extends OtherType ? TrueType : FalseType;
```
当 `extends` 的左边的类型可以赋值给右边的类型时，那么你将得到第一个分支中的类型（"true"分支）；否则，将得到后面的分支中的类型（"false"分支）。

从上面的例子来看，条件类型可能不会立即变得有用——可以告诉我们 `Dog extends Animal` 是否为真，并选择 `number` 或 `string`！但是条件类型的强大之处在于与泛型一起使用。

例如，下面的 `createLabel` 函数：
```ts
interface IdLabel {
    id: number /* some fields */;
}
interface NameLabel {
    name: string /* other fields */;
}

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
    throw "unimplemented";
}
```
这些 `createLabel` 的重载描述了一个，根据输入类型做出选择的 JavaScript 函数。注意以下几点：
1. 如果一个库必须在其 API 中一遍又一遍地做出相同类型的选择，这就变得很麻烦。
2. 我们创建了三个重载：根据输入确定类型时，然后选择对应重载，每个情况一个重载（一个用于 `string`，一个用于 `number`），和一个最常用情况（使用 `string | number`）。`createLabel` 每处理一种新类型，那么重载的数量会呈指数性增长

然而，我们可以将该逻辑编码为条件类型：
```ts
type NameOrId<T extends number | string> = T extends number
? IdLabel
: NameLabel;
```
使用条件类型将重载简化为一个没有重载的函数。
```ts
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
    throw "unimplemented";
}
let a = createLabel("typescript");
// a 类型：let a: NameLabel
let b = createLabel(2.8);
// b 类型：let b: IdLabel
let c = createLabel(Math.random() ? "hello" : 42);
// c 类型：let c: NameLabel | IdLabel
```
## 条件类型约束
通常，条件类型中的检查将为我们提供一些新信息。就像使用类型守卫进行缩窄一样，可以为我们提供更具体的类型，条件类型的 "true" 分支，将通过检查的类型进一步约束泛型。

例如，以下为例：
```ts
type MessageOf<T> = T["message"];
// Error："message" 不能用于索引类型 "T"。
```
在这个例子中，TypeScript 会报错，因为不知道 `T` 是否有一个名为 `message` 的属性。我们可以约束 `T`, 并且 TypeScript 将不再报错：
```ts
type MessageOf<T extends { message: unknown }> = T["message"];

interface Email {
    message: string;
}

type EmailMessageContents = MessageOf<Email>;
//  EmailMessageContents 类型：type EmailMessageContents = string
```
但是，如果我们希望 `MessageOf` 采用任意类型，并在 `message` 属性不可用时默认为 `never`。我们可以移除约束并引入条件类型来实现：
```ts
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;
interface Email {
    message: string;
}
interface Dog {
    bark(): void;
}

type EmailMessageContents = MessageOf<Email>;
// EmailMessageContents 类型：type EmailMessageContents = string
type DogMessageContents = MessageOf<Dog>;
// DogMessageContents 类型：type DogMessageContents = never
```
在 "true" 分支中，TypeScript 将知道 `T` 有一个 `message` 属性。

另一个例子，我们编写一个名为 `Flatten` 的类型，如果是数组类型，将返回它们的元素类型，不是的话返回自身：
```ts
type Flatten<T> = T extends any[] ? T[number] : T;

// 提取出元素类型.
type Str = Flatten<string[]>;
// str 类型：type Str = string

// 保持类型不变。
type Num = Flatten<number>;
// Num 类型：type Num = number
```
当 `Flatten` 被赋予一个数组类型时，使用 `number` 的索引访问类型来获取 `string[]` 的元素类型。否则，返回给定的类型。(可参考 [索引访问类型](https://juejin.cn/post/7213934749093445691))

## 条件类型中推断
我们发现自己只是使用条件类型去应用约束，然后提取出类型。这是一种非常常见的操作，条件类型使它变得更容易。

条件类型为我们提供了一种方法，使用 `infer` 关键字。下面例子中，使用 `infer` 关键字为 "true" 分支中的元素类型进行推断，而不是 "手动" 使用索引访问类型获取元素类型：
```ts
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```
这里，我们使用 `infer` 关键字以声明的方式引入了一个名为 `Item` 的新泛型类型变量，而不是指定如何在 "true" 分支中检索 `T` 的元素类型。这使我们不必考虑如何挖掘和探索类型的结构。

我们可以使用 `infer` 关键字，编写一些有用的辅助类型别名。例如以下简单的情况，我们可以提取出函数类型的返回类型：
```ts
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
? Return
: never;
type Num = GetReturnType<() => number>;
// Num 类型：type Num = number
type Str = GetReturnType<(x: string) => string>;
// Str 类型：type Str = string
type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>;
// Bools 类型：type Bools = boolean[]
```
当从具有多个调用签名的类型（例如重载函数的类型）进行推断时，将使用最后一个签名进行推断（这大概是最宽松的全面捕获情况）。不可能根据参数类型列表执行重载解析。
```ts
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
? Return
: never;

declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;
type T1 = GetReturnType<typeof stringOrNum>;
// T1 类型：type T1 = string | number
```

## 分配条件类型
当条件类型作用于泛型类型时，并给定联合类型，将会变得具有分配性。例如：
```ts
type ToArray<Type> = Type extends any ? Type[] : never;
```
如果我们将一个联合类型传递给 `ToArray`，那么条件类型将应用于该联合的每个成员。
```ts
type ToArray<Type> = Type extends any ? Type[] : never;
type StrArrOrNumArr = ToArray<string | number>;
// StrArrOrNumArr 类型：type StrArrOrNumArr = string[] | number[]
```
将联合类型的每个成员类型映射为：
```ts
ToArray<string> | ToArray<number>;
```
最后为：
```ts
string[] | number[];
```
通常，分配性是理想的行为。但也可以避免这种行为，可以将 `extends` 关键字的两边用方括号括起来。
```ts
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

// 'StrArrOrNumArr' 不再是联合类型
type StrArrOrNumArr = ToArrayNonDist<string | number>;
// StrArrOrNumArr 类型：type StrArrOrNumArr = (string | number)[]
```




> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/2/conditional-types.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

