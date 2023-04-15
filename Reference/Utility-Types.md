# Utility Types

TypeScript 提供了几种工具类型方便常用类型转换。这些工具类型是全局可用的。

## Awaited\<Type>

这个类型模拟 `async` 函数里的 `await` 操作，或者是 `promise` 上的 `.then()` 方法-具体来说，就是递归打开 `Promise` 的方式。

内置实现：

```ts
type Awaited<T> =
    T extends null | undefined ? T : // 当没打开"--strictNullChecks" 模式时，处理 'null | undefined' 的特殊情况
        T extends object & { then(onfulfilled: infer F, ...args: infer _): any } ? // ' await ' 只打开有 ' then ' 方法的对象类型，非对象类型不打开
            F extends ((value: infer V, ...args: infer _) => any) ? // 如果 then 方法的第一个参数是函数，则提取该函数的第一个参数                
                Awaited<V> : // 递归打开第一个参数
                never : // 第一个参数不是方法
        T; // 非对象或没有 then 方法的对象
```

用法：

```ts
type A = Awaited<Promise<string>>; 
// type A = string
type B = Awaited<Promise<Promise<number>>>;
// type B = number
type C = Awaited<boolean | Promise<number>>;
// type C = number | boolean
```
## Partial\<Type>

返回一个与泛型参数类型 `T` 一样的类型，但所有属性都为`可选的`。

内置实现：

```ts
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```

用法：

```ts
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}
// 参数 fieldsToUpdate 类型为
//  fieldsToUpdate:{  
//     title?: string | undefined;  
//     description?: string | undefined;  
// }


const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};
 
const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});
```


## Required\<Type>

返回一个与泛型参数类型 `T` 一样的类型，但所有属性都为`必选的`，与 `Partial` 完全相反。

内置实现：

```ts
type Required<T> = {
    [P in keyof T]-?: T[P];
};

```

用法：

```ts
interface Props {
  a?: number;
  b?: string;
}
 
const obj: Props = { a: 5 };
 
const obj2: Required<Props> = { a: 5 };
Error：// Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
```
## Readonly\<Type>

返回一个与泛型参数类型 `T` 一样的类型，但所有属性都为`必选的`，意味着这个类型的属性不能再**重新赋值**。

内置实现：

```ts
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

用法：

```ts
interface Todo {
  title: string;
  data: number[];
}
 
const todo: Readonly<Todo> = {
  title: "Delete inactive users",
  data: []
};

todo.data.push(123) // OK
todo.data = [1]; 
Error：// Cannot assign to 'data' because it is a read-only property.
todo.title = "Hello";
Error：// Cannot assign to 'title' because it is a read-only property.
```

这个工具类型可用于表达，在运行时赋值表达式失败（例如：当尝试给一个 [冻结对象](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) 属性重新赋值时）

***Object.freeze***
```ts
function freeze<T extends Function>(f: T): T;
function freeze<T extends {[idx: string]: U | null | undefined | object}, U extends string | bigint | number | boolean | symbol>(o: T): Readonly<T>;
function freeze<Type>(obj: Type): Readonly<Type>;
```
## Record\<Keys, Type>

构造一个对象类型，其属性为泛型参数类型 `K` 的成员（约束为 `keyof any`），属性值为泛型参数类型 `T`。该工具可将一种类型的属性映射到另一种类型。

内置实现：

```ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

用法：

```ts
interface CatInfo {
  age: number;
  breed: string;
}
 
type CatName = "miffy" | "boris" | "mordred";
 
const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
cats.boris
// const cats: Record<CatName, CatInfo>
// (property) boris: CatInfo
```

## Pick\<Type, Keys>

返回一个与泛型参数类型 `T` 一样的类型，只包含 `K`（字符串字面量类型/字符串字面量联合类型）中的属性，注意 `K` 受到 `keyof T` 约束的，必须为 `T` 中的属性。

内置实现：

```ts
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

用法：

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}
 
type TodoPreview = Pick<Todo, "title" | "completed">;
// type TodoPreview = { title: string;  completed: boolean; }

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
```
## Omit\<Type, Keys>

返回一个与泛型参数类型 `T` 一样的类型，但是不包含 `K`（字符串字面量类型/字符串字面量联合类型）中的属性，与 `Pick` 完全相反。

内置实现：

```ts
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

用法：

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}

type TodoPreview = Omit<Todo, "description">;
// type TodoPreview = { title: string; completed: boolean; createdAt: number; }
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
  createdAt: 1615544252770,
};


type TodoInfo = Omit<Todo, "completed" | "createdAt">;
// type TodoInfo = { description: string; title: string; }
const todoInfo: TodoInfo = {
  title: "Pick up kids",
  description: "Kindergarten closes at 5pm",
};

```
## Exclude\<UnionType, ExcludedMembers>

构造一个类型，排除泛型参数类型 `T` 能分配给 `U` 的所有联合成员。

内置实现：

```ts
type Exclude<T, U> = T extends U ? never : T;
```

用法：

```ts
type T0 = Exclude<"a" | "b" | "c", "a">;
// type T0 = "b" | "c"

type T1 = Exclude<"a" | "b" | "c", "a" | "b">;
// type T1 = "c"

type T2 = Exclude<string | number | (() => void), Function>;
// type T2 = string | number
```
## Extract\<Type, Union>

构造一个类型，提取出泛型参数类型 `T` 能分配给 `U` 的所有联合成员，和 `Exclude` 完全相反。

内置实现：

```ts
type Extract<T, U> = T extends U ? T : never;
```

用法：

```ts
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
// type T0 = "a"

type T1 = Extract<string | number | (() => void), Function>;
// type T1 = () => void
```
## NonNullable\<Type>

构造一个类型，排除泛型参数类型 `T` 的 `null` 和 `undefined`。

内置实现：

```ts
type NonNullable<T> = T & {};
```

用法：

```ts
type T0 = NonNullable<string | number | undefined>;
// type T0 = string | number

type T1 = NonNullable<string[] | null | undefined>;
// type T1 = string[]
```
## Parameters\<Type>

构造一个元组类型或数组类型。泛型参数类型` T` ，如果为函数时会生成该函数参数列表的元组类型。如果为 `any` 会生成数组类型 `unknown[]`。（如果 `T` 不是函数，则生成 `never` 类型，并报错）。

内置实现：

```ts
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
```

用法：

```ts
declare function f1(arg: { a: number; b: string }): void;
 
type T0 = Parameters<() => string>;
//type T0 = []

type T1 = Parameters<(s: string) => void>;
// type T1 = [s: string]

type T2 = Parameters<<T>(arg: T) => T>;
// type T2 = [arg: unknown]

type T3 = Parameters<typeof f1>;
// type T3 = [arg: {
//     a: number;
//     b: string;
// }]

type T4 = Parameters<any>;
// type T4 = unknown[]

type T5 = Parameters<never>;
// type T5 = never

type T6 = Parameters<unknown>;
// type T6 = never
Error：// Type 'unknown' does not satisfy the constraint '(...args: any) => any'.

type T7 = Parameters<string>;
// type T7 = never
Error：// Type 'string' does not satisfy the constraint '(...args: any) => any'.

type T8 = Parameters<Function>;
// type T8 = never
Error：
// Type 'Function' does not satisfy the constraint '(...args: any) => any'.
// Type 'Function' provides no match for the signature '(...args: any): any'.
```
## ConstructorParameters\<Type>

构造一个元组或数组类型。泛型参数类型` T` ，如果为构造函数时会生成该构造函数参数列表的元组类型，如果为 `any` 会生成数组类型 `unknown[]`。（如果 `T` 不是函数，则生成 `never` 类型，并报错）。

内置实现：

```ts
type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never;
```

用法：

```ts
type T0 = ConstructorParameters<ErrorConstructor>;  
// type T0 = [message?: string]

type T1 = ConstructorParameters<FunctionConstructor>;     
// type T1 = string[]

type T2 = ConstructorParameters<RegExpConstructor>;
//type T2 = [pattern: string | RegExp, flags?: string]

type T3 = ConstructorParameters<any>;
// type T3 = unknown[]

type T4 = ConstructorParameters<never>;
// type T4 = never

type T5 = ConstructorParameters<unknown>;
// type T5 = never
Error：// Type 'unknown' does not satisfy the constraint 'abstract new (...args: any) => any'.

type T6 = ConstructorParameters<string>;
// type T6 = never
Error：// Type 'string' does not satisfy the constraint 'abstract new (...args: any) => any'.

type T7 = ConstructorParameters<Function>;
// type T7 = never
Error：
// Type 'Function' does not satisfy the constraint 'abstract new (...args: any) => any'.
// Type 'Function' provides no match for the signature 'new (...args: any): any'.
```
## ReturnType\<Type>

构造一个类型，由函数  `T` 的返回类型组成。

内置实现：

```ts
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

用法：

```ts
declare function f1(): { a: number; b: string };
 
type T0 = ReturnType<() => string>;
// type T0 = string

type T1 = ReturnType<(s: string) => void>;
// type T1 = void

type T2 = ReturnType<<T>() => T>;
// type T2 = unknown

type T3 = ReturnType<<T extends U, U extends number[]>() => T>;
// type T3 = number[]

type T4 = ReturnType<typeof f1>;
// type T4 = {
//     a: number;
//     b: string;
// }

type T5 = ReturnType<any>;
// type T5 = any

type T6 = ReturnType<never>;
// type T6 = never

type T7 = ReturnType<unknown>;
// type T7 = any
Error：
// Type 'unknown' does not satisfy the constraint '(...args: any) => any'

type T8 = ReturnType<string>;
// type T8 = any
Error: // Type 'string' does not satisfy the constraint '(...args: any) => any'.

type T9 = ReturnType<Function>;
// type T9 = any
Error：
// Type 'Function' does not satisfy the constraint '(...args: any) => any'.
// Type 'Function' provides no match for the signature '(...args: any): any'.
```
## InstanceType\<Type>

构造一个类型，由构造函数 `T` 的返回类型组成。

内置实现：

```ts
type InstanceType<T extends abstract new (...args: any) => any> = T extends abstract new (...args: any) => infer R ? R : any;
```

用法：
```ts
class C {
  x = 0;
  y = 0;
}

type T0 = InstanceType<typeof C>;
// type T0 = C

type T1 = InstanceType<any>;
// type T1 = any

type T2 = InstanceType<never>;
// type T2 = never

type T3 = InstanceType<unknown>;
// type T3 = any
Error：
// Type 'unknown' does not satisfy the constraint 'abstract new (...args: any) => any'.

type T4 = InstanceType<string>;
// type T4 = any
Error：
// Type 'string' does not satisfy the constraint 'abstract new (...args: any) => any'.

type T5 = InstanceType<Function>;
// type T5 = any
Error：
// Type 'Function' does not satisfy the constraint 'abstract new (...args: any) => any'.
// Type 'Function' provides no match for the signature 'new (...args: any): any'.
```
## ThisParameterType\<Type>

提取函数 `T` 的 [this](https://www.typescriptlang.org/docs/handbook/functions.html#this-parameters) 参数，如果函数没有 `this` 参数，则为  [unknow](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type) 类型。

内置实现：

```ts
type ThisParameterType<T> = T extends (this: infer U, ...args: never) => any ? U : unknown;
```

用法：

```ts
function toHex(this: Number) {
  return this.toString(16);
}
 
function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}
```
## OmitThisParameter\<Type>

1. 移除函数 `T` 的 [this](https://www.typescriptlang.org/docs/handbook/functions.html#this-parameters) 参数，根据  `T` 创建一个没有 `this` 参数的新函数类型。
2. 移除 `this` 参数的同时，如果函数有泛型参数也会被擦除。
3. 多个函数重载只有最后一个重载签名参数有 `this`，那么只有它传播进 `OmitThisParameter`。
4. 如果 `T` 没有显式声明 `this` 参数，直接返回 `T`。


内置实现：

```ts
type OmitThisParameter<T> = unknown extends ThisParameterType<T> ? T : T extends (...args: infer A) => infer R ? (...args: A) => R : T;
```

用法：

```ts
function toHex(this: Number) {
  return this.toString(16);
}
type T0 = OmitThisParameter<typeof toHex>
// type T0 = () => string


function identity<T>(this: Number, value: T): T {
  return value;
}
type T1 = OmitThisParameter<typeof identity>
// type T1 = (value: unknown) => unknown


function identity2(value: number): string
function identity2(this: Number, value: string): number
function identity2(value: number | string) {
  if (typeof value === "number") {
    return value + ''
  }
  return isNaN(+value) ? 0 : +value;
}
type T2 = OmitThisParameter<typeof identity2>
// type T2 = (value: string) => number


function identity3(value: number): string
function identity3(value: string): number
function identity3(value: number | string) {
  if (typeof value === "number") {
    return value + ''
  }
  return isNaN(+value) ? 0 : +value;
}
type T3 = OmitThisParameter<typeof identity3>
// type T3 = {
//    (value: number): string;
//    (value: string): number;
// }
```
## ThisType\<Type>

这个工具没有返回转换后的类型。反而，它作为上下文 [this](https://www.typescriptlang.org/docs/handbook/functions.html#this) 类型的标记。请注意，必须启用 [noImplicitThis](https://www.typescriptlang.org/tsconfig#noImplicitThis) 标志才能使用此工具类型。

内置实现：

```ts
interface ThisType<T> { }
```

用法：

```ts
type ObjectDescriptor<D, M> = {
  data?: D;
  methods?: M & ThisType<D & M>; // Type of 'this' in methods is D & M
};
 
function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
  let data: object = desc.data || {};
  let methods: object = desc.methods || {};
  return { ...data, ...methods } as D & M;
}
 
let obj = makeObject({
  data: { x: 0, y: 0 },
  methods: {
    moveBy(dx: number, dy: number) {
      this.x += dx; // Strongly typed this
      this.y += dy; // Strongly typed this
    },
  },
});
 
obj.x = 10;
obj.y = 20;
obj.moveBy(5, 5);
```

在上面的例子中，`makeObject` 参数中的 `methods` 对象具有上下文类型，它包括`ThisType<D & M>`，因此，`methods` 对象中的方法上下文 [this](https://www.typescriptlang.org/docs/handbook/functions.html#this) 类型是 `{ x: number, y: number } & { moveBy(dx: number, dy: number): number }` 。注意，`methods` 属性的类型同时是一个推理目标，以及方法中 `this` 类型的来源。

`ThisType<T>` 标记接口只是在 `lib.d.ts` 中声明的一个空接口。除了在对象字面量的上下文类型中被识别之外，该接口的行为类似于空接口。
## Intrinsic String Manipulation Types

`Uppercase<StringType>`

`Lowercase<StringType>`

`Capitalize<StringType>`

`Uncapitalize<StringType>`

为了帮助对模板字符串字面量进行字符串操作，TypeScript 包含一组类型，可用于类型系统中的字符串操作。可参考 [Template Literal Types](https://juejin.cn/post/7217267332656693308) 。


感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/utility-types.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
