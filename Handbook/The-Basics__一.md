---
highlight: vs2015
---
# The Basics

欢迎来到手册第一页，如果这是你第一次使用 TypeScript - 你可以从 ["入门"](https://www.typescriptlang.org/docs/handbook/intro.html#get-started) 指南开始。

在 JavaScript 中每个值都有一系列行为，你可以通过运行不同的操作进行观察。听起来很抽象，举个简单的例子，假设我们可能在一个名为 `message` 的变量上运行这些操作：
```ts
  // 访问 'message' 上的 'toLowerCase' 属性并调用
  message.toLowerCase();
  // 调用 'message'
  message();
```
第一行可运行的代码访问 `message` 上的 `toLowerCase` 属性，然后调用它。第二行尝试直接调用 `message` 。

但是，假设我们不知道 `message` 的值 ( 这很常见 )，那么我们就不能保证运行这些代码能得到什么结果。JavaScript 每个操作的行为完全取决于我们起初得到的值。

* `message` 是否可调用 ?
* 它是否有一个叫 `toLowerCase` 的属性 ?
* 如果有，`toLowerCase` 是否可调用 ?
* 如果这两个值都可以是可调用的，那它们返回什么 ?

这些问题的答案通常是我们在编写 JavaScript 的时候牢记于心的。我们只能希望所有细节都是正确的。

假设 `message` 是按照以下方法定义的。
```ts
const message = "Hello World!";
```

你可能已经猜到，如果我们尝试运行 `message.toLowerCase()`，将会得到只有小写的 "hello world!"。

那第二行代码呢？如果你熟悉 JavaScript，知道这会抛出异常：
```ts
TypeError: message is not a function
```
当我们运行代码的时候，JavaScript 运行时是通过确认值的类型而决定它有什么类型的行为和能力。这就是 `TypeError` 报错提示的内容（字符串 `"Hello World!"` 不能作为函数调用）。

js 中数据类型可通过 `typeof`，`instanceof` 等标识符识别类型，再执行相应的操作，但是这样就显得非常麻烦。例子：
```ts
function fn(x) {
  return x.flip();
}
```
这个函数参数得是一个对象，并且有一个可调用的 `filp` 属性，才能正常运行。但是 JavaScript 并没有在运行时检查并显示这些信息。在 JavaScript 中，要知道 `fn` 对特定值做了什么，就是调用看看会发生什么。这样的特征使得我们很难在代码执行前进行相关的预测。我们编写代码时必须预测 `fn` 函数调用后的多种可能，并用控制流处理：
```ts
function fn(x) {
  if(x.flip && typeof x.flip === 'function'){
    return x.flip(); 
  }
}
```
从例子这个角度看，类型就是描述了哪些值可以传递给 `fn` ，哪些值会抛出异常。JavaScript 只有提供了动态类型——运行代码才知道会发生什么。

TS 使用了静态类型系统，在运行代码之前可以对期望的代码进行预测。

## 静态类型检查
之前我们假设 `message` 为 `string` 时，尝试把它当函数调用，得到了 `TypeError`。大多数开发者不喜欢运行他们代码的时候得到任何报错——他们是 bug，我们应该尽量避免。

如果我们在此基础上添加一点代码，保存文件，重新运行，抛出错误。我们也许就能马上找到问题所在，从而解决。但如果我们代码很多，并且没有充分的测试该功能，我们可能永远不会遇到，抛出潜在的错误。或者我们有幸目睹了这个错误，我们可能会做大量的重构和添加许多不同的代码，被迫深入研究。

理想情况下，我们得有一个工具来帮我们找到这些 bug，这就是 TypeScript 静态类型检查器所做的。静态类型检查系统描述了当我们程序运行时，值的形状和行为。并利用这些信息并告诉我们什么时候事情可能会抛出异常。
```ts
const message = "hello!";
message();

// This expression is not callable.
// Type 'String' has no call signatures.
```
用 TypeScript 编写上面的示例，会在我们运行代码之前给我们一个错误提示。

## 非异常报错
目前为止，我们一直在讨论运行时错误——JavaScript 运行时会告诉我们它认为荒谬的情况。出现这种情况是因为 [ECMAScript 规范](https://tc39.es/ecma262/) 明确说明，语言在遇到意外情况时如何表现。

例如，规范说尝试调用某些不可调用的东西时，应该抛出错误。这听起来像是"显而易见的行为"，那如果访问一个对象上不存在的属性呢？也应该抛出错误吧。然而，JavaScript 给了我们不同的行为，返回了 undefined 值。
```ts
const user = {
  name: "Daniel",
  age: 26,
};
user.location; // undefined
```
而静态类型系统会标志为错误的代码，即使他是不会抛出错误的 "有效 JavaScript"。下面代码会产生一个关于 `location` 没有定义的错误：
```ts
const user = {
  name: "Daniel",
  age: 26,
};
 
user.location;
// Property 'location' does not exist on type '{ name: string; age: number; }'.
```
虽然有时得在可以表达的内容上进行让步（无该属性时，不能再表达 undefined 值），但目的是在我们的程序中捕获合法的错误。TypeScript 捕获了很多合法的错误。例如：拼写错误：
```ts
const announcement = "Hello World!";

// 你能多快发现错别字?
announcement.toLocaleLowercase();
announcement.toLocalLowerCase();

// 我们可能想这么写...
announcement.toLocaleLowerCase();
```
或未调用的函数：
```ts
function flipCoin() {
  // 忘记调用 Math.random()
  return Math.random < 0.5;
Operator '<' cannot be applied to types '() => number' and 'number'.
}
```
或基本逻辑错误：
```ts
const value = Math.random() < 0.5 ? "a" : "b";
if (value !== "a") {
  // ...
} else if (value === "b") {
  // 永远不会到达
}
```
## 类型工具
当我们代码错误时 TypeScript 能够捕获，而且 TypeScript 还能在一开始就阻止我们犯错。  
类型检查器会去检查我们是否访问变量的正确属性，和其它属性。它还可以开始建议你可能想要使用哪些属性。

这意味着 TypeScript 也能够影响到编辑代码，类型检查器可以在您编辑器中键入，提供错误消息和代码补全。

![1678159799726.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e315bd2ab7e647ebaf8254c74ddcc0bf~tplv-k3u1fbpfcp-watermark.image?)

TypeScript 非常重视工具的使用，而不仅仅是提供错误信息和输入时的补全。支持 TypeScript 的编辑器可能提供 “快速修复” 功能来自动修复错误，重构功能来轻松地重新组织代码，还提供了导航功能来跳转到变量的定义，或者找到给定变量的所有引用。所有这些都构建在类型检查器之上，并且是完全跨平台的，你最喜欢的编辑器可能有 [TypeScript 支持](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support)。

## `tsc`——TypeScript 编译器
我们已经讨论了类型检查器，但是我们到目前为止还没有使用它。让我们来认识新的朋友 `tsc` —— TypeScript 编译器，首先我们需要通过 npm 获取它：
```ts
npm install -g typescript
```
这是全局安装 TypeScript 编辑器 tsc。如果你更喜欢从本地 node_modules 包运行 tsc，你可以使用 npx 或者类似的工具。

现在让我们新建个空的文件夹，创建 `hello.ts` 文件，并尝试编写我们的第一个 TypeScript 程序：
```ts
// Greets the world.
console.log("Hello world!");
```
注意这些这里没有任何装饰，这个 "hello world" 程序看起来与用 JavaScript 编写的 "hello world" 程序相同。现在让我们通过运行 `tsc` 命令（刚刚安装的 typescript 包），对其进行类型检查和编译。
```ts
tsc hello.ts
```
我们运行 `tsc` 什么都没发生！嗯，也没有报错，所以我们控制台没有任何输出，因为没有什么可以报告的。  
如果我们查看当前目录，会发现 `hello.ts` 旁边多了个 `hello.js` 文件。这是我们运行 `tsc` 命令后编译或者转换 `hello.ts` 文件而输出的普通 JavaScript 文件。如果我们检查其内容，我们将看到 TypeScript 在编译 .ts 文件后输出的内容：
```js
// Greets the world.
console.log("Hello world!");
```
在这种情况下，TypeScript 需要转换的东西非常少，所以它看起来和我们写的完全相同。编译器试图产出干净的代码，使之看起来像开发者编写的一样。这并不容易，TypeScript 编译会保持缩进，换行，并尽量保留注释。

让我们重写 `hello.ts`，刻意让类型检查报错:
```ts
// 这是一个问候工厂函数
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date}!`);
}
 
greet("Brendan");
```

如果我们再次运行 `tsc hello.ts`，注意，我们在命令行会得到一个错误！
```ts
Expected 2 arguments, but got 1.
```
TypeScript 告诉我们少传了一个参数给 `greent` 函数，我们只是编写了普通的 JavaScript，而类型检查仍然能够发现我们代码中的问题。

## 就算报错仍然编译
在上一个例子中你可能没有注意到一件事，我们的 `hello.js` 文件再次改变了。如果我们打开那个文件，我们会看到内容仍然和我们的 `hello.ts` 文件是一样的。`tsc` 报告了有关我们代码的错误，竟然还是编译更新了。但这是 TypeScript 的核心之一：就算有报错信息，TypeScript 也不阻碍你编译运行。

因为在某些情况下，这些检查会成为阻碍。例如，你将 JavaScript 代码迁移到 TypeScript，并引入了类型检查器检查错误。就得因类型检查器检查而清理一些东西。但是，原始的 JavaScript 代码已经可以工作了，转为 TypeScript 反而阻碍你运行。

所以 TypeScript 不会妨碍您。当然，随着时间的推移，你可能希望更能防范错误，并使 TypeScript 的行为更加严格。你可以使用 [noEmitOnError](https://www.typescriptlang.org/tsconfig#noEmitOnError) 编译器选项，在有错误信息时停止编译。
```ts
tsc --noEmitOnError hello.ts
```
现在尝试改变 `hello.ts` 文件代码，并使用以上指令：你会注意到有报错信息时 `hello.js` 不会更新了。
## 显式类型
直到现在我们还没有告诉 TypeScript `person` 或者 `date` 是什么类型。让我们编写代码，告诉 TypeScript `person` 是 `string`，`date` 应该是 `Date` 对象。并在 `date` 上使用 `toDateString()` 方法。
```ts
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
```
我们在 `person` 和 `date` 上添加了类型注释，描述 `greet` 函数接收 `person` 为 `string` 类型，`date` 为 `Date` 类型。

有了类型注释，TypeScript 就可以告诉我们 `greet` 可能被错误调用的其他情况。例如：
```ts
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
 
greet("Maddison", Date());
// Argument of type 'string' is not assignable to parameter of type 'Date'.
```
TypeScript 在我们第二个参数上报告了一个错误，这是为什么？

因为在 JavaScript 中调用 `Date()` 返回一个字符串。实际上我们所期望的是用 `new Dta()` 实例化一个 `Date` 对象。

不管怎样，我们都可以因为错误信息快速修复这个错误:
```ts
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
 
greet("Maddison", new Date());
```
记住，我们不用总编写显式的类型注释，大多数情况下，TypeScript 可以为我们推断（或"计算出"）类型。

![1678169427098.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3f0ec6ef16e485d88f11b8d3f5a880f~tplv-k3u1fbpfcp-watermark.image?)

虽然我们没有告诉 TypeScript `msg` 是 `string` 类型，但它也能推断出来，这是一个特性。类型系统能推断出的类型就没必要再添加显示类型注释。

>注意：代码示例中的消息气泡是您将鼠标悬停在该变量上时编辑器显示的内容。
## 抹除类型
让我们看看用 `tsc` 编译上面的 `greet` 函数，会输出什么 JavaScript：
```ts
"use strict";
function greet(person, date) {
    console.log("Hello ".concat(person, ", today is ").concat(date.toDateString(), "!"));
}
greet("Maddison", new Date());
```
注意两点：
1. 我们的 `person` 和 `date` 参数没有了类型注释。
2. 我们的 "字符串模板" —— 反引号字符 ( \` ) —— 被转换为 `concat` 方法连接字符串。

稍后会详细介绍第二点，但现在让我们关注第一点。类型注释不是 JavaScript （或者 ECMAScript）的一部分，没有任何浏览器或其它环境能直接运行运行 TypeScript。这就是 TypeScript 需要编译器的原因——剥离或转换 TypeScript 为 JavaScript 代码。所以，我们的类型注释被完全擦除了。

>记住：类型注释永远不会改变你程序的运行时行为。
## 降级
与类型注释不同的是，我们的模板字符串是从
```js
`Hello ${person}, today is ${date.toDateString()}!`
```
变成
```js
"Hello " + person + ", today is " + date.toDateString() + "!";
```
为什么会这样？

因为，模板字符串是 ECMAScript 2015 版本（又叫 ECMAScript 6，ES2015，ES6，等等）的特性。从较新或 “更高” 版本的 ECMAScript 向下移动到较旧或 “较低” 版本的过程有时称为降级。

默认情况下 TypeScript 编译为 JavaScript 的版本为 ES3 版本，ECMAScript 一个非常的旧的版本，我们可以使用 [target](https://www.typescriptlang.org/tsconfig#target) 选项来选择更高的版本。运行 `--target es2015` 改变 TypeScript 编译目标为 ECMAScript 2015，意味着代码能够在支持 ECMAScript 2015 的任何地方运行。所以运行 `tsc --target es2015 hello.ts` 代码编译为：
```ts
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
greet("Maddison", new Date());
```
虽然默认版本是 ES3，但当前浏览器绝大多数都支持 ES2015。因此，大多数开发人员可以安全地指定 ES2015 或更高版本作为目标，除非要与某些古老浏览器做兼容。
## 严格模式
不同的用户来到 TypeScript ，在使用在类型检查器时，一些用户寻求更加宽松的开发体验，他们希望类型检查仅作用于部分代码，同时还可享受 TypeScript 提供的功能。这是 TypeScript 的默认体验，类型是可选的，推断采用最宽松的类型，并且不检查潜在的 `null/undefined` 值。就像 `tsc` 在面对错误时仍然能编译文件一样，避免影响你的工作。如果您正在迁移现有的 JavaScript，这可能是理想的第一步。

相比之下，很多用户更喜欢让 TypeScript 直接进行尽可能多的验证，这就是为什么该语言也提供了严格设置。TypeScript 为你检查得越多，你就可能需要做一些额外的工作，但从长远看这是值得的，并且支持更彻底的检查和更精确的工具。在适合的情况下，新的代码库应该始终启用这些严格性检查。

TypeScript 有几个可以打开或关闭的严格类型检查标志，我们所有的示例都将启用这些标志，除非另有说明。CLI 中的 [strict](https://www.typescriptlang.org/tsconfig#strict) 标志 或者 [tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) 中的 `"strict": true`，可以将这些标志全部打开，我们也可以单独使用它们。你应该了解的两个最大的检查是 [noImplicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) 和 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks)。

## noImplicitAny
回想一下，在一些地方，TypeScript 不会尝试为我们推断类型，而是退回到最宽松的类型：`any`，这是很糟糕的事情——毕竟，回退到 `any` 只是普通的 JavaScript 体验。
 
无论如何，经常使用 `any` 违背了使用 TypeScript 的本意。你的程序有更多类型，将会有更多的验证和工具，意味着你编写代码时会遇到更少的 bug，打开 [noImplicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) 标志，任何隐性推断为 any 的变量将会报错。
## strictNullChecks
默认情况下，像 `null`  和 `undefined` 能分配给其它任何类型。这使得写代码更简单，但是有时候忘记去处理 `null` 和 `undefined` 是造成无数的 bug 的原因——考虑到这是重要的错误！[strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 标志会在值为 `null` 和 `undefined` 类型的时候进行信息提示，使我们不必担心是否忘记处理 `null` 和 `undefined` 。

感谢观看，如有错误，望指正

>官网地址： <https://www.typescriptlang.org/docs/handbook/2/basic-types.html>
>
>github 资料： <https://github.com/Mario-Marion/TS-Handbook>

