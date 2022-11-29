# The Basics
welcome to the first page of the handbook if this your first experience with TypeScript - you may want to start at one of the [Getting Started](https://www.typescriptlang.org/docs/handbook/intro.html#get-started) guides

Each and every value in JavaScript has a set of behaviors you can observe from running different operations.That sounds abstract,but as a quick example,consider some operations we might run on a variable named message.

欢迎来到手册第一页,如果这是你第一次使用 TypeScript - 你可以从"入门"指南开始
在 JavaScript 中每个值都有一系列行为,你可以通过运行不同的操作进行观察。听起来很抽象,举个简单的例子,考虑一些我们可能在一个名为 message 的变量上运行的操作
```ts
  // Accessing the property 'toLowerCase'
  // on 'message' and then calling it
  message.toLowerCase();
  // Calling 'message'
  message();
```
if we break this down,the first runnable line of code accesses a property called toLowerCase and then call it.The second one tries to call message directly.

But assuming we don't know the value of message - and that's pretty common - we can't reliably say what results we'll get from trying to run any of this code

如果我们分析下,第一行可运行的代码访问一个叫做 toLowerCase 的属性,然后调用它。第二行尝试直接调用 message。
但是,假设我们不知道 message 的值 - 这很常见 - 我们不能可靠的说,尝试运行这些代码能得到什么结果。

· Is message callable?
· Does it have a property called toLowerCase on it?
· if it does,is toLowerCase even callable?
· if both of these values are callable,that do they return?
