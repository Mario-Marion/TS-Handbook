# Variable Declaration

`let` 和 `const` 是 JavaScript 中变量声明的两个相对较新的概念。`let` 在某些方面类似于 `var`，但增加了块级作用域和暂时性死区等特性。

`const` 是 `let` 的扩充，它可以防止给变量重新赋值。

TypeScript 是 JavaScript 的扩展，该语言自然支持 `let` 和 `const`。在这里，我们将详细说明这些新声明以及它们为什么比 `var` 更好。

如果你非常熟悉 JavaScript 中 `var` 声明的所有怪癖，你可以跳过前面几个小节。

## `var` 声明

在 JavaScript 中声明一个变量通常是用 `var` 关键字来完成的。

```js
var a = 10;
```

例子中声明了变量 `a`，值为 `10`

也可以在函数内部声明变量：

```js
function f() {
  var message = "Hello, world!";
  return message;
}
```

我们也可以在其他函数中访问这些相同的变量:

```js
function f() {
  var a = 10;
  return function g() {
    var b = a + 1;
    return b;
  };
}
var g = f();
g(); // returns '11'
```

在上面的例子中，`g` 捕获了在 `f` 中声明的变量 `a`。在 `g` 被调用的时候，`a` 的值将与 `f` 中的 `a` 的值绑定。即使在 `f` 完成运行后调用 `g`，它也能够访问和修改 `a`。

```js
function f() {
  var a = 1;
  a = 2;
  var b = g();
  a = 3;
  return b;
  function g() {
    return a;
  }
}
f(); // returns '2'
````

### 作用域规则

对于那些习惯于其他语言的人来说，`var` 声明有一些奇怪的作用域规则。举个例子：

```ts
function f(shouldInitialize: boolean) {
  if (shouldInitialize) {
    var x = 10;
  }
  return x;
}
f(true); // returns '10'
f(false); // returns 'undefined'
```

一些读者可能会对这个例子多看两眼。变量 `x` 是在 `if` 块中声明的，但我们能够从该块外部访问它。这是因为 `var` 声明没有块级作用域，只有函数作用域。参数也是函数作用域的。

这些作用域规则可能导致几种类型的错误。更恶劣的问题是，多次声明同一个变量并不是错误:

```ts
function sumMatrix(matrix: number[][]) {
  var sum = 0;
  for (var i = 0; i < matrix.length; i++) {
    var currentRow = matrix[i];
    for (var i = 0; i < currentRow.length; i++) {
      sum += currentRow[i];
    }
  }
  return sum;
}
```

对于一些经验丰富的 JavaScript 开发人员来说，这可能很容易发现，内部 `for` 循环会意外地覆盖变量 `i`，因为 `i` 引用了相同的函数作用域变量。

### 变量捕获怪癖

猜一下以下代码片段会输出什么：

```js
for (var i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100 * i);
}
```

对于不熟悉 JavaScript 的人来说，会认为 `setTimeout` 将在一定毫秒数后执行回调，打印 0-9。  

可事实是：

```js
10 10 10 10 10 10 10 10 10 10
```

确实是一定毫秒后执行了，可是打印的却都是 `10`。`setTimeout` 的每个函数表达式实际上都引用了来自同一作用域的同一个 `i`。  

让我们花点时间考虑一下这是什么意思。`setTimeout` 将在一定毫秒后运行一个回调，但是只会在 `for` 循环执行完毕之后执行；也就是说，`for` 循环执行还未完毕，就算 `seTimeout` 毫秒数时间到了，也是不会执行的。（可查阅 JS 微任务，宏任务相关资料）

当 `for` 循环停止执行时，`i` 的值为 `10`。所以每次给定的回调被调用时，它都会输出 `10`!  
一种常见的解决方法是使用 IIFE —— 立即调用的函数表达式 —— 在每次迭代中产生闭包，捕获 `i`：

```js
for (var i = 0; i < 10; i++) {
  // capture the current state of 'i'
  // by invoking a function with its current value
  (function (i) {
    setTimeout(function () {
      console.log(i);
    }, 100 * i);
  })(i);
}
```

这种看起来很奇怪的模式实际上很常见。参数列表中的 `i` 实际上储存了 `for` 循环中声明的 `i`，只是由于我们将它们命名为相同的名称。（可查阅 JS 闭包相关资料）

## `let` 声明


到目前为止，你已经发现 `var` 存在的一些问题，这正是引入 `let` 语句的原因。除了关键字不同之外，`let` 语句的编写方式与 `var` 语句相同。

```js
let hello = "Hello!";
```

关键的区别不在于语法，而在于语义，现在让我们深入研究这一点。

### 块级作用域

当使用 `let` 声明变量时，它使用了所谓的 词法作用域 或 块作用域。块作用域的变量在 最近的包含块 或 `for` 循环之外 是不可见的。

```ts
function f(input: boolean) {
  let a = 100;
  if (input) {
    // Still okay to reference 'a'
    let b = a + 1;
    return b;
  }
  // Error: 'b' doesn't exist here
  return b;
}
```

在这里，我们有两个局部变量 `a` 和 `b`。`a` 的作用域被限制在 `f` 函数体中，而 `b` 的作用域被限制在 `if` 语句的块中。  

在 `catch` 子句中声明的变量也有类似的作用域规则：

```js
try {
  throw "oh no!";
} catch (e) {
  console.log("Oh well.");
}
// Error: 'e' doesn't exist here
console.log(e);
```

块级作用域变量的另一个特点是，它们在实际声明之前不能被读取或写入。虽然这些变量在它们的作用域中“存在”，但所有指向它们声明之前的点都是它们暂时死区的一部分。也就是说不能在 `let` 语句之前访问它们，幸运的是 TypeScript 会让你知道这一点：

```ts
a++; // 在声明之前非法使用 'a'
let a;
```

下面例子中，需要注意的是，你仍然可以在声明块级作用域变量之前捕获它。唯一的问题是，在声明之前调用该函数是非法的。如果目标是 ES2015，运行时将抛出错误；然而，在 TypeScript 是允许的，不会将此报告为错误，但是编译为 JS 运行时还是会抛出错误：

```ts
function foo() {
  // 可以捕获 'a'
  return a;
}
// 在声明 'a' 之前非法调用 'foo'
// 运行时应该在这里抛出一个错误
foo();
let a;
```

有关暂时性死区的更多信息，请参阅 [Mozilla Developer Network](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let)

### 重新声明和遮蔽

在 `var` 声明中，我们提到过你在相同作用域声明变量多少次并不重要；你只会得到同一个：

```js
function f(x) {
  var x;
  var x;
  if (true) {
    var x;
  }
}
```

在上面的例子中，`x` 的所有声明实际上都指向同一个 `x`，这是完全有效的。但也往往最终是成为 bug 的来源。值得庆幸的是，`let` 声明没有那么宽松：

```js
let x = 10;
let x = 20; // error: 不能在相同作用域重复声明 'x'
```

重复声明 TypeScript 就会告诉我们有问题：

```ts
function f(x:number) {
  let x = 100; // error: 与参数冲突
}
function g() {
  let x = 100;
  var x = 100; // error: 不能声明两个 'x'
}
```

这并不是说块级作用域的变量永远不能用函数作用域的变量声明。而是，块级作用域的变量需要在一个明显不同的块中声明：

```js
function f(condition, x) {
  if (condition) {
    let x = 100;
    return x;
  }
  return x;
}
f(false, 0); // returns '0'
f(true, 0); // returns '100'
```

在多层嵌套的作用域中，使用上层作用域变量名称的行为称为遮蔽。例如，假设我们使用 `let` 变量编写之前的 `sumMatrix` 函数：

```ts
function sumMatrix(matrix: number[][]) {
  let sum = 0;
  for (let i = 0; i < matrix.length; i++) {
    var currentRow = matrix[i];
    for (let i = 0; i < currentRow.length; i++) {
      sum += currentRow[i];
    }
  }
  return sum;
}
```

这个版本的循环实际上会正确地执行求和，因为内循环的 `i` 遮蔽了外循环的 `i`。

为了编写更清晰的代码，通常应该避免使用遮蔽。

### 块级作用域变量捕获

当我们第一次接触到用 `var` 声明捕获变量的概念时，我们简要地讨论了捕获变量后的行为。为了更好地直观理解这一点，可理解为每次运行作用域时，它都会创建一个变量的“环境”，该环境及其捕获的变量，在作用域中的所有内容都执行完毕之后依然可以存在。

```js
function theCityThatAlwaysSleeps() {
  let getCity;
  if (true) {
    let city = "Seattle";
    getCity = function () {
      return city;
    };
  }
  return getCity();
}
```

因为我们从它的环境中捕获了 `city`，所以我们仍然可以访问它，尽管 `if` 块已经执行完毕。

回想一下，在前面的 `setTimeout` 示例中，我们最终需要使用 IIFE 来捕获 `for` 循环的每次迭代的变量状态。实际上，我们所做的是为我们捕获的变量创建一个新的变量环境。这有点痛苦，但幸运的是，你再也不用这么做了。

`let` 声明在声明为循环的一部分时具有截然不同的行为。这些声明不是仅仅为循环本身引入一个新的环境，而且在每次迭代中创建一个新的作用域。这就是我们在 IIFE 中所做的，所以我们可以将旧的 `setTimeout` 示例更改为使用 `let` 声明：

```js
for (let i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100 * i);
}
```

和预期的一样，会打印出：

```js
0 1 2 3 4 5 6 7 8 9
```

## `const` 声明

`const` 声明是声明变量的另一种方式。

```js
const numLivesForCat = 9;
```
  
它们类似于 `let` 声明，但顾名思义，它们的值一旦被绑定就无法更改。换句话说，它们具有与 `let` 相同的作用域规则，但不能重新赋值。

需要注意的是，不应与它们所引用的值是不可变的想法相混淆，例如：

```js
const numLivesForCat = 9;
const kitty = {
  name: "Aurora",
  numLives: numLivesForCat,
};
// Error
kitty = {
  name: "Danielle",
  numLives: numLivesForCat,
};
// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

除非你采取特定措施来避免它，否则 `const` 变量的内部状态仍然是可修改的。幸运的是，TypeScript 允许你指定对象的成员是只读的。详情可参考 [接口章节](https://www.typescriptlang.org/docs/handbook/interfaces.html)。

## `let` vs. `const`

`let` 与 `const`声明类型具有相同的作用域语义，我们很自然地会问应该使用哪一种。答案是：视情况而定。

应用 [最小权限原则](https://wikipedia.org/wiki/Principle_of_least_privilege)，除了计划修改的声明外，所有声明都应该使用 `const`。其基本原理是，如果一个变量不需要写入，那么在同一代码库上工作的其他人也不应该能够写入对象，不需要考虑他们是否需要重新赋值给该变量。在对数据流进行推理时，使用 `const` 还使代码更加可预测。

如果觉得适用的话，可以和你团队的其他成员商量一下。

本手册的大部分使用 `let` 声明。

## 解构

TypeScript 的另一个 ECMAScript 2015 特性是解构。要获得完整的参考资料，请参阅 [Mozilla Developer Network](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)。在本节中，我们将简要概述。

### 数组解构

最简单的解构形式是数组解构赋值：

```js
let input = [1, 2];

let [first, second] = input;

console.log(first); // outputs 1

console.log(second); // outputs 2
```

这将创建两个新变量，名为 `first` 和 `second`。这等同于使用索引，但更方便：

```js
first = input[0];
second = input[1];
```

解构也适用于已经声明的变量：

```js
// 变量互换
[first, second] = [second, first];
```

还有函数的参数：

```js
function f([first, second]: [number, number]) {
  console.log(first);
  console.log(second);
}
f([1, 2]);
```

你可以为数组中剩余的元素创建一个变量，使用 `...` 语法：

```js
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

可以忽略你不关心的尾随元素:

```js
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```
或不关心某些元素：

```js
let [, second, , fourth] = [1, 2, 3, 4];
console.log(second); // outputs 2
console.log(fourth); // outputs 4
```

### 元组解构

元组可以像数组一样被解构；解构变量获得相应元组元素的类型：

```ts
let tuple: [number, string, boolean] = [7, "hello", true];
let [a, b, c] = tuple; // a: number, b: string, c: boolean
```

在元组的元素范围之外解构元组是错误的：

```ts
let tuple: [number, string, boolean] = [7, "hello", true];
let [a, b, c, d] = tuple; // Error, 索引 3 没有元素
```
与数组一样，你可以使用 `...` 来解构元组的其余部分，获得另一个更短的元组：

```ts
let tuple: [number, string, boolean] = [7, "hello", true];
let [a, ...bc] = tuple; // bc: [string, boolean]
let [a, b, c, ...d] = tuple; // d: [], 空元组
```

或者忽略尾随元素或其他元素：

```ts
let [a] = tuple; // a: number
let [, b] = tuple; // b: string
```

### 对象解构

你也可以解构对象：

```js
let o = {
  a: "foo",
  b: 12,
  c: "bar",
};
let { a, b } = o;
```

这将从 `o.a` 和 `o.b` 中创建新的变量 `a` 和 `b`。注意，如果不需要 `c`，可以跳过它。

像数组解构一样，你可以在没有声明的情况下赋值：

```js
({ a, b } = { a: "baz", b: 101 });
```

注意，我们必须用圆括号把这个语句括起来。JavaScript 通常解析 `{` 为"块"的开头。

你可以使用语法 `...` 为对象中的剩余项创建一个变量:

```js
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;
```

### 属性重命名

你也可以给属性起不同的别名：

```js
let { a: newName1, b: newName2 } = o;
```

这里的语法开始变得奇怪。实际上你可以把 `a: newName1` 读成 `"a as newName1"`。方向是从左到右：

```js
let newName1 = o.a;
let newName2 = o.b;
```

令人困惑的是，这里的冒号并不表示类型。如果要指定类型，需要在整个解构之后写入：

```ts
let { a: newName1, b: newName2 }: { a: string; b: number } = o;
```

### 默认值

默认值，允许你在属性未定义的情况下指定默认值：

```ts
function keepWholeObject(wholeObject: { a: string; b?: number }) {
  let { a, b = 1001 } = wholeObject;
}
```

在这个例子中，`b?` 表示 `b` 是可选的，因此可能为 `undefined`，所以在解构赋值的时候，设置 `b` 默认值为 `1001`


## 函数声明

解构也适用于函数声明：

```ts
type C = { a: string; b?: number };
function f({ a, b }: C): void {
  // ...
}
```

但是对于参数来说，指定默认值更为常见，而通过解构获得正确的默认值可能看起来有点混乱。

```ts
function f({ a, b = 0 } = { a: "" }): void {
  // ...
}
f({ a: "yes" }); // ok, default b = 0
f(); // ok, default to { a: "" }, which then defaults b = 0
f({}); // error, 'a' is required if you supply an argument
```

小心使用解构。正如前面的例子所演示的，除了最简单的解构表达式之外，任何东西都令人困惑。对于深度嵌套解构尤其如此，即使没有重命名、默认值和类型注释，这种解构也很难理解。尽量使解构表达式保持小而简单。

## 扩展

展开操作符与解构相反。它允许你将一个数组扩展到另一个数组，或者将一个对象扩展到另一个对象。例如：

```js
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
```

`bothPlus` 值为 `[0,1,2,3,4,5]`。扩展创建了 `first` 和 `second` 的浅拷贝。它们本身不会因扩展而改变。

对象也可以扩展：

```js
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };
```

现在 `search` 是 `{ food: "rich", price: "$$", ambiance: "noisy" }`。对象扩展比数组扩展更复杂。像数组扩展一样，它从左到右进行，但结果仍然是一个对象。这意味着扩展对象中较晚出现的属性会覆盖较早出现的属性。例如：

```js
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { food: "rich", ...defaults };
```

`defaults` 中的 `food: "spicy"` 属性会覆盖 `food: "rich"`

对象扩展还有其他一些令人惊讶的限制。首先，它只包含对象 [自身的可枚举属性](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)。这意味着当你扩展一个对象的实例时，你会丢失原型上的方法：

```js
class C {
  p = 12;
  m() {}
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/variable-declarations.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

