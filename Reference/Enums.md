# Enums

枚举类型是 TypeScript 为数不多的，不是从 JavaScript 类型级扩展的特性之一。

枚举允许开发者定义一组命名常量。使用枚举可以更容易的实现记录意图，或者创建一组不同的用例。TypeScript 提供了数值枚举类型和字符串枚举类型。

## 数值枚举

首先从数值枚举开始，如果你熟悉其他语言，可能会很熟悉它。枚举可以使用 `enum` 关键字定义。

```ts
enum Direction {
    Up = 1,
    Down,
    Left,
    Right,
}
```

上面有一个数值枚举，其中 `Up` 初始化为 `1`。从该点开始往下的所有成员自动递增。换句话说，`Direction.Up` 值为 `1`，`Down` 是 `2`，`Left` 是 `3`，`Right` 是 `4`。

我们完全可以忽略初始化设置：

```ts
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

这里，`Up` 值为 `0`，`Down` 值为 `1`，等。我们可能不关心成员值本身，但关心的是每个值，在同一枚举中，与其他值不同，这时候自动递增行为就很有用。

使用枚举很简单：只需要将任何成员作为枚举本身的属性访问，并使用枚举的名称声明类型：

```ts
enum UserResponse {
  No = 0,
  Yes = 1,
}

function respond(recipient: string, message: UserResponse): void {
  // ...
}

respond("Princess Caroline", UserResponse.Yes);
```

数值枚举可以在 [计算成员和常量成员（见下文）](https://juejin.cn/post/7224344504776589373#heading-4) 中混合使用。但是，有初始化设置的枚举成员，需要放在最后（数值常量成员例外），或者 有初始化设置的枚举成员（数值常量成员例外），后面必须有初始设置的数值常量成员。

BAD ❌：

```ts
const getSomeValue = () => 23;

enum E {
  A = getSomeValue(),
  B,
}
Error：// Enum member must have initializer.
```

GOOD ✔：

```ts
const getSomeValue = () => 23;

enum E {
  A,
  B = getSomeValue(),
}
// ---或---
enum E {
  A = getSomeValue(),
  B = 1,
  C
}
```

## 字符串枚举

字符串枚举和数值枚举相似，但是有以下一些细微的 [运行时差异](https://juejin.cn/post/7224344504776589373#heading-6)。在字符串枚举中，每个成员使用 字符串字面量 或使用 另一个字符串枚举成员 进行常量初始化。

```ts
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```

虽然字符串枚举没有自动递增的行为，但字符串枚举的好处是它们可以很好地 "序列化"。换句话说，如果你正在调试并且必须读取数字枚举的运行时值，则该值通常是隐晦难懂的 —— 它本身并不传达任何有用的含义（尽管 [反向映射](https://juejin.cn/post/7224344504776589373#heading-8) 通常可以提供帮助）。字符串枚举允许代码在运行时提供有意义且可读性好的值，而与枚举成员本身的名称无关。

## 多样化枚举

从技术上讲，枚举可以是字符串和数字成员混合使用，但没必要这样做：

```ts
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = "YES",
}
```

除非你真的想以巧妙的方式利用 JavaScript 的运行时行为，否则建议你不要这样做。

## 计算成员和常量成员

每个枚举成员都有一个与之关联的值，该值可以是常量，也可以是计算值。如果满足以下条件，枚举成员被认为是常量成员：

- 它是枚举中的第一个成员，并且没有初始化设置，在这种情况下，它被默认赋值为 `0`：
  ```ts
  // E.X is constant:
  enum E {
    X,
  }
  ```
- 它没有初始化设置，并且前一个枚举成员是数值常量成员。在这种情况下，当前枚举成员的值将等于前一个枚举成员的值加 1。
  ```ts
  // All enum members in 'E1' and 'E2' are constant.
  enum E1 {
    X,
    Y,
    Z,
  }

  enum E2 {
    A = 1,
    B,
    C,
  }
  ```

- 枚举成员使用常量枚举表达式初始化设置。常量枚举表达式是 TypeScript 表达式的子集，可以在编译时计算出值。如果满足以下条件，表达式就是常量枚举表达式：

    1. 一个字面量枚举表达式（字符串字面量或数值字面量）
    2. 引用之前定义的常量枚举成员（可以来自不同的枚举）
    3. 带括号的常量枚举表达式
    4. `+`、`-`、`~` 一元运算符应用于常量枚举表达式
    5. 常量枚举表达式是 `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^` 二进制运算符的运算对象

  常量枚举表达式计算值为 `NaN` 或 `Infinity`，在编译时会报错。

除了以上的其它情况下，枚举成员都被认为是计算成员：

```ts
enum FileAccess {
  // constant members
  None,
  Read = 1 << 1,
  Write = 1 << 2,
  ReadWrite = Read | Write,
  // computed member
  G = "123".length,
}
```

## 联合枚举和枚举成员类型

常量枚举成员有一个特殊的子集是不计算的：字面量枚举成员。字面量枚举成员是没有初始化值的常量枚举成员，或值初始化为：

- 任意字符串字面量（例如：`"foo"`，`"bar"`，`"baz"`）
- 任意数值字面量（例如：`1`，`100`）
- 一元减号应用于任意数值字面量（例如：`-1`，`-100`）

当枚举中的所有成员都具有字面量的枚举值时，就会产生一些特殊的语义。

首先，枚举成员也成为类型！比如说某些成员只能具有枚举成员的值：

```ts
enum ShapeKind {
  Circle,
  Square,
}

interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}

interface Square {
  kind: ShapeKind.Square;
  sideLength: number;
}

let c: Circle = {
  kind: ShapeKind.Square,
  Error：// Type 'ShapeKind.Square' is not assignable to type 'ShapeKind.Circle'.
  
  radius: 100,
};
```

另一个变化是枚举类型本身有效地成为每个枚举成员的联合。使用联合枚举，类型系统能够利用它知道枚举本身中存在的确切值集合。因此，TypeScript 可以捕捉到我们可能错误的去比较值。例如：

```ts
enum E {
  Foo,
  Bar,
}

function f(x: E) {
  if (x !== E.Foo || x !== E.Bar) {
      Error：// This comparison appears to be unintentional because the types 'E.Foo' and 'E.Bar' have no overlap.
      //
  }
}
```

在例子中，我们首先检查 `x` 是否不是 `E.Foo`。如果检查成功，那么我们的 `||` 将短路，并且 `if` 的主体将运行。然而，如果检查不成功，那么 `x` 只能是 `E.Foo`，所以看它是否等于 `E.Bar` 没有意义。

## 运行时的枚举

枚举在运行时实际是一个对象。

```ts
enum E {
  X,
  Y,
  Z,
}
```

可以传递给函数：

```ts
enum E {
  X,
  Y,
  Z,
}

function f(obj: { X: number }) {
  return obj.X;
}

// Works, since 'E' has a property named 'X' which is a number.
f(E);
```

## 编译时的枚举

尽管枚举在运行时是一个真实对象，但使用关键字 `keyof` 的工作方式与对典型对象的期望不同。相反，使用 `keyof typeof` 来获得类型，该类型表示枚举所有键，为字符串字面量联合类型。

```ts
enum LogLevel {
  ERROR,
  WARN,
  INFO,
  DEBUG,
}

/**
 * This is equivalent to:
 * type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
 */
type LogLevelStrings = keyof typeof LogLevel;

function printImportant(key: LogLevelStrings, message: string) {
  const num = LogLevel[key];
  if (num <= LogLevel.WARN) {
    console.log("Log level key is:", key);
    console.log("Log level value is:", num);
    console.log("Log level message is:", message);
  }
}
printImportant("ERROR", "This is a message");
```

### 反向映射

除了为成员创建具有属性名称的对象之外，数值枚举成员还获得从枚举值到枚举名称的反向映射。例如：

```ts
enum Enum {
  A,
}

let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

以下为编译后的 JavaScript 代码：

```js
"use strict";
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

在生成的代码中，枚举被编译成一个对象，该对象存储正向（`name` -> `value`）和反向（`value` -> `name`）映射。对其他枚举成员的引用总是作为属性访问发出，从不 [内联](https://juejin.cn/post/7224344504776589373/#heading-9)。

> 字符串枚举成员根本不会生成反向映射。

### 内联枚举

内联枚举是指在 TypeScript 中，当一个枚举成员被引用时，编译器将其直接替换为其对应的值，而不是生成对该枚举的访问。这样做可以在编译后的 JavaScript 代码中减少对枚举的依赖，从而减小代码的体积。

```ts
const enum Color {
  Red,
  Green,
  Blue
}

const myColor = Color.Green
```

编译后:

```js
"use strict";
const myColor = 1 /* Color.Green */;
```

这个例子中，我们使用了内联枚举将枚举成员 `Color.Green` 替换为其对应的值 `1`。这样在编译后的 JavaScript 代码中，`myColor` 的值就直接是 `1`，而不是通过对枚举 `Color` 的访问来获取该值。

需要注意的是，虽然内联枚举可以使编译后的代码更加简洁，但是过度使用内联枚举可能会降低代码的可读性和可维护性，因为枚举成员的含义将不再明确。因此，在选择使用内联枚举时，需要权衡代码大小和可维护性之间的折衷。

### `const` 枚举

在大多数情况下，枚举是一个完全有效的解决方案。然而有时候需求是严格的。为了避免在访问枚举值时，额外生成代码和附加的间接操作消耗，可以使用 `const enum`。在枚举上使用 `const` 修饰符定义 Const 枚举：

```ts
const enum Enum {
  A = 1,
  B = A * 2,
}
```

Const 枚举只能使用常量枚举表达式，并且与常规枚举不同，它们在编译过程中是被完全擦除的。枚举成员使用 [内联](https://juejin.cn/post/7224344504776589373/#heading-9)。这是因为 Const 枚举不能有计算成员。

```ts
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}

let directions = [
  Direction.Up,
  Direction.Down,
  Direction.Left,
  Direction.Right,
];
```

以下为编译后的 JavaScript 代码：

```ts
"use strict";
let directions = [
    0 /* Direction.Up */,
    1 /* Direction.Down */,
    2 /* Direction.Left */,
    3 /* Direction.Right */,
];
```

#### `const` 枚举缺陷

内联枚举值一开始很简单，但有一些微妙的含义。这些缺陷只适用于环境 const 枚举（基本上是 `.d.ts` 文件中的 const 枚举）。并在项目之间共享它们，但是如果你正在发布或使用 `.d.ts` 声明文件，这些陷阱可能适用于你，因为 `tsc --declaration` 将 `.ts` 文件转换为 `.d.ts` 文件。

1. 由于 [isolatedModules documentation](https://www.typescriptlang.org/tsconfig#references-to-const-enum-members) 中列出的原因，该模型从根本上与环境枚举不兼容。这意味着如果你发布环境 const 枚举，下游消费者将不能同时使用 [isolatedModules](https://www.typescriptlang.org/tsconfig#isolatedModules) 和那些枚举值。
2. 你可以很容易地在编译时内联依赖的版本 A 的值，并在运行时导入版本 B。版本 A 和版本 B 的枚举可能有不同的值，如果不小心，结果会导致 [令人惊讶的 bug](https://github.com/microsoft/TypeScript/issues/5219#issue-110947903)，比如使用错误的 `if` 语句分支。这些错误尤其有害，因为通常在构建项目的同时运行自动化测试，使用相同的依赖版本，这完全忽略了这些错误。
3. [`importsNotUsedAsValues: "preserve"`](https://www.typescriptlang.org/tsconfig#importsNotUsedAsValues) 不会省略作为值使用的 const 枚举的导入，但是环境 const 枚举不能保证运行时 `.js` 文件存在。无法解析的导入会在运行时导致错误。明确省略导入的常用方法是 [纯类型导入](https://www.typescriptlang.org/docs/handbook/modules.html#importing-types)，目前 [不允许使用 const 枚举值](https://github.com/microsoft/TypeScript/issues/40344)。

以下是避免这些缺陷的两种方法：

A. 完全不要使用 const 枚举。在 [linter](https://github.com/typescript-eslint/typescript-eslint/tree/main/docs) 的帮助下，可以很容易地禁止 const 枚举。显然，这避免了const 枚举的任何问题，但阻止了项目内联自己的枚举。与内联其他项目的枚举不同，内联项目自己的枚举没有问题，并且具有性能影响。

B. 不要发布环境 const 枚举，通过使用 [preserveConstEnums](https://www.typescriptlang.org/tsconfig#preserveConstEnums) 对它们进行反序列化。这是 [TypeScript 项目本身](https://github.com/microsoft/TypeScript/pull/5422) 内部采用的方法。[preserveConstEnums](https://www.typescriptlang.org/tsconfig#preserveConstEnums) 对 const 枚举和普通枚举发出相同的 JavaScript。然后，你可以在 [构建步骤中](https://github.com/microsoft/TypeScript/blob/1a981d1df1810c868a66b3828497f049a944951c/Gulpfile.js#L144) 安全地从 `.d.ts` 文件中删除 `const` 修饰符。

这样下游消费者就不会内联项目中的枚举，避免了上面的缺陷，但项目仍然可以内联自己的枚举，而不是完全禁止 const 枚举。

## 环境枚举

环境枚举用于描述已经存在的枚举类型的形状。

```ts
declare enum Enum {
  A = 1,
  B,
  C = 2,
}
```

环境枚举和非环境枚举之间的一个重要区别，在常规枚举中，成员没有初始化设置，而且前面的枚举成员被视为常量成员，则该成员将被视为常量成员。相比之下，没有初始化设置的环境（非const）枚举成员总是被认为是计算成员。

## 对象 VS 枚举

在现代 TypeScript 中，你可能不需要枚举，有一个带 `as const` 的对象就足够了：

```ts
const enum EDirection {
  Up,
  Down,
  Left,
  Right,
}

const ODirection = {
  Up: 0,
  Down: 1,
  Left: 2,
  Right: 3,
} as const;

EDirection.Up;
// (enum member) EDirection.Up = 0
ODirection.Up;
// (property) Up: 0

// Using the enum as a parameter
function walk(dir: EDirection) {}

// It requires an extra line to pull out the values
type Direction = typeof ODirection[keyof typeof ODirection];
function run(dir: Direction) {}

walk(EDirection.Left);
run(ODirection.Right);
```

与 TypeScript 的枚举格式相比，支持这种格式的最大理由是，它使你的代码库与 JavaScript 的状态保持一致，[当/如果](https://github.com/rbuckton/proposal-enum) 枚举被添加到 ECMAScript 正式标准，你可以转移为其它语法。

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/enums.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>


