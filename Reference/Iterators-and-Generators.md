## 迭代器

一个对象如果实现了 [Symbol.iterator](https://www.typescriptlang.org/docs/handbook/symbols.html#symboliterator) 属性，则认为它是可迭代的。称为可迭代对象。一些内置类型，如：`Array`, `Map`, `Set`, `String`, `Int32Array`, `Uint32Array`，等。这些都实现了 `Symbol.iterator` 属性。对象上的 `Symbol.iterator` 函数负责返回要迭代的值列表。
### `Iterable` 接口

如果我们想接收可迭代类型，则可以使用 `Iterable` 类型：

```ts
function toArray<X>(xs: Iterable<X>): X[] {
    return [...xs]
}
```

### `for...of` 语法

`for...of` 在可迭代对象上循环，调用对象上的 `Symbol.iterator` 属性。例子：

```ts
let someArray = [1, "string", false];

for (let entry of someArray) {
    console.log(entry); // 1, "string", false
}
```

### `for...of` VS. `for...in` 语法

`for...of` 和 `for...in` 都可以遍历列表；然而迭代的值是不同的，`for...in` 返回被迭代对象的的键列表，而 `for..of` 返回被迭代对象的数值属性的值列表。

例子：

```ts
let list = [4, 5, 6];

for (let i in list) {
    console.log(i); // "0", "1", "2",
}

for (let i of list) {
    console.log(i); // 4, 5, 6
}
```

另一个不同是，`for...in` 可操作任意对象；它的作用是检查对象的属性。`for...of` 只遍历可迭代对象的值。内置对象如： `Map` 和 `Set` 实现了 `Symbol.iterator` 属性，允许访问存储的值。

```ts
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
    console.log(pet); // "species"
}

for (let pet of pets) {
    console.log(pet); // "Cat", "Dog", "Hamster"
}
```

## 代码生成
### 针对 ES5 和 ES3

当 ES5 或 ES3 兼容引擎为目标时，迭代器只允许用于 `Array` 类型的值。对非 `Array` 的值使用 `for...of` 循环是错误的，即使这些非数组值实现了 `Symbol.iterator` 属性。

编译器会为 `for...of` 循环生成一个简单的 `for` 循环，例如：

```ts
let numbers = [1, 2, 3];

for (let num of numbers) {
    console.log(num);
}
```

将生成为：

```ts
var numbers = [1, 2, 3];

for (var _i = 0; _i < numbers.length; _i++) {
    var num = numbers[_i];
    console.log(num);
}
```

### 针对 ECMAScript 2015 和 更高版本

当符合 ECMAScipt 2015 的引擎为目标时，编译器将生成 `for..of` 循环，以引擎中的内置迭代器实现为目标。


感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/iterators-and-generators.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>