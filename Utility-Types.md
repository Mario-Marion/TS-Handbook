# Utility Types
TypeScript provides several utility types to facilitate common type transformations.These utilities are available globally.

TypeScript 提供了几种实用类型来促进常用类型转换。这些实用类型是全局可用的。

## Awaited\<Type>
This type is meant to model operations like `await` in `async` function,or the `.then()` method on `Promise`s-specifically,the way that they recursively unwrap `Promise`s.

这类型像模拟 `async` 函数里的 `await` 操作，或者是 `promise` 上的 `.then()` 方法-具体来说，就是它们递归展开 Promise 的方式。

```ts
type A = Awaited<Promise<string>>; 
// type A = string
type B = Awaited<Promise<Promise<number>>>;
// type B = number
type C = Awaited<boolean | Promise<number>>;
// type C = number | boolean
```
## Partial\<Type>
Constructs a type with all properties of `Type` set to optional.This utility will return a type that represents all subsets of a given type.

构造一个 `Type` 的所有属性都设置为可选的类型，这个工具类型将返回一个表示给定类型的所有子集的类型。

```ts
interface Todo {
  title: string;
  description: string;
}
 
function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}
 
const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};
 
const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});
```

## Required\<Type>
Constructs a type consisting of all properties of `Type` set to required.The opposite of <a href="#partialtype">Partial</a>.

构造一个 `Type` 的所有属性都设置为必须的，组成的类型，与 `Partial` 相反。
```ts
interface Props {
  a?: number;
  b?: string;
}
 
const obj: Props = { a: 5 };
 
const obj2: Required<Props> = { a: 5 };
// Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
```
## Readonly\<Type>
Constructs a type with all properties of `Type` set to `readonly`,meaning the properties of constructed type cannot be reassigned.

构造一个`Type` 的所有属性都设置为 `readonly` 的类型，意味着这个类型的属性不能再重新分配。

```ts
interface Todo {
  title: string;
}
 
const todo: Readonly<Todo> = {
  title: "Delete inactive users",
};
 
todo.title = "Hello";
// Cannot assign to 'title' because it is a read-only property.
```
This utility is useful for representing assignment expressions that will fail at runtime(i.e.when attempting to reassign properties of a [fronzen object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)).

这个工具类型表现在运行时，赋值表达式报错的时候是有用的（当尝试给一个冻结对象属性重新赋值时）

Object.freeze
```ts
function freeze<Type>(obj: Type): Readonly<Type>;
```
## Record\<Keys, Type>
Constructs an object type whose property keys are `keys` and whose property values are `Type`.This utility can be used to map the properties of a type to another type.

构造一个对象类型，其属性的keys为 `Keys` ，其属性值为 `Type`。该工具可用于将一种类型的属性映射到另一种类型。
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
cat.boris
// const cats: Record<CatName, CatInfo>
// (property) boris: CatInfo
```
## Pick\<Type, Keys>
Constructs a type by picking the set of properties `Keys`(string literal or union of string literals) from `Type`.

构造一个类型，设置其属性为 `Type` 挑选出的 `Keys`(字符串字面量或字符串字面量联合)的属性
```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}
 
type TodoPreview = Pick<Todo, "title" | "completed">;
 
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
 
// TodoPreview = {title: string;completed: boolean;}
```
## Omit\<Type, Keys>
Constructs a type by picking all properties from `Type` and then removing `Keys`(string literal or union of string literals).

构造一个类型，挑选出 `Type` 的所有属性和移除 `Keys`(字符串字面量或字符串字面量联合)的属性

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}
 
type TodoPreview = Omit<Todo, "description">;
 
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
  createdAt: 1615544252770,
};

// TodoPreview: {title: string;completed: boolean;createdAt: number;}
 
type TodoInfo = Omit<Todo, "completed" | "createdAt">;
 
const todoInfo: TodoInfo = {
  title: "Pick up kids",
  description: "Kindergarten closes at 5pm",
};
// TodoInfo: {title: string;description: string;}
```
## Exclude\<UnionType, ExcludedMembers>

Constructs a type by excluding from `UnionType` all union members that are assignable to `ExcludedMembers`.

构造一个类型，排除 `UnionType` 能分配给 `ExcludedMembers` 的所有联合成员

```ts
type T0 = Exclude<"a" | "b" | "c", "a">;
// type T0 = "b" | "c"

type T1 = Exclude<"a" | "b" | "c", "a" | "b">;
// type T1 = "c"

type T2 = Exclude<string | number | (() => void), Function>;
// type T2 = string | number
```
## Extract\<Type, Union>
Constructs a type by extracting from `Type` all union members that are assignable to `Union`.

构造一个类型，提取出 `Type` 能分配给 `Union` 的所有联合成员

```ts
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
// type T0 = "a"

type T1 = Extract<string | number | (() => void), Function>;
// type T1 = () => void
```
## NonNullable\<Type>
Constructs a type by excluding `null` and `undefined` from `Type`.

构造一个类型，排除 `Type` 的 `null` 和 `undefined`。

```ts
type T0 = NonNullable<string | number | undefined>;
// type T0 = string | number

type T1 = NonNullable<string[] | null | undefined>;
// type T1 = string[]
```
## Parameters\<Type>
Constructs a tuple type from the types used in the parameters of a function type `Type`.

构造一个元组类型，来自函数类型 `Type` 使用的参数。
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

type T6 = Parameters<string>;
// Type 'string' does not satisfy the constraint '(...args: any) => any'.
// type T6 = never

type T7 = Parameters<Function>;
// Type 'Function' does not satisfy the constraint '(...args: any) => any'.
// Type 'Function' provides no match for the signature '(...args: any): any'.
// type T7 = never
```
## ConstructorParameters\<Type>
Constructs a tuple or array type from the types of a constructor function type.It produces a tuple type with all the parameter types(or the `never` if `Type` is not a function).

构造一个元组或数组类型，来自构造函数类型。它生成一个包含所有参数类型的元组类型（如果 Type 不是函数，则生成 never 类型）。

```ts
type T0 = ConstructorParameters<ErrorConstructor>;  
// type T0 = [message?: string]

type T1 = ConstructorParameters<FunctionConstructor>;     
// type T1 = string[]

type T2 = ConstructorParameters<RegExpConstructor>;
//type T2 = [pattern: string | RegExp, flags?: string]

type T3 = ConstructorParameters<any>;
// type T3 = unknown[]
 
type T4 = ConstructorParameters<Function>;
// Type 'Function' does not satisfy the constraint 'abstract new (...args: any) => any'.
// Type 'Function' provides no match for the signature 'new (...args: any): any'.
// type T4 = never
```
## ReturnType\<Type>
Constructs a type consisting of the return type of function `Type`.

构造一个类型，由函数  `Type` 的返回类型组成。
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

type T7 = ReturnType<string>;
// Type 'string' does not satisfy the constraint '(...args: any) => any'.
// type T7 = any

type T8 = ReturnType<Function>;
// Type 'Function' does not satisfy the constraint '(...args: any) => any'.
// Type 'Function' provides no match for the signature '(...args: any): any'.
// type T8 = any
```
## InstanceType\<Type>
Constructs a type consisting of the instance type of a constructor function in `Type`.

构造一个类型，由 `Type` 中构造函数的实例类型组成。
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

type T3 = InstanceType<string>;
// Type 'string' does not satisfy the constraint 'abstract new (...args: any) => any'.
type T3 = any

type T4 = InstanceType<Function>;
// Type 'Function' does not satisfy the constraint 'abstract new (...args: any) => any'.
// Type 'Function' provides no match for the signature 'new (...args: any): any'.
// type T4 = any
```
## ThisParameterType\<Type>
Extracts the type of [this](https://www.typescriptlang.org/docs/handbook/functions.html#this-parameters) parameter for a function type,or [unknow](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type) if the function type has no this parameter.

提取函数 `Type` 的 this 参数，如果函数没有 this 参数，为 unknow。

```ts
function toHex(this: Number) {
  return this.toString(16);
}
 
function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}
```
## OmitThisParameter\<Type>
Removes the [this](https://www.typescriptlang.org/docs/handbook/functions.html#this-parameters) parameter from `Type`.if `Type` has no explicitly declared `This` parameter,the result is simply `Type`.Otherwise,a new function type with no `this` parameter is created from `Type`.Generics are erased and only the last overload signature is propagated into the new function type.

移除 `Type` 的 this 参数，如果 `Type` 没有显式声明 `this` 参数，结果仅为 `Type`。否则，根据  `Type` 创建一个没有 `this` 参数的新函数类型。泛型被擦除，只有最后一个重载签名参数有 `this` 才传播进新函数类型。(不是最后一个重载函数签名，返回 `Type`)
(`Type`不是函数，仅返回`Type`)
```ts
function toHex(this: Number) {
  return this.toString(16);
}
 
const fiveToHex: OmitThisParameter<typeof toHex> = toHex.bind(5);
 
console.log(fiveToHex());
```
## ThisType\<Type>
This utility does not return a transformed type.Instead,it serves as a marker for a contextual [this](https://www.typescriptlang.org/docs/handbook/functions.html#this) type.Note that the noImplicitThis flag must be enabled to use this utility.

这个工具没有返回转换后的类型。反而，它作为上下文 this 类型的标记。请注意，必须启用 `noImplicitThis` 标志才能使用此实用程序。
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
n the example above, the `methods` object in the argument to `makeObject` has a contextual type that includes `ThisType<D & M>` and therefore the type of [this](https://www.typescriptlang.org/docs/handbook/functions.html#this) in methods within the `methods` object is `{ x: number, y: number } & { moveBy(dx: number, dy: number): number }`. Notice how the type of the `methods` property simultaneously is an inference target and a source for the `this` type in methods.

The `ThisType<T>` marker interface is simply an empty interface declared in `lib.d.ts`. Beyond being recognized in the contextual type of an object literal, the interface acts like any empty interface.

在上面的例子中，`makeObject` 参数中的 `methods` 对象具有上下文类型，其中包括`ThisType<D & M>`，因此，方法对象中方法中的 this 类型是 `{ x: number, y: number } & { moveBy(dx: number, dy: number): number }` ，注意，方法属性的类型同时是一个推理目标，以及方法中 this 类型的来源。  
`ThisType<T>` 标记接口只是在 `lib.d.ts` 中声明的一个空接口。除了在对象字面量的上下文类型中被识别之外，该接口的行为类似于任何空接口。
## Intrinsic String Manipulation Types

`Uppercase<StringType>`

`Lowercase<StringType>`

`Capitalize<StringType>`

`Uncapitalize<StringType>`

To help with string manipulation around template string literals, TypeScript includes a set of types which can be used in string manipulation within the type system. You can find those in the [Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html#uppercasestringtype) documentation.

为了帮助对模板字符串字面量进行字符串操作，TypeScript 包含一组类型，可用于类型系统中的字符串操作。你可以在 Template Literal Types 文档中找到它们。