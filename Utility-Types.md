# Utility Types
TypeScript provides several utility types to facilitate common type transformations.These utilities are available globally.

TypeScript 提供了几种实用类型来促进常用类型转换。这些实用类型是全局可用的。

## <a href="#Awaited">Awaited\<Type></a>
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
## <a href="#Partial">Partial\<Type></a>
Constructs a type with all properties of type set to optional.This utility will return a type that represents all subsets of a given type.

构造一个 `Type` 的所有属性都设置为可选的类型，这个功能将返回一个表示给定类型的所有子集的类型。

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

## <a href="#Required">Required\<Type></a>
Constructs a type consisting of all properties of `Type` set to required.The opposite of <a href="#Partial">Partial</a>.

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
