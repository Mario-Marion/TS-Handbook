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