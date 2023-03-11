---
highlight: vs2015
theme: juejin
---
# Narrowing
假设我们有一个名为 `padLeft` 的函数
```ts
function padLeft(padding: number | string, input: string): string { }
```
要是实现这样的功能：如果参数 `padding` 是 `number` 类型，把它将当作我们想要在 `input` 前加上的空格数量。如果是 `string` 类型，它应该是在 `input` 前追加的字符串。

让我们先实现参数 `padding` 是 `number` 类型时的逻辑：
```ts
function padLeft(padding: number | string, input: string) {
       return " ".repeat(padding) + input;
  // Error：
  // 实参 'string | number' 类型不能赋值给行参 'number' 类型
  // 'string' 类型不能赋值给 'number' 类型
}
```
>repeat 方法的定义是：(method) String.repeat(count: number): string

Uh-oh，我们得到一个 `padding` 的错误，TypeScript 警告我们 `number | string` 类型添加到 `number` 类型可能得不到我们想要的结果。或者说，我们没有先明确检查 `padding` 是否为 `number` 类型，也没有处理它是 `string` 类型的情况，所以让我们这么做：
```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```
看起来像 JavaScript 代码，这就是问题所在。除了注释类型的地方，这个 TypeScript 代码看起来像 JavaScript。这是因为 TypeScript 类型系统为了让你尽可能轻松的编写代码，而无需费尽心思的编写额外代码得到类型安全。

然而并不是看起来这么简单，其实这里涵盖了好多东西。例如 TypeScript 运行时使用了静态类型分析，分析值的类型，它将类型分析覆盖在 JavaScript 的运行时控制流结构上，如 `if/else`、三元表达式、循环、真值检查，等等，这些都可能影响值的类型。

在 `if` 检查中，TypeScript 看到 `typeof padding === "number"`，并将其理解为一种特殊形式的代码，称为类型守卫。TypeScript 跟随程序可能执行的路径，分析给定位置的值最可能具有的类型。它查看这些特殊的检查（类型守卫）和赋值，并将类型精炼为比声明更具体的类型，这个过程称为缩窄。在许多编辑器中，鼠标停留在值上就能观察到这些类型的变化。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc5667a5568a43d58a6c9e23bb7ed3b2~tplv-k3u1fbpfcp-watermark.image?)

以上结构，TypeScript 可以理解并缩窄。

## `typeof` 类型守卫
JavaScript 支持的 `typeof` 操作符，它可以提供 js 运行时，值所拥有的类型信息。TypeScript 中要求它返回以下的字符串：

-   `"string"`
-   `"number"`
-   `"bigint"`
-   `"boolean"`
-   `"symbol"`
-   `"undefined"`
-   `"object"`
-   `"function"`

就像上面例子，这个操作符经常出现在许多 JavaScript 库中，TypeScript 可以理解，并用它来缩窄不同分支中的类型。

在 TypeScript 中，检查 `typeof` 返回的值是一种类型守卫。TypeScript 编码了 `typeof` 对不同值的操作方式，因为它知道 JavaScript 中一些奇怪的行为。例如，在上面的列表中，也和 JavaScript 一样，Typeof 不返回字符串 null：

```ts
function printAll(strs: string | string[] | null) {
    if (typeof strs === "object") {
        for (const s of strs) {
        // Error：strs 可能为 “null”。
        console.log(s);
        }
    } else if (typeof strs === "string") {
        console.log(strs);
    } else {
        // do nothing
    }
}
```
在 `printAll` 函数中，我们尝试检查 `strs` 是否是一个对象，以确定它是否是一个数组类型(在 JavaScript 中，`Array` 使用 `typeof` 也返回 "object")。但是，在 JavaScript 中，`typeof null` 实际上也是 "object" ！这是历史遗留问题。

有丰富的 js 开发者可能不会感到惊讶，但并不是每个人都遇到过这种情况；幸运的是，TypeScript 会让我们知道 `strs` 只被缩窄到 `string[] | null`，而不是 `string[]`。

这能让我们很好的过渡到 "真值" 检查。

## 真值缩窄
"真值" 可能在字典中找不到，但在 JavaScript 中却经常听到它。

在 JavaScript 里，你可以在条件语句里使用任意表达式，`&&`，`||`，`if` 语句。取反运算符（`!`），等。例如，`if` 语句并不要求它们的条件总是具有布尔类型。

```ts
function getUsersOnlineMessage(numUsersOnline: number) {
    if (numUsersOnline) {
        return `There are ${numUsersOnline} online now!`;
    }
    return "Nobody's here. :(";
}
```
在 JavaScript 中，像 `if` 这样的结构，首先将它们的条件 "强转" 为布尔值来理解它们，然后根据结果是 `true` 还是 `false` 来选择它们的分支。

-   `0`
-   `NaN`
-   `""` (空字符串)
-   `0n` (0 的 `bigint` 类型)
-   `null`
-   `undefined`

以上值强转都为 `false`，除此之外的值强转都为 `true`。你可以通过 `Boolean` 函数 或 使用两个取反运算符（！！）将值强转为布尔值。(后者的优点是 TypeScript 会推断为缩窄的布尔字面量类型：`true`，而第一个推断为 `boolean` 类型。)
```ts
// 两个结果都为 'true'

Boolean("hello"); // 类型: boolean, 值: true

!!"world"; // 类型: true, 值: true
````
这种行为是很常见的，特别是用来防止 null 或 undefined 值的时候。例如，让我们尝试在 `printAll` 函数中使用它：
```ts
function printAll(strs: string | string[] | null) {
    if (strs && typeof strs === "object") {
        for (const s of strs) {
            console.log(s);
        }
    } else if (typeof strs === "string") {
        console.log(strs);
    }
}
```
注意，通过检查 `strs` 是否为真值，我们已经消除了上面的错误。这至少可以防止运行代码时出现以下可怕错误：
```ts
TypeError: null is not iterable
```
但是请记住，对原始类型进行真值检查很容易出错。例如：

```ts
function printAll(strs: string | string[] | null) {
// !!!!!!!!!!!!!!!!
// DON'T DO THIS!
// KEEP READING
// !!!!!!!!!!!!!!!!

    if (strs) {
        if (typeof strs === "object") {
            for (const s of strs) {
                console.log(s);
            }
        } else if (typeof strs === "string") {
            console.log(strs);
        }
    }
}
```
我们将整个函数体包装在一个真值检查中，但这有一个不易察觉的缺点：不再正确处理空字符串。

TypeScript 在这里完全没有提示信息，如果你不熟悉 JavaScript，那你就要注意这个行为。TypeScript 通常可以帮助你在早期捕获错误，但如果你选择对一个值什么都不做，它能做的事情就只有这么多，并没有过度的约束。如果你愿意，可以用一个 linter 确保自己正确处理了这样的情况。

最后一点，取反 `!` 可以过滤出否定分支
```ts
function multiplyAll(
    values: number[] | undefined,
    factor: number
    ): number[] | undefined {
    if (!values) {
        return values;
    } else {
        return values.map((x) => x * factor);
    }
}
```
## 同等缩窄
TypeScript 还使用 switch 语句并和相等性检查，如 `===`，`!==`，和 `!=` 来缩窄类型。例如:
```ts
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // x 和 y 都是字符串类型
    // 我们现在可以在 'x' or 'y' 上调用任意的字符串方法了
    x.toUpperCase();
    y.toLowerCase();
  } else {
    console.log(x); // 类型：(parameter) x: string | number
    console.log(y); // 类型：(parameter) y: string | boolean
  }
}
```
在上面的例子中，当我们检查 `x` 和 `y` 是否全等时，TypeScript 知道它们的类型也必须相等。由于 `x` 和 `y` 只有字符串是唯一相同类型，所以在第一个分支中 `x` 和 `y` 是字符串类型。

检查特定的字面量（不是变量）也可以。在上面的真值缩窄的部分中，我们写了一个 `printAll` 函数，这个函数很容易出错，因为它没有正确地处理空字符串。现在，我们可以做一个特定的检查来屏蔽 `null`， TypeScript 就正确地从 `strs` 类型中删除 null。
```ts
function printAll(strs: string | string[] | null) {
    if (strs !== null) {
        if (typeof strs === "object") {
            // strs 类型：(parameter) strs: string[]
            for (const s of strs) {
                console.log(s);
            }
        } else if (typeof strs === "string") {
            console.log(strs);
            // strs 类型：(parameter) strs: string
        }
    }
}
```
JavaScript 中 `==` 和 `!=` 这些宽松的相等性检查也被正确地缩窄。因为 `null == undefined`，所以检查是否有东西 `== null` ，实际上检查的是值是否为 `null` 或 `undefined`。`== undefined` 也一样。

```ts
interface Container {
    value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
    // 从类型中删除 'null' 和 'undefined'。
    if (container.value != null) {
        console.log(container.value);
        // container.value 类型：(property) Container.value: number
        container.value *= factor;
    }
}
```
## `in` 操作符缩窄
JavaScript 有一个操作符，用于确定对象是否具有某属性：`in` 操作符。TypeScript 将此作为一种缩窄潜在类型的方法。
```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
function move(animal: Fish | Bird) {
    if ("swim" in animal) {
        return animal.swim();
        // animal 类型：(parameter) animal: Fish
    }
    return animal.fly();
    // animal 类型：(parameter) animal: Bird
}
```
下面的代码：`"swim" in animal`。`"swim"` 是字面量字符串，`animal` 是联合类型。"true" 分支将缩窄 `animal` 类型为具有可选 或 必选属性 `swim`，"false" 分支将缩窄 `animal` 类型为具有可选 或 缺少属性 `swim`。也就是说，可选属性将存在于 `in` 缩窄后的 "true" 分支和 "false" 分支。

例如，一个人会游泳和飞行，因此应该出现在两个分支:
```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
    if ("swim" in animal) {
        animal; // animal 类型：(parameter) animal: Fish | Human
    } else {
        animal; // animal 类型：(parameter) animal: Bird | Human
    }
}
```
## `instanceof` 缩窄
JavaScript 有一个操作符可以检查一个值是否为另一个值的实例。具体讲就是，在 JavaScript 中 `x instanceof Foo` 检查 `x` 原型链是否包含 `Foo.prototype`。虽然我们不会深入讨论，你将会在了解类的时候经常遇到。你可能已经猜到，`instanceof` 也是一个类型守卫，TypeScript 可以通过 `instanceof` 进行缩窄。
>注意！数组，函数 instanceof Object 会返回 true，因为它们原型上都包含 Objecct.prototype
```ts
function logValue(x: Date | string) {
    if (x instanceof Date) {
    console.log(x.toUTCString());
    // x 类型：(parameter) x: Date
    } else {
    console.log(x.toUpperCase());
    // x 类型：(parameter) x: string
    }
}
```
## 赋值
正如我们之前提到的，当我们给任意变量赋值时，TypeScript 查看赋值的右侧，并为左侧进行合适的缩窄
```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
// x 类型：let x: string | number

x = 1;
console.log(x);
// x 类型：let x: number

x = "goodbye!";
console.log(x);
// x 类型：let x: string
```
注意，这些赋值都是有效的。即使是第一次赋值之后，观察到 `x` 的类型变成了 `number`，我们仍然可以把 `string` 类型的值赋值给 `x`。这是因为 `x` 一开始声明的类型为 `string | number`，并且赋值总是检查其一开始的声明类型。

如果我们给 `x` 赋值 `boolean` 类型的值，我们会得到报错信息，因为 `boolean` 不是一开始声明类型的一部分。并且 `x` 又变回了 `string | number` 类型。
```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
// x 类型：let x: string | number

x = 1;
console.log(x);
// x 类型：let x: number

x = true;
// Error：'boolean' 类型不能赋值给 'string | number' 类型
console.log(x);
// x 类型：let x: string | number
```
## 控制流分析
到目前为止，我们见过了 TypeScript 如何在特定分支中缩窄的一些基础案例。但是
除了从每个变量中查找类型守卫（`if`、`while`、条件语句，等）外，还有更多的事情要做。例如：
```ts
function padLeft(padding: number | string, input: string) {
    if (typeof padding === "number") {
        return " ".repeat(padding) + input;
        //  padding 类型：(parameter) padding: number
    }
    return padding + input;
    //  padding 类型：(parameter) padding: string
}
```
如果传递 `padding` 为数值给 `padLeft` 函数，将从第一个 `if` 块中返回。TypeScript 能够分析这段代码，并且看到函数体剩余部分（`return padding + input`）在这个案例中是不可到达的，因为 `padding` 是一个 `number` 类型。因此，它能够在函数体剩余部分移除 `padding` 类型的 `number`（`string | number` 缩窄为 `string`）。

这种基于 '可到达性' 的代码分析，称为控制流分析，并且 TypeScript 在遇到类型守卫和赋值时使用流分析去缩窄类型。当分析一个变量时，控制流可以一遍又一遍的分离与重新合并。并且可以观察到变量在每个点上都有不同的类型。

```ts
function example() {
    let x: string | number | boolean;
    x = Math.random() < 0.5;
    console.log(x); // x 类型：let x: boolean
    
    if (Math.random() < 0.5) {
        x = "hello";
        console.log(x); // x 类型：let x: string
       
    } else {
        x = 100;
        console.log(x); // x 类型：let x: number
        
    }
    return x; // x 类型：let x: string | number
}
```
## 使用类型谓词
到目前为止，我们一直在使用现有的 JavaScript 结构来处理窄化，然而，有时你希望更直接地控制类型在整个代码中的变化。

那么可以自定义类型守卫，只需要定义一个函数的返回类型为类型谓词
```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}
```
`pet is Fish` 是我们在例子中的类型谓词。一个谓词的形式为 `parameterName is Type`。其中 `parameterName` 必须是当前函数签名的参数名。

任何时候 `isFish` 调用某个变量时，如果与 `Fish | Bird | Human` 类型兼容, TypeScript 都会将变量缩窄为特定的类型。
```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { speak: () => void };
function isFish(pet: Fish | Bird | Human): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
let pet: Fish | Bird = Math.random() > 0.5 ? { swim: () => { } } : { fly: () => { } }

if (isFish(pet)) {
  pet.swim(); // pet 类型：let pet: Fish
} else {
  pet.fly(); // pet 类型：let pet: Bird
}
```
注意，TypeScript 不仅知道在 `if` 分支中，`pet` 是 `Fish` 类型；它还知道在 `else` 分支中，你没有 `Fish` 类型，所以 `else` 分支中必须 `Bird` 类型。

你可以使用类型守卫 `isFish` 来过滤一个` Fish | Bird` 数组，并获得一个 `Fish` 数组：
```ts
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// 等价于
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];

// 对于更复杂的示例，谓词可能需要重复
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
    if (pet.name === "sharkey") return false;
    return isFish(pet);
});
```
此外，类可以使用 [this is 类型](https://www.typescriptlang.org/docs/handbook/2/classes.html#this-based-type-guards) 来缩窄它们的类型。
## 辨别联合类型
到目前为止，我们看到的大多数例子都在用简单类型（如 `string`，`boolean` 和 `number`）缩窄为单个变量。虽然这很常见，但在 JavaScript 中，大多数情况下我们要处理稍微复杂一些的结构。

假设我们正在尝试编码圆形和正方形的形状，圆有半径，正方形有边长。我们将使用 `kind` 字段来告诉我们处理的是哪个形状。先尝试定义一个 `Shape` 类型：
```ts
interface Shape {
    kind: "circle" | "square";
    radius?: number;
    sideLength?: number;
}
```
注意，我们正在使用字符串字面量联合类型：`"circle"` 和 `"square"` ，告诉我们是该把形状看成圆形还是正方形。通过使用 `"circle" | "square"` 类型而不是 `string` 类型，可以避免拼写错误：
```ts
function handleShape(shape: Shape) {
    if (shape.kind === "rect") {
        // 这个条件将总是返回 'false'，因为 "circle" | "square"` 类型 和 "rect" 不重叠
    }
}
```
我们可以编写一个 `getArea` 函数，根据它处理的是圆形还是正方形来应用正确的逻辑。我们先来处理圆形。
```ts
function getArea(shape: Shape) {
    return Math.PI * shape.radius ** 2;
    // Error：对象可能是 'undefined'
}
```
开启 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 的情况下（之前说了，例子都会开启所有严格检查），给予了我们一个错误——因为 `radius` 可能没未定义。但是如果我们对 `kind` 属性执行适当的检查呢？
```ts
function getArea(shape: Shape) {
    if (shape.kind === "circle") {
        return Math.PI * shape.radius ** 2;
        // 对象可能是 'undefined'
    }
}
```
TypeScript 还是不知道这里到底有没有 `radius` 属性。在这一点上，我们比类型检查更了解值的信息，我们可以尝试使用非空断断言（`shape.radius` 后边附加 `!`），去告诉 TypeScript `radius` 属性是有定义的。
```ts
function getArea(shape: Shape) {
    if (shape.kind === "circle") {
        return Math.PI * shape.radius! ** 2;
    }
}
```
但是这并不是很好的解决方案。我们不得不用非空断言（`!`）去让类型检查相信 `shape.radius` 是有定义的，如果我们开始移动代码，这些断言就很容易出错。另外，就算开启 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) ，我们也能意外地访问这些字段中的任何一个（因为告诉了类型检查，可选属性总是存在）。我们可以有更好的解决方案。

`Shape` 定义是有问题的，类型检查器无法根据 `kind` 属性知道是否存在 `radius` 或 `sideLength` 属性。我们需要将已知的信息传达给类型检查器。让我们重新定义 `Shape`。

```ts
interface Circle {
    kind: "circle";
    radius: number;
}
interface Square {
    kind: "square";
    sideLength: number;
}

type Shape = Circle | Square;
```
现在，我们把 `Shape` 分成了两种类型，它们分别有相同的 `kind` 属性，但值不同，并且 `radius` 和 `sideLength` 在它们各自的类型里被声明为必选的属性。

让我们看看当访问  `Shape.radius` 的时候会发生什么
```ts
function getArea(shape: Shape) {
    return Math.PI * shape.radius ** 2;
    // 'Shape' 类型上不存在 'radius' 属性
    // 'Square' 类型上不存在 'radius' 属性
}
```
虽然依然错误，但是错误信息已经和一开始定义的 `Shape` 类型不同了。之前 `radius` 为可选时，我们会得到一个错误（启动了[strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks)），TypeScript 告诉我们无法判断该属性是否存在。现在 `Shape` 是个联合类型，告诉我们 `shape` 对象可能是一个 `Square` 类型，而 `Square` 上没有定义 `radius` 属性。只有 `Shape` 为联合类型时才会不顾及如何配置 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 都会产生错误。

如果我们再次检查 `kind` 属性呢?
```ts
function getArea(shape: Shape) {
    if (shape.kind === "circle") {
        return Math.PI * shape.radius ** 2;
        // shape 类型：(parameter) shape: Circle
    }
}
```
现在消除了错误！当联合类型中的每个类型都具有一个公共的属性，但不同字面量值时，TypeScript 认为这是一个能辨识的联合，并可以缩窄联合类型的成员。

在本例中，`kind` 是公共属性（即 `Shape` 的判别属性）。检查 `kind` 属性是否为 `"circle"` ，可以去掉 `Shape` 联合类型中 `kind` 属性不为 `"circle"` 类型的成员。缩窄 `shape` 对象的类型为 `Circle`。

现在也适用于 `switch` 语句检查。并且不需要非空断言（!）编写 `getArea` 函数
```ts
function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
            // shape  类型：(parameter) shape: Circle
        case "square":
            return shape.sideLength ** 2;
            // shape  类型：(parameter) shape: Square
    }
}
```
这里最重要的是 `Shape` 的定义。将正确的信息传递给了 TypeScript —— `Circle` 和 `Square` 是分别具有特定 `kind` 字段的独立类型——这是至关重要的。这样做可以让我们编写类型安全的 TypeScript 代码，看起来与我们原本编写的 JavaScript 没有什么不同。但是从这里开始，类型系统就能够做 "正确" 的事情，并在 `switch` 语句的每个分支中算出类型。

>顺便说一句，尝试一下上面的例子，并删除一些返回关键字。你将看到在 **switch** 语句中，不同的子句里面意外出错时，类型检查可以帮助避免错误。

有辨识的联合类型不仅仅用于辨别形状。它们适用于在 JavaScript 中表示，任何类型的消息传递方案，例如通过网络发送消息（客户端/服务器通信）或在状态管理框架中编码转变。


## `never` 类型
当缩窄时，你可以将联合类型的选项减少到一个点。就是说，消除了所有可能性，什么都不剩下。在这些情况下，TypeScript 将使用 `never` 类型来表示状态不应该存在。
### 穷尽检查
`never` 类型能赋值给任意类型；但是，没有任何类型能赋值给 `never` 类型（除了 `never` 自身）。这意味着你可以在 `switch` 语句中使用缩窄并依赖 `never` 来执行穷尽的检查。

如下例子，添加 `default` 在我们的 `getArea` 函数中，当所有可能的情况都不能处理时（例子中只有两种类型），它会尝试将形状赋值为 `never`：
```ts
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
        default:
            const _exhaustiveCheck: never = shape;
            return _exhaustiveCheck;
    }
}
```
向 `Shape` 联合类型添加新成员，将导致 TypeScript 错误，因为 `shape` 还有是 `Triangle` 类型的可能：
```ts
interface Triangle {
    kind: "triangle";
    sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
        default:
            const _exhaustiveCheck: never = shape;
            // Error：'Triangle' 类型不能赋值给 'never' 类型
            return _exhaustiveCheck;
    }
}
```
感谢观看，如有错误，望指正

>官网地址： <https://www.typescriptlang.org/docs/handbook/2/narrowing.html>
>
>github 资料： <https://github.com/Mario-Marion/TS-Handbook>

