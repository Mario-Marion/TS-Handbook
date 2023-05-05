# Everyday Types
本章节将介绍一些在 JavaScript 中常见的值的类型，并讲解在 TypeScript 中，描述这些类型的相应方法。没讲解到的类型，将在以后的章节描述。

除了类型注释外，类型还能出现在很多地方。我们将了解类型本身，还有了解在哪些地方可以引用它们形成新的结构。

我们将从最基础的类型和你编写 JavaScript 或 TypeScript 时遇到的常用类型开始。这些将是以后构建复杂类型的基础。

## 原始类型：`string`，`number`，`boolean`
JavaScript 有三个常用的 [原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)：`string`，`number` 和 `boolean`。它们在 TypeScript 中都有相应的类型。如果你在这些类型的值上使用 JavaScript 的 `Typeof` 操作符将会看到它们的类型名称，类型名称和 TypeScript 是相同的。

- `string` 表示字符串，例如：`"Hello，world"`
- `number` 表示数值，例如：`42`。JavaScript 对整数没有特殊的运行时值，所以没有 `int` 或 `float`——只有简单的 `number`
- `boolean` 表示布尔值，`true` 和 `false`两个值

>类型名称 **String**，**Number**，和 **Boolean**（开头大写字母）是符合规范的，但指的是某些特殊内置类型（表示 JS 中 String，Number，Boolean 构造的对象类型，例如：new String() 等...），很少出现在你的代码中。

## Arrays
要表示像 `[1,2,3]` 这样的数组类型，你可以使用 `number[]` 语法。这个语法适用于任何数组类型（例如：string[] 是一个字符串数组）。你可能见过 `Array<number>` 这种写法，两种写法是一样的。学习泛型的时候再了解更多 `T<U>` 语法。

>注意！`[number]` 是不同的类型，可参考：[Tuples](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)

## `any`
TypeScript 也有特殊类型，`any`，可以用在你不想因类型检查而错误的特殊值。

当值为 `any` 类型，你可以访问它的任意属性（属性的类型也将转换为 `any` 类型），或像函数一样调用，或将它赋值给其它类型：
```ts
let obj: any = { x: 0 };
// 下面代码不会抛出编译错误
// 使用' any ' 相当于禁用对此类型的类型检查
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```
当你不想编写出很长的类型，去告诉 TypeScript 某行代码是没问题的时候，可以用 `any` 类型。

### `noImplicitAny`
当值没有指定类型，并且 TypeScript 不能根据上下文推断出类型时，编译器会将此值设置为 `any`类型。

如果想要避免这种情况，可以使用编译器标志 [noImplicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) 。去标记任何隐式的 `any` 为错误。

## 变量上的类型注释
当你使用 `const`，`var` 或 `let` 声明一个变量的时候，你可以选择添加类型注释去明确指定变量类型。
```ts
let myName: string = "Alice";
```
>TypeScript 没有使用 “左类型” 风格的声明，例如：int x = 0；类型注释总是在后边。

大多数情况下，不需要这么写。因为 TypeScript 会尝试去自动推断你代码的类型。例如：基于初始化类型推断出来：
```ts
// 不需要类型注释 -- 'myName' 推断为 'string' 类型
let myName = "Alice";
```
在大多数情况下，不需要明确的去了解 TypeScript 推理规则。你可以尽量少用类型注释，让 TypeScript 去自动推断类型——你可能会惊讶地发现，你只需要很少的类型注释就能让 TypeScript 完全理解发生了什么。

## Functions
在 JavaScript 中，函数是传递数据的主要方法。TypeScript 允许你去指定函数输入和输出值的类型。

### 参数类型注释
当你声明一个函数，你可以在每个形参的后面添加类型注释去声明函数接收什么类型的参数。参数类型注释跟在行参名称的后面：

```ts
// 形参类型注释
function greet(name: string) {
    console.log("Hello, " + name.toUpperCase() + "!!");
}
```
当行参有类型注释的时候，该函数的参数将被检查:
```ts
// Would be a runtime error if executed!
greet(42);
// Error：'number' 类型的实参不能赋值给 'string' 类型的行参
```
>即使你的参数没有类型注释，TypeScript 也会检查你传递的参数数量是否正确。

### 返回类型注释
你还可以添加返回类型注释，返回类型注释出现在形参列表的后面
```ts
function getFavoriteNumber(): number {
    return 26;
}
```
很像变量类型注释，你通常不需要返回类型注释，因为 TypeScript 会基于函数的 `return` 语句推断出函数的返回类型。类型注释在上面的例子中没有改变任何东西。一些代码库会显式的指定返回类型用于文档说明。去防止意外的更改，或者只是出于个人喜好。

### 匿名函数
匿名函数和函数声明略有不同，当一个函数出现在任何 TypeScript 可以确定它被调用的地方，该函数的参数将自动给定类型。

例如：
```ts
// 这里没有类型注释，但是 TypeScript 可以发现错误
const names = ["Alice", "Bob", "Eve"];

// 函数的上下文类型
names.forEach(function (s) {
    console.log(s.toUppercase());
    // Error：'string' 类型上不存在 'toUppercase' 属性。你是打算用 'toUpperCase' 吗？
});

// 上下文类型同样适用于箭头函数
names.forEach((s) => {
    console.log(s.toUppercase());
    // Error：'string' 类型上不存在 'toUppercase' 属性。你是打算用 'toUpperCase' 吗？
});
```
尽管形参 `s` 没有类型注释，TypeScript 使用 `forEach` 方法的类型，以及推断出的数组类型，来确定参数 `s` 将具有的类型。

这个过程称为上下文类型，因为函数发生的上下文告知它应该具有什么类型。

和推断规则类似，你不需要明确了解是如何发生的，但了解它会发生什么，可以帮助您意识到何时不需要类型注释。稍后，我们将看到更多示例，解释值出现的上下文是如何影响其类型的。

## 对象类型
除了原始类型，还经常的类型就是对象类型。指的是任何带有属性的 JavaScript 值，几乎所有值都是！让我们定义一个对象类型，简单列出它的属性和属性的类型。

例如，有一个函数，接收一个对象：
```ts
// 参数类型注释是一个对象类型
function printCoord(pt: { x: number; y: number }) {
    console.log("The coordinate's x value is " + pt.x);
    console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```
我们的参数类型注释有两个属性——`x` 和 `y`——它们都是 `number` 类型。你可以使用 `,` 或 `;` 分隔符去分隔属性，并且最后一个属性可不用写分隔符。

如果你没有指定属性类型，它将被认定为 `any`。

### 可选属性
对象类型还可以指定属性是否为可选的。在属性名称的后面添加一个 `?` 标识符。

```ts
function printName(obj: { first: string; last?: string }) {
    // ...
}
// 都可以
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```
在 JavaScript 中，如果你访问一个不存在的属性，你将得到 `undefined` 值，而不是运行时错误。所以，当你读取一个可选属性时，必须检查它是否为 `undefined`。
```ts
function printName(obj: { first: string; last?: string }) {
    // 如果没有 'obj.last' 属性可能会崩溃!
    console.log(obj.last.toUpperCase());
    // Error：Object is possibly 'undefined'.
    
    if (obj.last !== undefined) {
        // OK
        console.log(obj.last.toUpperCase());
    }
    // 使用先进的 JavaScript 语法，安全替代方案:
    console.log(obj.last?.toUpperCase());
}
```
## 联合类型
TypeScript 类型系统允许你使用各种各样的操作符在现有类型的基础上构建新类型。让我们用有趣的方法开始组合它们吧。

### 定义一个联合类型
组合类型的第一种方法是联合类型。联合类型是由两个或多个其他类型组成的，表示值可以是这些类型中的任何一种。我们把这些类型中的每一个称为联合成员。

让我们来编写一个可以操作 `string` 或 `numer` 的函数：
```ts
function printId(id: number | string) {
    console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
// Error：'{ myID: 22342 }' 类型实参不能赋值给 'string | number ' 类型形参
```
### 使用联合类型
赋值给联合类型匹配的值很容易——只需提供与联合的任何成员匹配的类型。那如果你有一个联合类型的值，又该如何使用它呢?

使用联合类型的值要注意，TypeScript 只允许对联合类型的**每个成员都有效的操作**。例如，如果你有联合 `string | number`，你不能使用仅在 `string` 上可用的方法：
```ts
function printId(id: number | string) {
    console.log(id.toUpperCase());
    // Error：
    // 类型 'string | number' 不存在 'toUpperCase' 属性。
    // 类型 'number' 不存在 'toUpperCase' 属性。
}
```
解决方案是用代码缩窄联合类型范围，就像在的 JavaScript 中一样。当 TypeScript 可以根据代码结构为值推断出更具体的类型时，就会发生缩窄。

例如，TypeScript 知道只有 `string` 的值使用 `typeof` 操作符返回 `"string"`：
```ts
function printId(id: number | string) {
    if (typeof id === "string") {
        // 在这个分支中, id 是 'string' 类型
        console.log(id.toUpperCase());
    } else {
        // 这里, id 是 'number' 类型
        console.log(id);
    }
}
```
另一个例子，使用 `Array.isArray` 的函数：
```ts
function welcomePeople(x: string[] | string) {
    if (Array.isArray(x)) {
        // 这里 'x' 是 'string[]' 类型
        console.log("Hello, " + x.join(" and "));
    } else {
        // 这里 'x' 是 'string' 类型
        console.log("Welcome lone traveler " + x);
    }
}
```
注意，在else分支中，我们不需要做任何特殊的事情，如果 `x` 不是 `string[]` 类型，那么它就是 `string` 类型。

联合类型的所有成员有可能会有共同点。例如，联合中的每个成员都有一个共同的属性，那么你可以使用该属性而不需要窄化。

例子，数组和字符串都有一个 `slice` 方法:
```ts
// 返回类型被推断为 'number[] | string'
function getFirstThree(x: number[] | string) {
    return x.slice(0, 3);
}
```
## 类型别名
我们之前使用对象类型和联合类型是通过直接编写类型注释。这很方便，但是不能在其它相同类型的地方引用它，这时候我们就得用类型别名。

类型别名的语法是：
```ts
type Point = {
    x: number;
    y: number;
};

// 和之前的例子完全一样
function printCoord(pt: Point) {
    console.log("The coordinate's x value is " + pt.x);
    console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```
你可以使用类型别名给任意类型命名。例如，类型别名还可以给联合类型命名:
```ts
type ID = number | string;
```
请注意，别名只是别名而已——你不能使用类型别名来创建同一类型的不同的 “版本”。当您使用别名时，就像你编写了实际类型一样。如下例子，这段代码可能看起来不合法，但其实是没问题的，因为 `UserInputSanitizedString` 和 `new input` 都是 `string` 类型:
```ts
declare function getInput(): string;
declare function sanitize(str: string): string;
// ---cut---
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
    return sanitize(str);
}

let userInput = sanitizeInput(getInput());

// 仍然可以重新赋值字符串
userInput = "new input";
```
## 接口
接口声明是命名对象类型的另一种方式:
```ts
interface Point {
    x: number;
    y: number;
}

function printCoord(pt: Point) {
    console.log("The coordinate's x value is " + pt.x);
    console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```
就像我们上面使用的类型别名一样，这个示例就像我们在使用匿名对象类型。TypeScript 只关心我们传递给 `printCoord` 的值的结构——它只关心它是否具相同的属性。只关心类型的结构和功能，这就是为什么我们称 TypeScript 为结构类型系统的原因。

### 类型别名和接口之间的区别
类型别名和接口非常相似，在许多情况下使用哪个都可以。接口的大多数特性，类型是可以用的，关键的区别是，类型别名不能参与[声明合并](https://www.typescriptlang.org/play?#code/PTAEnvlRTuUB41DgzQSOUFRygYFULAqgQt0KXGhj5UDD-gseUG34wGQjAIFUDc9ALgChqBLAOwBcBTAJwDMBDAYxdACyXALbCuAG1ABvaqDmgA5iwYBXAM6VQapm0YLqAX1qNWnXvyGiJ02fIBGbFiwAmAfk3bdDfUeo8A9gzaoFwMdGLimpYRoAC8NvKKyuqaAEQAjABMAMwALKkANLZyIKCAjJqApLGA9GaAB2qAXHKgDk7OoICj+oDeGVWA6tqAZN6ATHLFjY4umumGtKWAsHKAvwFwSIAfbsjogN4+gNHqgNRKqLRMAJ4ADvwASix7THTi-PEyiUqqGlo6euPUuwegx6fnlwn2w24ej28hiAA)，但接口可以。

![1678285151707.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acefb5fe1be242af830a5fa4924e1054~tplv-k3u1fbpfcp-watermark.image?)

![1678285167682.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbc26043927c4339b2b740de7f9dabc8~tplv-k3u1fbpfcp-watermark.image?)

你将在后面的章节中了解更多相关的知识，如果你不能立即理解以下概念，请不要担心。
- 在 TypeScript 4.2 版本之前，类型别名的名称可能会[出现在错误消息中](https://www.typescriptlang.org/play?ts=4.1.5#code/PTAEjR-RF6MCzVExUx76MBG2h7A0GA6hpzUBN+h-eUBSuoBLAOwBcBTAJwDMBDAYzNEBC3QWBVAHzwCgPjzr7GAsjQC2wmgBtQAbw6g5oIiLIAuUAGcSFYgHMOAXy5UArkTokCAeyKgydABYWhoiQAphqp2PEBKabPl0VmoW4mQAdOIW2m5hisJk3vpcIDZh2mGggNBygEbpTAiAh-LIgNwGgPRmeOxynhKgHLYOVeIuUnJxKqAAjABMAMwALN2geokcKYDK+oAAcoBUcoDePoDR6oCFNoCQ5oDzfoDj8YDfcoCq8oBG+oDw+kyAsyaAOvKAb6aAMP+AxQmAEnKABvKAs3KAbI6A98qAp3KA6fqADEqAm-Ezs4DUSiwjoAuOUAQ8qAYADlBwSABPAAOjAAMgQAF40CgAE1AAF4-PIFEpVBotERdAYOMZTOYrDZ7BYkaiMS5xKp6Wj0b4ZHjAkRgqEIlEmbElIkySlAAJGLwKyEeZVwFVZGNqtIV6Ka+Piqi6fQArENkmBAGLyJQmv0A1RGAG3jAEbGJxGYEA0fKAIM1AFj-eEAYXKAUMVAHb+gFPdQDuinNQAB5YQEEhnCUIFCoQB8ZoAuT0Ay36AHPM5oAYAMAsHIldhcGHw0AAQQo9DsRAImJxwdDAB5mq1CZodAAaUChbRqVQADkGTYA5C21N2AHw1ckmMyWax1CwAZVhpcoTNUBaLJfZuICQRC4Ui0XEQviIttoClS3KbHzhfsK6VDhnc4oatrHR6vW1Tb7HaGQA)，而接口的名称则始终出现在报错信息中。
- 接口名称始终出现在错误消息中，但只有在按名称使用时才会出现。（[例子](https://www.typescriptlang.org/play?#code/PTAEjR-RF6MCzVExUx76MBG2h7A0GA6hpzUBN+h-eUBSuhS40GPlQWBVAHzwC4AoSgSwDsAXAUwCcAzAQwGMnQBZDgFtBHADagA3pVAzQdIU3KgAzgxb0A5pQC+1NgFc6XBjQD2dUEy4ALUwOFiAFIKX2RogJSTpsrueWmokwAdKKmGs7B8oJMHjrUIJbBGsGggNBygEbpgCFuCKiA3AaA9GZ4ZPxC7qCUVrZuThJyCkoAjABMAMwALK2g2nGUiYBG+oBDyqCAOASCgLgEoFmA3j6A0eqAMSpEpQ7iM7OAMdqAFoqAAHKA7BZ9YIBi8qAAKgCeAA5MAMpc6hcMoICwcoBY8oAU6ggogLRygAvGgAxKy3cekMxjMFiqpgAgnQaO5nEo6tFFCo1Jpul4pLJQH46AEgqFwpEkXFdJUbFCYXDEQ1QC0OgBWdFAA)）
- 类型别名不能参与[声明合并](https://www.typescriptlang.org/play?#code/PTAEnvlRTuUB41DgzQSOUFRygYFULAqgQt0KXGhj5UDD-gseUG34wGQjAIFUDc9ALgChqBLAOwBcBTAJwDMBDAYxdACyXALbCuAG1ABvaqDmgA5iwYBXAM6VQapm0YLqAX1qNWnXvyGiJ02fIBGbFiwAmAfk3bdDfUeo8A9gzaoFwMdGLimpYRoAC8NvKKyuqaAEQAjABMAMwALKkANLZyIKCAjJqApLGA9GaAB2qAXHKgDk7OoICj+oDeGVWA6tqAZN6ATHLFjY4umumGtKWAsHKAvwFwSIAfbsjogN4+gNHqgNRKqLRMAJ4ADvwASix7THTi-PEyiUqqGlo6euPUuwegx6fnlwn2w24ej28hiAA)，但接口可以。
- 接口只能用于声明对象的形状，不能用于重命名原始类型（[例子](https://www.typescriptlang.org/play?#code/PTAEn95QKV0bx9Gj1RqJULAqgYlUKXGhj5UN3Kh540A-xgAOUCo5QT+1BDGIChyBLAOwBcBTAJwDMBDAYwdAEEaB5AEYArBhzoBGUAG9yoeaABubADYBXBgC5QAZzpNaAc3IBfSnQCeAB258ho8QCZQAXhlyFy9Vt36jpyhAIGAREQFnEwF8VREB85UBpzThyIOIMQFNzQBfowHozQFg5QHvlQBkIwBC3c2tuAGU2Gio6KgAvBgATUr8aQ1dfAxbySxtQAFFFBhoAOTUAW0FmNpoxiaZKWkZWTm4ADVAGAA9GGnqddqN3AKA)）。

大多数情况下，你可以根据个人喜好来选择是用接口还是类型别名，TypeScript 在必要的时候会告诉你需要使用其它类型的声明。如果你想要去探索，可以一直使用接口，直到你需要使用类型别名的功能。

## 类型断言
有时候你知道关于值的类型信息，但 TypeScript 不知道。

例如，如果你使用 `document.getElementById`，TypeScript 只知道将返回某种 `HTMLElement` 类型，但是你可能知道你的页面总是有一个带有给定 ID 的 `HTMLCanvasElement`，可以用类型断言：
```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```
与类型注释一样，类型断言也会被编译器删除，不会影响代码的运行时行为。

你也可以使用尖括号语法(在 `.tsx` 文件中不行)，这是等效的：
```ts
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```
>提醒：因为类型断言是在编译时删除的，所以没有与类型断言相关的运行时检查。如果类型断言错误，则不会生成异常或null。

TypeScript只允许类型断言为更具体或更宽松的类型。这条规则可以防止出现 “不可能的” 强制类型转换，例如:
```ts
const x = "hello" as number;
// 转换 "string" 类型为 "number" 类型可能是错误的，因为两种类型不能充分重叠。如果是故意的，请先将表达式转换为 "unknown" 再转 "number"
```
有时这条规则可能过于保守，可能不允许有效且更复杂的操作。那么你可以使用两个断言，首先指向 `any`（或 `unknow`，我们将在后面介绍），然后指向所需的类型:
```ts
const a = (expr as any) as T;
```
## 字面量类型
除了一般的字符串和数字类型，我们还可以在类型位置引用具体的字符串和数字。

其实我们只需要考虑 JavaScript 声明变量的不同方式即可。`var` 和 `let` 声明的变量都可以修改值，但 `const` 不行。这种特点反映在 TypeScript 为：
```ts
let changingString = "Hello World";

changingString = "Olá Mundo";
// 因为 'changingString' 可以表示任意字符串
// 在 TypeScript 类型系统中是这么描述它的
// let changingString: string

const constantString = "Hello World";
// 因为 'constantString' 只能表示一种字符串
// 在 TypeScrip 类型系统中为字面量类型表示
// const constantString: "Hello World"
```
字面量类型本身并不是很有用:
```ts
let x: "hello" = "hello";
// OK
x = "hello";

x = "howdy";
// Error：'"howdy"' 类型不能赋值给 '"hello"' 类型
```
只能有一个值的变量并没有多大用处！

但是将字面量类型结合为联合类型，可以表示一个更有实用价值的概念——例如，函数只接收一组特定的值：
```ts
function printText(s: string, alignment: "left" | "right" | "center") {
    // ...
}
printText("Hello, world", "left"); // OK

printText("G'day, mate", "centre");
// Error：'"centre"' 类型的实参不能赋值给 '"left" | "right" | "center"' 类型的形参
```
数值字面量类型也一样：
```ts
function compare(a: string, b: string): -1 | 0 | 1 {
    return a === b ? 0 : a > b ? 1 : -1;
}
```
当然，你可以将字面量类型与非字面量类型结合起来：
```ts
interface Options {
    width: number;
}
function configure(x: Options | "auto") {
    // ...
}
configure({ width: 100 }); // OK
configure("auto"); // OK

configure("automatic");
// Error：'"automatic"' 类型的实参不能赋值给 'Options | "auto"' 类型的行参
```
还有一种字面量类型：布尔字面量类型。布尔字面量类型只有两种，因为布尔类型只有`true` 和 `false` 两个值。布尔类型实际上只是 `true | false` 联合类型的别名。

### 字面量推断
当你用一个对象初始化一个变量时，TypeScript 假设该对象的属性值以后可能会改变。例如：
```ts
const obj = { counter: 0 };
if (someCondition) {
    obj.counter = 1;
}
```
TypeScript 并不认为将 `1` 赋值给之前为 `0` 的字段是错误。或者说 `obj.Counter` 为 `number` 类型，而不是字面量类型 `0`，因为类型决定读和写行为。  

同样适用于字符串:
```ts
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
// handleRequest 定义为：function handleRequest(url: string, method: "GET" | "POST"): void
// Error：'string' 类型的实参不能赋值给 '"GET" | "POST"' 类型的行参
```
在上面的例子中，`req.method` 被推断为字符串类型，而不是 `"GET"` 字面量类型。因为在创建 `req` 变量和调用 `handleRequest` 函数之间可能会执行其它代码。例如，在调用 `handleRequest` 函数前  `req.method` 也许会被赋值为类似 `"GUESS"` 这样的字符串。因此 TypeScript 认为这样的代码是错误的。

有两种方法可以解决这个问题。

1. 你可以在任意位置添加类型断言来改变推断：
    ```ts
    // Change 1:
    const req = { url: "https://example.com", method: "GET" as "GET" };

    // Change 2
    handleRequest(req.url, req.method as "GET");
    ```
    Change1 表示：打算把 `req.method` 始终设置为 `"GET"` 字面类型，防止字段被再次赋值。
    
    Change2 表示：在调用 `handleRequest` 函数的时候，告诉 TypeScript `req.method` 的值为 `"GET"`。
2. 你可以使用 `const` 将整个对象转换为字面量类型:
    ```ts
    const req = { url: "https://example.com", method: "GET" } as const;
    handleRequest(req.url, req.method);
    ```
    `as const` 后缀的作用类似于 `const`，但适用于类型系统，确保所有属性都被定义为字面量类型，而不是字符串或数字等更通用的版本。

## `null` 和 `undefined`
JavaScript 有两个基本值用来表示不存在或未初始化： `null` 和 `undefined`。

TypeScript 也有两个同名的对应类型。这些类型的行为取决于是否打开了 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 选项。

### `strictNullChecks` 关闭
关闭 strictNullChecks 后，可能为 `null` 或 `undefined` 的值仍然可以正常访问，并且 `null` 和 ` undefined` 的值可以分配给**对象任何类型的属性**。这类似于没有 null 检查的语言（如c#， Java）的行为。缺乏对这些值的检查往往是 bug 的主要来源；建议在代码库中使用严格的检查方法。
### `strictNullChecks` 开启
开启 strictNullChecks 后，当值为 `null` 或 `undefined` 时，需要在使用该值的方法或属性之前，测试这些值。就像在使用可选属性之前检查是否为 `undefined` 一样。例如，使用缩窄来检查可能为 `null` 的值：
```ts
function doSomething(x: string | null) {
    if (x === null) {
        // do nothing
    } else {
        console.log("Hello, " + x.toUpperCase());
    }
}
```
## 非空断言运算符（后缀 `!`）
TypeScript 还有一个特殊的语法，可以在不做任何显式检查的情况下，从类型中删除 `null` 和 `undefined`。在任何表达式之后添加类型断言 `!` 都是有效的，表明该值不是 `null` 或 `undefined`：
## 枚举
枚举是 TypeScript 添加到 JavaScript 的一个特性，它允许描述一个值，用一组命名常量的其中之一。与大多数 TypeScript 特性不同，枚举不是在类型层面添加到 JavaScript 中的，而是添加到了语言本身和运行时的。正因为如此，这是一个你应该知道的功能。你可以在 [Enum reference page](https://www.typescriptlang.org/docs/handbook/enums.html) 中阅读更多关于枚举的信息。
## 不常用原始类型
在 JavaScript 中的其它原始类型，TypeScript 中也有对应的表示形式。在这里简单介绍下，不会深入讨论。
### bigInt
从 ES2020 版本开始，JavaScript 中有一个用来表示非常大的整数的原始类型，`BigInt`：
```ts
// 通过 BigInt 函数创建一个 bigint
const oneHundred: bigint = BigInt(100);

// 通过字面量语法创建一个 bigint
const anotherHundred: bigint = 100n;
```
你可以在 [TypeScript 3.2 发布说明](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-2.html#bigint) 了解关于 BigInt 的更多信息。
### symbol
JavaScript 中，通过函数 `Symbol()` 创建一个全局唯一引用的原始类型：
```ts
const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
    // Error：这个条件总是返回'false'，因为类型 'typeof firstName' 和类型 'typeof secondName' 没有重叠。
}
```
你可以在 [Symbols 参考页](https://www.typescriptlang.org/docs/handbook/symbols.html) 了解关于 Symbol 的更多信息。


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/2/everyday-types.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
