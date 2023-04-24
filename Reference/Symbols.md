# Symbols

从 ECMAScript 2015 开始，`symbol` 是一种基本数据类型，就像 `number` 和 `string` 一样。

`symbol` 值通过调用 `Symbol` 构造函数创建。

```ts
let sym1 = Symbol();

let sym2 = Symbol("key"); // optional string key
```

Symbol 是不可变的，并且唯一的。

```ts
let sym2 = Symbol("key");

let sym3 = Symbol("key");

sym2 === sym3; // false, symbols are unique
```

就像字符串一样，symbol 可以作为对象的属性

```ts
const sym = Symbol();

let obj = {
    [sym]: "value",
};

console.log(obj[sym]); // "value"
```

symbol 也可以与计算属性声明结合使用，去声明对象的属性和类成员。

```ts
const getClassNameSymbol = Symbol();

class C {
    [getClassNameSymbol]() {
        return "C";
    }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

## `unique symbol`

为了能够将 symbol 被视为唯一的字面量类型，可以使用特殊类型 `unique symbol`。`unique symbol` 是 `symbol` 的子类型，并且只能通过调用 `Symbol()` 或 `Symbol.for()` 产生，或使用显式的类型注释。这个类型只能在 `const` 声明和 `readonly static` 属性上使用，并且引用特定的 唯一标识（unique symbol），需要使用 `typeof` 操作符。每个对 唯一标识 的引用，都意味着把给定声明绑定完全唯一的标识。

```ts
const sym1: unique symbol = Symbol();

// sym2 can only be a constant reference.
let sym2: unique symbol = Symbol();
Error：// A variable whose type is a 'unique symbol' type must be 'const'.

// Works - refers to a unique symbol, but its identity is tied to 'sym1'.
let sym3: typeof sym1 = sym1;

// Also works.
class C {
  static readonly StaticSymbol: unique symbol = Symbol();
}
```

因为每个 `unique symbol` 是每个单独的个体，所以两个 `unique symbol` 类型不能互相赋值或比较。

```ts
const sym2 = Symbol();
const sym3 = Symbol();

if (sym2 === sym3) {
  Error：// This comparison appears to be unintentional because the types 'typeof sym2' and 'typeof sym3' have no overlap.
  // ...
}
```

## 众所周知的 Symbols

除了用户定义的 symbols 外，还有一些众所周知的内置 symbols。内置 symbols 用于表示内部语言行为。

以下是一些众所周知的 Symbols：

### [`Symbol.hasInstance`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/hasInstance)

一个方法，返回值会被转为布尔值。用于确定对象是否为某构造函数的实例。通过 `instanceof` 操作符调用。

### [`Symbol.isConcatSpreadable`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/isConcatSpreadable)

一个布尔值，决定对象作为 Array.prototype.concat 的参数，是否应该展开其数组元素。

### [`Symbol.iterator`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator)

一个方法，返回对象的默认迭代器。通过 `for...of` 语句调用。

### [`Symbol.match`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/match)

一个正则表达式方法，将正则表达式与字符串匹配。通过 `String.prototype.match` 方法调用。

### [`Symbol.replace`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/replace)

一个正则表达式方法，替换字符串中匹配的子字符串。通过 `String.prototype.replace` 方法调用。

### [`Symbol.search`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/search)

一个正则表达式方法，返回字符串中匹配的子字符串索引。通过 `String.prototype.search` 方法调用。

### `Symbol.species`

一个函数值属性，其被构造函数用以创建派生对象。

### [`Symbol.split`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/split)

一个正则表达式方法，在匹配字符串的索引处拆分字符串。通过 `String.prototype.split` 方法调用。

### [`Symbol.toPrimitive`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toPrimitive)

一个方法，将对象转换为相应原始值的方法。通过 `ToPrimitive` 抽象操作调用。

### [`Symbol.toStringTag`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toStringTag)

一个字符串值，用于创建对象的默认字符串描述。通过内置方法 `Object.prototype.toString` 调用。

### [`Symbol.unscopables`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/unscopables)

一个对象，指定对象自身或继承的属性名称，从关联对象的 "with" 环境绑定中排除该属性名称。

感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/symbols.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>












