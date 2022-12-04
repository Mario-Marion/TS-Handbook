# The Basics
Welcome to the first page of the handbook. If this is your first experience with TypeScript - you may want to start at one of the [Getting Started](https://www.typescriptlang.org/docs/handbook/intro.html#get-started) guides

Each and every value in JavaScript has a set of behaviors you can observe from running different operations. That sounds abstract, but as a quick example, consider some operations we might run on a variable named `message`.

欢迎来到手册第一页，如果这是你第一次使用 TypeScript - 你可以从"入门"指南开始。  
在 JavaScript 中每个值都有一系列行为，你可以通过运行不同的操作进行观察。听起来很抽象，举个简单的例子，考虑一些我们可能在一个名为 `message` 的变量上运行的操作。
```ts
  // Accessing the property 'toLowerCase'
  // on 'message' and then calling it
  message.toLowerCase();
  // Calling 'message'
  message();
```
If we break this down, the first runnable line of code accesses a property called `toLowerCase` and then calls it. The second one tries to call `message` directly.

But assuming we don’t know the value of `message` - and that’s pretty common - we can’t reliably say what results we’ll get from trying to run any of this code. The behavior of each operation depends entirely on what value we had in the first place.

如果我们分析下，第一行可运行的代码访问一个叫做 `toLowerCase` 的属性，然后调用它。第二行尝试直接调用 `message` 。  
但是，假设我们不知道 message 的值 ( 这很常见 )，我们不能可靠的说，尝试运行这些代码能得到什么结果。每个操作的行为完全却决于我们起初得到的值。

* Is `message` callable ?
* Does it have a property called `toLowerCase` on it ?
* if it does,is `toLowerCase` even callable ?
* if both of these values are callable,that do they return ?
* `message` 是否可调用 ?
* 它是否有一个叫 `toLowerCase` 的属性 ?
* 如果有，`toLowerCase` 是否可调用 ?
* 如果这两个值都可以是可调用的，那它们返回什么 ?

The answers to these questions are usually things we keep in our heads when we write JavaScript, and we have to hope we got all the details right.

Let’s say `message` was defined in the following way.

这些问题的答案通常是我们在编写 JavaScript 的时候牢记于心的。我们希望我们把所有细节弄对。  
假设 `message` 是按照以下方法定义的。
```ts
const message = "Hello World!";
```
As you can probably guess,if we try to run `message.toLowerCase()`,we'll get the same string only in lower-case.

What about that second line of code?if you're familiar with JavaScript,you'll know this fails with an exception.

正如你可能猜到的，如果我们尝试运行`message.toLowerCase()`，我们将会得到相同的字符串，只是小写的。  
那第二行代码呢？如果你熟悉 JavaScript，你会知道这个失败的异常：
```ts
TypeError: message is not a function
```
it'd be great if we could avoid mistakes like this.

When we run our code, the way that our JavaScript runtime chooses what to do is by figuring out the type of the value - what sorts of behaviors and capabilities it has. That’s part of what that `TypeError` is alluding to - it’s saying that the string `"Hello World!"` cannot be called as a function.

如果我们能够避免这样的错误就太好了。  
当我们运行代码的时候，JavaScript 运行时选择做什么的方式是通过确认值的类型（它有什么类型的行为和能力）。这就是`TypeError` 暗指的部分（它表示字符串 `"Hello World!"` 不能作为函数调用）。  

For some values, such as the primitives `string` and `number`, we can identify their type at runtime using the `typeof` operator. But for other things like functions, there’s no corresponding runtime mechanism to identify their types. For example, consider this function:

对于某些值，比如基本类型 `string` 和 `number`，我们可以在运行时使用 `typeof` 操作符识别它们的类型。但是对于像函数这样的其它的类型，没有相应的的运行机制来识别类型。例如，思考这个函数：
```ts
function fn(x) {
  return x.flip();
}
```
We can observe by reading the code that this function will only work if given an object with a callable flip property, but JavaScript doesn’t surface this information in a way that we can check while the code is running. The only way in pure JavaScript to tell what fn does with a particular value is to call it and see what happens. This kind of behavior makes it hard to predict what code will do before it runs, which means it’s harder to know what your code is going to do while you’re writing it.

我们阅读这个代码观察到，这个函数只有在给予了一个有可调用的 `filp` 属性的时候才能工作。但是 JavaScript 并没有可以在运行时检查的方式显示这些信息。在纯粹的 JavaScript 中，要知道 `fn` 对特定值做了什么,唯一的方法是调用看看会发生什么。这种行为使很难预测代码运行前会做什么，这意味着你编写代码的时候，很难知道你的代码将要做什么。

Seen in this way, a type is the concept of describing which values can be passed to `fn` and which will crash. JavaScript only truly provides dynamic typing - running the code to see what happens.

The alternative is to use a static type system to make predictions about what code is expected before it runs.

从这个角度看，类型是描述了哪些值可以传递给 `fn` ，哪些值会崩溃的概念。JavaScript 只有提供了动态类型-运行代码看看会发生什么。  
另一种方法是使用静态类型系统，在运行代码之前对期望的代码进行预测。  

## Static type-checking

Think back to that `TypeError` we got earlier from trying to call a `string` as a function.Most people don't like to get any sorts of errors when running their code - those are considered bugs!And when we write new code,we try our best to avoid introducing new bugs.

If we add just a bit of code, save our file, re-run the code, and immediately see the error, we might be able to isolate the problem quickly; but that’s not always the case. We might not have tested the feature thoroughly enough, so we might never actually run into a potential error that would be thrown! Or if we were lucky enough to witness the error, we might have ended up doing large refactorings and adding a lot of different code that we’re forced to dig through.

Ideally, we could have a tool that helps us find these bugs before our code runs. That’s what a static type-checker like TypeScript does. Static types systems describe the shapes and behaviors of what our values will be when we run our programs. A type-checker like TypeScript uses that information and tells us when things might be going off the rails.

回想一下之前我们尝试把 `string` 当函数调用时，得到的 `TypeError`。大多数人们不喜欢运行他们代码的时候得到任何报错-他们是bug，我们尽量避免引入新的bug。  
如果我们只添加一点代码，保存文件，重新运行，立马就能看到错误。我们也许能马上找到问题所在。但情况并非总是这样。我们可能没有完全充分的测试该功能，所以我们可能永远不会遇到,抛出,潜在的错误。或者我们足够幸运的目睹这个错误，我们可能会做大量的重构和添加许多不同的代码，被迫深入研究。  
理想情况下，我们可以有一个工具来帮我们找到这些bug,这就像 TypeScript 静态类型检查器所做的。静态类型检查系统描述了当我们运行程序时，值的形状和行为。像 TypeScript 这样的类型检查器使用该信息并告诉我们什么时候事情可能会偏离轨道。
```ts

const message = "hello!";
message();

This expression is not callable.
Type 'String' has no call signatures.

```
Running that last sample with TypeScript will give us an error message before we run the code in the first place.

用TypeScript运行最后一个示例，会在我们运行代码之前给我们一个错误消息。

## Non-exception Failures

So far we’ve been discussing certain things like runtime errors - cases where the JavaScript runtime tells us that it thinks something is nonsensical. Those cases come up because [the ECMAScript specification](https://tc39.es/ecma262/) has explicit instructions on how the language should behave when it runs into something unexpected.

For example, the specification says that trying to call something that isn’t callable should throw an error. Maybe that sounds like “obvious behavior”, but you could imagine that accessing a property that doesn’t exist on an object should throw an error too. Instead, JavaScript gives us different behavior and returns the value undefined:

目前为止，我们一直讨论某些事情，比如运行时错误-JavaScript 运行时告诉我们它认为荒谬的情况。出现这种情况是因为 ECMAScript 规范语言在遇到意外情况时，如何表现有明确说明。

例如，规范说尝试调用某些不可调用的东西时，应该抛出错误。这听起来像是"显而易见的行为"，但你可以想象访问一个不存在对象上的属性也应该抛出错误。相反，JavaScript 会给我们不同的行为，并返回 undefined 值。
```ts
const user = {
  name: "Daniel",
  age: 26,
};
user.location; // returns undefined
```
最终，静态类型系统必须调用应该在其系统中标志为错误的代码，即使他是不会立即抛出错误的"有效 JavaScript"，在 TypeScript 中，下面代码会产生一个关于 `location` 没有定义的错误。
```ts
const user = {
  name: "Daniel",
  age: 26,
};
 
user.location;
Property 'location' does not exist on type '{ name: string; age: number; }'.
```
While sometimes that implies a trade-off in what you can express, the intent is to catch legitimate bugs in our programs. And TypeScript catches a lot of legitimate bugs.

For example: typos,

虽然有时这意味着在可以表达的内容上进行让步，但目的是在我们的程序中捕获合法的错误。 TypeScript 捕获了很多合法的错误。
例如:拼写错误，
```ts
const announcement = "Hello World!";

// How quickly can you spot the typos?
announcement.toLocaleLowercase();
announcement.toLocalLowerCase();

// We probably meant to write this...
announcement.toLocaleLowerCase();
```
uncalled functions,
未调用的函数
```ts
function flipCoin() {
  // Meant to be Math.random()
  return Math.random < 0.5;
Operator '<' cannot be applied to types '() => number' and 'number'.
}
```
or basic logic error.
或基本逻辑错误
```ts
const value = Math.random() < 0.5 ? "a" : "b";
if (value !== "a") {
  // ...
} else if (value === "b") {
This condition will always return 'false' since the types '"a"' and '"b"' have no overlap.
  // Oops, unreachable
}
```

## Types for Tooling

TypeScript can catch bugs when we make mistakes in our code. That’s great, but TypeScript can also prevent us from making those mistakes in the first place.

The type-checker has information to check things like whether we’re accessing the right properties on variables and other properties. Once it has that information, it can also start suggesting which properties you might want to use.

That means TypeScript can be leveraged for editing code too, and the core type-checker can provide error messages and code completion as you type in the editor. That’s part of what people often refer to when they talk about tooling in TypeScript.

当我们代码错误时 TypeScript 能够捕获，这很好，而且 TypeScript 还能在一开始就阻止我们犯错。  
类型检查器有信息去检查诸如，我们是否访问变量的正确属性，和其它属性。一旦它获得信息，它还可以开始建议你可能想要使用哪些属性。  
这意味着 TypeScript 也能够影响到编辑代码，核心类型检查器可以在您编辑器中键入时提供错误消息和代码完成。这是人们在谈论关于 TypeScript 中的工具时经常提到的部分。

```ts
import express from "express";
const app = express();
 
app.get("/", function (req, res) {
  res.sen
      * send
      * sendDate
      * sendfile
      * sendFile
      * sendStatus
});
 
app.listen(3000);
```
TypeScript takes tooling seriously, and that goes beyond completions and errors as you type. An editor that supports TypeScript can deliver “quick fixes” to automatically fix errors, refactorings to easily re-organize code, and useful navigation features for jumping to definitions of a variable, or finding all references to a given variable. All of this is built on top of the type-checker and is fully cross-platform, so it’s likely that [your favorite editor has TypeScript support available.](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support)

TypeScript非常重视工具的使用，而不仅仅是输入时的补全和错误。支持TypeScript的编辑器可以提供“快速修复”功能来自动修复错误，可以提供重构功能来轻松地重新组织代码，还提供有用的导航功能来跳转到变量的定义，或者找到给定变量的所有引用。所有这些都构建在类型检查器之上，并且是完全跨平台的，所以很可能你最喜欢的编辑器都有TypeScript支持。  

## `tsc`, the TypeScript compiler

We’ve been talking about type-checking, but we haven’t yet used our type-checker. Let’s get acquainted with our new friend tsc, the TypeScript compiler. First we’ll need to grab it via npm.

我们已经讨论了类型检查器，但是我们到目前为止还没有使用它。让我们来认识新的朋友 `tsc`，TypeScript 编辑器，首先我们需要通过 npm 抓取它

```ts
npm install -g typescript
```
`This installs the TypeScript Compiler tsc globally. You can use npx or similar tools if you’d prefer to run tsc from a local node_modules package instead.`

Now let’s move to an empty folder and try writing our first TypeScript program: hello.ts:

这是全局安装 TypeScript 编辑器 tsc。如果你反而更喜欢从本地 node_modules 包运行 tsc，你可以使用 npx 或者类似的工具。  
现在让我们移动到一个空的文件夹，并尝试编写我们的第一个 TypeScript 程序：hello.ts
```ts
// Greets the world.
console.log("Hello world!");
```
Notice there are no frills here; this “hello world” program looks identical to what you’d write for a “hello world” program in JavaScript. And now let’s type-check it by running the command tsc which was installed for us by the typescript package.

注意这些这里没有任何装饰；这个 "hello world" 程序看起来与用 JavaScript 编写的 "hello world" 程序相同。现在让我们通过运行由 typescript 包为我们安装的命令 tsc ，对其进行类型检查。

```ts
tsc hello.ts
```
Tada!
Wait,"tada" what exactly?We ran `tsc` and nothing happened!Well,there were no type errors,so we didn't get any output in our console since there was nothing to report.

But check again - we got some file output instead. If we look in our current directory, we’ll see a `hello.js` file next to `hello.ts`. That’s the output from our `hello.ts` file after `tsc` compiles or transforms it into a plain JavaScript file. And if we check the contents, we’ll see what TypeScript spits out after it processes a `.ts` file:

等等，到底什么是 "tada"？我们运行 `tsc` 什么都没发生！嗯，也没有报错，所以我们控制台没有任何输出，因为没有什么可以报告的。  
但是再次检查-我们得到了一些文件输出。如果我们查看当前目录，会看到 `hello.js` 文件在 `hello.ts` 旁边。这是我们 `hello.ts` 文件 `tsc` 编译或者转换后输出的普通 JavaScript 文件。如果我们检查其内容，我们将看到 TypeScript 在处理 .ts 文件后输出的内容：
```js
// Greets the world.
console.log("Hello world!");
```
In this case, there was very little for TypeScript to transform, so it looks identical to what we wrote. The compiler tries to emit clean readable code that looks like something a person would write. While that’s not always so easy, TypeScript indents consistently, is mindful of when our code spans across different lines of code, and tries to keep comments around.

What about if we did introduce a type-checking error? Let’s rewrite `hello.ts`:

在这种情况下，TypeScript 需要转换的东西非常少，所以它看起来和我们写的完全相同。编译器尝试发出干净可读的代码，看起来像人会写的东西。但这并不总是那么容易，TypeScript 是一致缩进的，他留意我们的代码什么时候跨越了不同代码行，并尽量保留注释。  
如果我们引入了类型检查错误呢？让我们重写 `hello.ts`:
```ts
// This is an industrial-grade general-purpose greeter function:
// 这是一个问候工厂函数
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date}!`);
}
 
greet("Brendan");
```
If we run `tsc hello.ts` agin,notice that we get an error on the command line!

如果我们再次运行 `tsc hello.ts`，注意，我们在命令行会得到一个错误！
```ts
Expected 2 arguments, but got 1.
```
TypeScript is telling us we forgot to pass an argument to the greet function, and rightfully so. So far we’ve only written standard JavaScript, and yet type-checking was still able to find problems with our code. Thanks TypeScript!

TypeScript 告诉我们忘记传递一个参数给 `greent` 函数，这是理所当然的，至今我们只是编写了普通的JavaScript，然而类型检查仍然能够发现我们代码中的问题。

## Emitting with Errors

One thing you might not have noticed from the last example was that our `hello.js` file changed again. If we open that file up then we’ll see that the contents still basically look the same as our input file. That might be a bit surprising given the fact that `tsc` reported an error about our code, but this is based on one of TypeScript’s core values: much of the time, you will know better than TypeScript.

To reiterate from earlier, type-checking code limits the sorts of programs you can run, and so there’s a tradeoff on what sorts of things a type-checker finds acceptable. Most of the time that’s okay, but there are scenarios where those checks get in the way. For example, imagine yourself migrating JavaScript code over to TypeScript and introducing type-checking errors. Eventually you’ll get around to cleaning things up for the type-checker, but that original JavaScript code was already working! Why should converting it over to TypeScript stop you from running it?

So TypeScript doesn’t get in your way. Of course, over time, you may want to be a bit more defensive against mistakes, and make TypeScript act a bit more strictly. In that case, you can use the [noEmitOnError](https://www.typescriptlang.org/tsconfig#noEmitOnError) compiler option. Try changing your `hello.ts` file and running `tsc` with that flag:

在上一个例子中你可能没有注意到一件事，我们的 `hello.js` 文件再次改变了。如果我们打开那个文件，我们会看到内容基本上仍然和我们的输入文件是一样的。考虑到 `tsc` 报告了有关我们代码的错误，这可能有点令人惊讶。但这是基于 TypeScript 的核心之一：很多时候，你会比 TypeScript 更清楚。  
重申一下之前的，类型检查代码限制了你可以运行的程序的种类，因此要权衡类型检查器可以接受的内容的种类。多数情况下，这没问题，在某些情况下，这些检查会成为阻碍。例如，设想你自己将 JavaScript 代码迁移到 TypeScript，并引入了类型检查错误。最终你会因为类型检查器而清理一些东西，但原始的 JavaScript 代码已经可以工作了!为什么把它转换成TypeScript 会阻止你运行它？  
TypeScript 不会妨碍您。当然，随着时间的推移，你可能希望更能防范错误，并使 TypeScript 的行为更加严格。你可以使用 `noEmitOnError` 编译器选项。尝试更改你的 `hello.ts` 文件并使用 `tsc` 和标志运行：
```ts
tsc --noEmitOnError hello.ts
```
You’ll notice that `hello.js` never gets updated.

你会注意到 `hello.js` 永远不会更新了。
## Explicit Types

Up until now, we haven’t told TypeScript what `person` or `date` are. Let’s edit the code to tell TypeScript that `person` is a `string`, and that `date` should be a `Date` object. We’ll also use the `toDateString()` method on `date`.

开始到现在，我们还没有告诉 TypeScript `person` 或者 `date` 是什么。让我们编辑代码，告诉 TypeScript `person` 是 `string`，`date` 应该是 `Date` 对象。我们将还要在 `date` 上使用 `toDateString()` 方法。
```ts
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
```
What we did was add type annotations on `person` and `date` to describe what types of values `greet` can be called with. You can read that signature as "`greet` takes a `person` of type `string`, and a `date` of type `Date`".

With this, TypeScript can tell us about other cases where `greet` might have been called incorrectly. For example…

我们在 `person` 和 `date` 上添加类型注释，描述调用 `greet` 可以用什么类型的值。你可以把该签名解读为 `greet` 取 `person` 为 `string` 类型，`date` 为 `Date` 类型。  
有了这个，TypeScript 可以告诉我们 `greet` 可能被错误调用的其他情况。例如…
```ts
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
 
greet("Maddison", Date());
`Argument of type 'string' is not assignable to parameter of type 'Date'.`
```
Huh? TypeScript reported an error on our second argument, but why?

Perhaps surprisingly, calling `Date()` in JavaScript returns a `string`. On the other hand, constructing a `Date` with `new Date()` actually gives us what we were expecting.

Anyway, we can quickly fix up the error:

嗯？TypeScript 在我们第二个参数上报告了一个错误，但是为什么？
可能令人惊讶的是，在 JavaScript 中调用 `Date()` 返回一个 `string`。另一方面，用 `new Dta()` 构造一个 `Date` 实际上给了我们我们所期望的。
不管怎样，我们可以快速修复这个错误:
```ts
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
 
greet("Maddison", new Date());
```
Keep in mind, we don’t always have to write explicit type annotations. In many cases, TypeScript can even just infer (or "figure out") the types for us even if we omit them.

记住，我们不用总写明确的类型注释，大多数情况下，TypeScript 甚至可以为我们推断（或"计算出"）类型，即使我们忽略了它们也是如此。
```ts
let msg = "hello there!";
    
    * let msg: string
```
Even though we didn’t tell TypeScript that `msg` had the type `string` it was able to figure that out. That’s a feature, and it’s best not to add annotations when the type system would end up inferring the same type anyway.

`Note: The message bubble inside the previous code sample is what your editor would show if you had hovered over the word.`

虽然我们没有告诉 TypeScript `msg` 是 `string` 类型，它也能计算出来。这是一个特性，类型系统最终推断相同的类型时，最好不要添加注释。  
注意：前面代码示例中的消息气泡是您将鼠标悬停在该词上时编辑器将显示的内容。
## Erased Types

Let’s take a look at what happens when we compile the above function `greet` with `tsc` to output JavaScript:

让我们看看用 `tsc` 编译上面的 `greet` 函数，输出 JavaScript 时会发生什么：
```ts
"use strict";
function greet(person, date) {
    console.log("Hello ".concat(person, ", today is ").concat(date.toDateString(), "!"));
}
greet("Maddison", new Date());
```
Notice two things here:
1. Our `person` and `date` parameters no longer have type annotaions.
2. Our "template string" - that string that used backticks (the ` character) - was converted to plain strings with concatenations.

注意两点：
1. 我们的 `person` 和 `date` 参数不再有类型注释。
2. 我们的 "字符串模板" - 使用反引号字符串( ` 字符) - 被转换为连接的普通字符串。

More on that second point later, but let’s now focus on that first point. Type annotations aren’t part of JavaScript (or ECMAScript to be pedantic), so there really aren’t any browsers or other runtimes that can just run TypeScript unmodified. That’s why TypeScript needs a compiler in the first place - it needs some way to strip out or transform any TypeScript-specific code so that you can run it. Most TypeScript-specific code gets erased away, and likewise, here our type annotations were completely erased.

`Remember: Type annotations never change the runtime behavior of your program.`

稍后会详细介绍第二点，但现在让我们关注第一点。类型注释不是 JavaScript （或者 ECMAScript）的一部分，实际上没有任何浏览器或其他运行时可以不加修改地运行 TypeScript。这就是 TypeScript 首先需要编译器的原因-它需要一些方法来剥离或转换任何 TypeScript 特定的代码，所以你可以运行它。大多数 typescript 特定的代码都被擦除了，同样，我们的类型注释也被完全擦除了。
记住：类型注释从不改变你的程序运行行为。
## Downleveling

One other difference from the above was that our template string was rewritten from

与上面的另一个区别是我们的模板字符串是从

```js
`Hello ${person}, today is ${date.toDateString()}!`
```
to
```js
"Hello " + person + ", today is " + date.toDateString() + "!";
```
Why did this happen?

Template strings are a feature from a version of ECMAScript called ECMAScript 2015 (a.k.a. ECMAScript 6, ES2015, ES6, etc. - don’t ask). TypeScript has the ability to rewrite code from newer versions of ECMAScript to older ones such as ECMAScript 3 or ECMAScript 5 (a.k.a. ES3 and ES5). This process of moving from a newer or “higher” version of ECMAScript down to an older or “lower” one is sometimes called downleveling.

By default TypeScript targets ES3, an extremely old version of ECMAScript. We could have chosen something a little bit more recent by using the target option. Running with `--target es2015` changes TypeScript to [target](https://www.typescriptlang.org/tsconfig#target) ECMAScript 2015, meaning code should be able to run wherever ECMAScript 2015 is supported. So running `tsc --target es2015 hello.ts` gives us the following output:

为什么会这样？  
模板字符串是 ECMAScript 2015版本的特性（又叫 ECMAScript 6，ES2015，ES6，等等）。从较新或“更高”版本的 ECMAScript 向下移动到较旧或“较低”版本的过程有时称为降级。  
默认情况下 TypeScript 的目标是ES3，ECMAScript 一个非常的旧的版本，我们可以使用目标选项来选择更近期的版本。运行 `--target es2015` 改变 TypeScript 目标为 ECMAScript 2015，意味着代码应该能够在支持 ECMAScript 2015 的任何地方运行。所以运行 `tsc --target es2015 hello.ts` 给我们以下输出：
```ts
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
greet("Maddison", new Date());
```
`While the default target is ES3, the great majority of current browsers support ES2015. Most developers can therefore safely specify ES2015 or above as a target, unless compatibility with certain ancient browsers is important.`

虽然默认目标是ES3，但当前浏览器绝大多数都支持ES2015。因此，大多数开发人员可以安全地指定 ES2015 或更高版本作为目标，除非与某些古老浏览器的兼容性很重要。
## Strictness

Different users come to TypeScript looking for different things in a type-checker. Some people are looking for a more loose opt-in experience which can help validate only some parts of their program, and still have decent tooling. This is the default experience with TypeScript, where types are optional, inference takes the most lenient types, and there’s no checking for potentially `null/undefined` values. Much like how `tsc` emits in the face of errors, these defaults are put in place to stay out of your way. If you’re migrating existing JavaScript, that might be a desirable first step.

In contrast, a lot of users prefer to have TypeScript validate as much as it can straight away, and that’s why the language provides strictness settings as well. These strictness settings turn static type-checking from a switch (either your code is checked or not) into something closer to a dial. The further you turn this dial up, the more TypeScript will check for you. This can require a little extra work, but generally speaking it pays for itself in the long run, and enables more thorough checks and more accurate tooling. When possible, a new codebase should always turn these strictness checks on.

TypeScript has several type-checking strictness flags that can be turned on or off, and all of our examples will be written with all of them enabled unless otherwise stated. The [strict](https://www.typescriptlang.org/tsconfig#strict) flag in the CLI, or `"strict": true` in a [tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) toggles them all on simultaneously, but we can opt out of them individually. The two biggest ones you should know about are [noImplicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) and [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks).

不同的用户来到 TypeScript，在类型检查器中查找不同的内容。一些人寻找更宽松的选择进行体验，这种体验可以帮助验证他们程序的某些部分，并且还有合适的工具。这是 TypeScript 的默认体验，其中类型是可选的，推断采用最宽松的类型，并且不检查潜在的 `null/undefined` 值。就像 `tsc` 在面对错误时发出的信号一样，这些默认值被放置在适当的位置，避免影响你的工作。如果您正在迁移现有的 JavaScript，这可能是理想的第一步。    

相比之下，很多用户更喜欢让 TypeScript 直接进行尽可能多的验证，这就是为什么该语言也提供了严格设置。这些严格设置将静态类型检查从一个开关(不管你的代码是否被检查)变成一个更接近于拨号盘的东西。你越往上拨，TypeScript 就会为你检查得越多。这可能需要一些额外的工作，但一般来说，从长远看这是值得的，并且支持更彻底的检查和更精确的工具。在适合的情况下，新的代码库应该始终启用这些严格性检查。

TypeScript 有几个可以打开或关闭的严格类型检查标志，我们所有的示例都将启用这些标志，除非另有说明。CLI 中的strict 标志，或者 tsconfig.json 中的 `"strict": true`。可以将这些标志同时全部打开，我们也可以单独选择它们。你应该知道的两个最大的检查是 noImplicitAny 和 strictNullChecks。

## noImplicitAny

Recall that in some places, TypeScript doesn’t try to infer types for us and instead falls back to the most lenient type: `any`. This isn’t the worst thing that can happen - after all, falling back to `any` is just the plain JavaScript experience anyway.

However, using any often defeats the purpose of using TypeScript in the first place. The more typed your program is, the more validation and tooling you’ll get, meaning you’ll run into fewer bugs as you code. Turning on the noImplicitAny flag will issue an error on any variables whose type is implicitly inferred as any.

回想一下在一些地方，TypeScript 不会尝试为我们推断类型，而是退回到最宽松的类型：`any`，这不是能发生很糟糕的事情-毕竟，回退到 `any` 只是普通的 JavaScript 体验。  
无论如何，经常使用后 `any` 违背了使用 TypeScript 的本意。你的程序有更多类型，将会有更多的验证和工具，意味着你编写代码时会遇到更少的 bug，打开 noImplicitAny 标志，任何隐性推断为 any 的变量将会报错。
## strictNullChecks

By default, values like `null` and `undefined` are assignable to any other type. This can make writing some code easier, but forgetting to handle `null` and `undefined` is the cause of countless bugs in the world - some consider it a [billion dollar mistake]! The [strictNullChecks] flag makes handling `null` and `undefined` more explicit, and spares us from worrying about whether we forgot to handle `null` and `undefined`.

默认情况下，像 `null`  和 `undefined` 能分配给其它任何类型。这使得写代码更简单，但是忘记去处理 `null` 和 `undefined` 是造成无数的 bug 的原因-考虑到这是重要的错误！strictNullChecks 标志使得处理 `null` 和 `undefined` 更明确，并使我们不必担心是否忘记处理 `null` 和 `undefined` 。
