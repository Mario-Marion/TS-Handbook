---
highlight: vs2015
---
# Indexed Access Types
我们可以使用索引访问类型去查找另一个类型上的特定属性值类型：
```ts
type Person = { age: number; name: string; alive: boolean };

type Age = Person["age"];
// Age 类型：type Age = number
```
索引类型本身就是一种类型，所以我们索引类型可以使用联合类型，`keyof` 操作符，或其他类型：
```ts
type Person = { age: number; name: string; alive: boolean };

type I1 = Person["age" | "name"];
// I1 类型：type I1 = string | number

type I2 = Person[keyof Person];
// I2 类型：type I2 = string | number | boolean

type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];
// I3 类型：type I3 = string | boolean
```
如果你试图索引一个不存在的属性，你甚至会看到一个错误:
```ts
type Person = { age: number; name: string; alive: boolean };
type I1 = Person["alve"];
// Error 'alve' 属性不存在 'Person' 类型上
```
不是索引签名定义的对象类型（查阅：[对象类型的索引签名](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures)），不能使用 `string` 索引类型去获取属性值类型。例子：
```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person[string];
// Error：'Person' 类型没有匹配 'string' 类型的索引签名

// Good
type Person2 = {
    [i: string]: number
}
type Age2 = Person2[string];
// Age2 类型：type Age2 = number
```
数组可以使用 `number` 索引类型去获取数组元素的类型，然后与 `typeof` 结合使用，可以方便地捕获数组的元素类型：
```ts
const MyArray = [
    { name: "Alice", age: 15 },
    { name: "Bob", age: 23 },
    { name: "Eve", age: 38 },
];
type Person = typeof MyArray[number];
// Person 类型：type Person = { name: string; age: number; }

type Age2 = Person["age"];
// Age2 类型：type Age2 = number

type Age = typeof MyArray[number]["age"];
// Age 类型：type Age = number
```
> 注意，这里的 `MyArray` 变量是值，所以要用 `typeof` 先推断出类型，再使用索引类型获取数组元素类型


索引类型只能使用类型，不能使用值：
```ts
type Person = { age: number; name: string; alive: boolean };

const key = "age";
type Age = Person[key];
// Error：
// 'key' 类型不能用作索引类型
// 'key' 是一个值，但在这里被用作类型。你是想用 'typeof key' 吗？

// Good
const key = "age";
type Age = Person[typeof key];
// Age 类型：type Age = number
// ------or------
type key = "age";
type Age = Person[key];
// Age 类型：type Age = number
```



感谢观看，如有错误，望指正

>官网地址： <https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html>
>
>github 资料： <https://github.com/Mario-Marion/TS-Handbook>

