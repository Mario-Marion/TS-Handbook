---
highlight: vs2015
---
# More on Functions
函数是任何应用程序的基本构件，无论是本地函数、从另一个模块导入的函数，还是类上的方法。它们也都是值，就像其他值一样，TypeScript 有很多方法来描述如何调用函数。让我们了解如何描述函数的类型。

## 函数类型表达式
描述函数的最简单方法是使用函数类型表达式。它的语法类似箭头函数：
```ts
function greeter(fn: (a: string) => void) {
    fn("Hello, World");
}

function printToConsole(s: string) {
    console.log(s);
}

greeter(printToConsole);
```
语法 `(a: string) => void` 意思是："函数拥有一个参数，名为 `a`，类型为 `string`，并且函数没有返回值"。如果函数参数没有指定类型，那么它就是隐性的 `any` 类型。

>注意，参数名称是必须的。函数表达式 `(string) => void`  表示的是：函数拥有一个参数，名为 string，类型为隐性的 any 类型！

当然，我们可以使用类型别名去命名函数类型：
```ts
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
    // ...
}
```

## 调用签名
在 JavaScript 中，函数除了可调用外，还可以拥有属性。然而，函数类型表达式语法不允许声明属性。如果我们想要描述一些可调用的属性，我们可以在对象类型里面编写调用签名：
```ts
type DescribableFunction = {
    description: string;
    (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
    console.log(fn.description + " returned " + fn(6));
}
```
注意，这个语法和函数表达式稍微有点不同——函数参数列表和返回类型之间使用的是 `:`，而不是 `=>` 。

## 构造签名
JavaScript 函数同样可以用 `new` 操作符调用。TypeScript 将它们称为构造函数，因为它们通常会创建一个新对象。你可以通过在调用签名前添加 `new` 关键字，编写构造签名：
```ts
type SomeConstructor = {
    new (s: string): SomeObject;
};

function fn(ctor: SomeConstructor) {
    return new ctor("hello");
}
```
有些对象，像 JavaScript 的 `Date` 对象，可以使用 `new` 或不使用 `new` 调用。你可以在同一类型中任意组合调用签名和构造签名:
```ts
interface CallOrConstruct {
    new (s: string): Date;
    (n?: number): number;
}
```

## 泛型函数

编写一个函数，输入的类型与输出的类型有关联，或者两个输入的类型以某种方式相关联，是很常见的。

假设一个函数返回数组的第一个元素：
```ts
function firstElement(arr: any[]) {
    return arr[0];
}
```
它的返回类型是 `any`。如果函数能返回数组元素的类型就更好了。

在 TypeScript 中，当我们想要描述两个值之间的对应关系时，就会使用泛型。我们通过在函数签名里面声明一个类型参数：
```ts
function firstElement<Type>(arr: Type[]): Type | undefined {
    return arr[0];
}
```
通过在这个函数添加类型参数 `Type` ，并在两个地方使用它，我们就创建了函数输入（数组）和输出（返回值）之间的链接、现在当我们调用它，会输出一个更具体的类型：
```ts
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);

// n is of type 'number'
const n = firstElement([1, 2, 3]);

// u is of type undefined
const u = firstElement([]);
```
### 推断
注意，`Type` 在上面例子中没有声明具体的类型。类型是 TypeScript 推断出来的。

我们可以使用多个类型参数。例如，`map` 的独立运行版本：
```ts
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
    return arr.map(func);
}

// 参数 'n' 是 'string' 类型
// 变量 'parsed' 是 'number[]' 类型
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```
注意，在这个例子中，TypeScript 可以推断 `Input` 泛型参数的类型（从给定的字符串数组而推断），也可以根据函数表达式的返回值（`number`）推断 `Output` 泛型参数的类型。

### 约束
我们编写了一些泛型函数，可以处理任意类型的值。有时候我们想要去关联两个值，但是规定只能操作某个值的子集。在这种情况下，我们可以使用'约束'来限制类型参数可以接受的类型。

让我们编写一个函数，接收两个参数，返回两个参数中较长的一个。首先，我们需要参数要有一个 `length` 属性，属性类型为 `number`。然后通过 `extends` 从句来约束类型参数为该类型：
```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
    if (a.length >= b.length) {
        return a;
    } else {
        return b;
    }
}
 

// longerArray 是 'number[]' 类型
const longerArray = longest([1, 2], [1, 2, 3]);

// longerString 是 'alice' | 'bob' 类型
const longerString = longest("alice", "bob");

// Error! 数值没有 'length' 属性
const notOK = longest(10, 100);
// Error：实参 'number' 类型不能赋值给行参 '{length: number;}' 类型
```
在这个例子中有一些有趣的事情需要注意。我们允许 TypeScript 推断 `longest` 的返回类型。返回类型推断也适用于泛型函数。

因为我们约束 `Type` 为 `{length: number}` 类型，所以我们可以访问参数 `a` 和 `b` 的 `length` 属性。如果没有类型约束，我们将无法访问 `length` 属性，因为参数可能是其它没有 `length` 属性的类型。

`longerArray` 和 `longerString` 的类型是根据参数推断出来的。记住，泛型都是将两个或多个值与同一类型相关联！

所以例子中，`longest(10,100)` 是不允许调用的，因为 `number` 类型没有 `length` 属性。

### 约束值
下面是使用泛型约束时常见的错误:
```ts
function minimumLength<Type extends { length: number }>(
    obj: Type,
    minimum: number
): Type {
    if (obj.length >= minimum) {
        return obj;
    } else {
        return { length: minimum };
        // Error：'{ length: number; }' 类型不能赋值给 'Type' 类型
    }
}
```
> 这里的返回值必须和 Type 参数类型保持匹配关系，所以不能直接返回 `{length: minimum}`

看起来这个函数是 OK 的—— `Type` 被约束为 `{length: number}`，函数返回 `Type` 或 匹配 `{length: number}` 的值。问题是该函数声明要求返回与传入参数相同类型的对象，也就是说，只能返回匹配 `Type` 类型的值。如果这段代码是合法的，那么你会编写出绝对不能工作的代码：
```ts
// 'arr' 值为 { length: 6 }
const arr = minimumLength([1, 2, 3], 6);

// 这里会崩溃，因为 'slice' 是数组方法，但 'minimumLength' 返回的是对象
console.log(arr.slice(0));
```

### 指定类型参数
TypeScript 通常可以在泛型调用中推断出类型参数，但并非总是如此。例如，你写了一个函数来组合两个数组：
 ```ts
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
    return arr1.concat(arr2);
}
```
TypeScript 根据第一个参数推断 `Type` 类型为 `number[]` 类型，数组不一致，则调用此函数会出错：
```ts
const arr = combine([1, 2, 3], ["hello"]);
// 'string' 类型不能赋值给 'number' 类型
```
如果你想这样做，得手动指定 `Type` 类型：
```ts
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```
### 编写良好泛型函数的指南
拥有太多类型参数或在不需要它们的地方使用约束，会降低类型推断的成功率。

####  下推类型参数
下面是两种编写函数的方法看起来很相似：
```ts
function firstElement1<Type>(arr: Type[]) {
    return arr[0];
}
function firstElement2<Type extends any[]>(arr: Type) {
    return arr[0];
}

// a: number (good)
const a = firstElement1([1, 2, 3]);

// b: any (bad)
const b = firstElement2([1, 2, 3]);
```
乍一看，这两个函数是一样的，但是 `firstElement1` 是编写这个函数更好的方法，它的推断返回类型是 `Type`。`firstElement2` 的推断返回类型是 `any`，因为 TypeScript 使用了约束类型解析 `arr[0]` 表达式，而不是在函数调用期间 "等待" 解析参数元素。

> **规则**：在可能的情况下，使用类型参数本身而不是约束它

#### 使用更少的类型参数
这是另一对相似的函数：
```ts
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
    return arr.filter(func);
}

function filter2<Type, Func extends (arg: Type) => boolean>(
    arr: Type[],
    func: Func
): Type[] {
    return arr.filter(func);
}
```
`filter2` 函数中多声明了个类型参数 `Func`。如果让 TypeScript 自行推断，两个函数倒是没什么区别。但是如果调用者想要手动指定类型参数时，`filter2` 函数必须传两个类型参数，这是没必要的。

> **规则**：总是使用尽可能少的类型参数
#### 类型参数应该出现两次
有时一个函数可能不需要泛型:
```ts
function greet<Str extends string>(s: Str) {
    console.log("Hello, " + s);
}
greet("world");
```
可以很容易地写出一个更简单的版本：
```ts
function greet(s: string) {
    console.log("Hello, " + s);
}
```
记住，类型参数用于关联多个值。如果一个类型参数在函数签名中只使用了一次，那么它与任何东西都没有关联。
> **规则**：如果一个类型参数只出现在一个位置，那么你就要考虑是否真的需要它
## 可选参数
JavaScript 中的函数允许接收不固定数量的参数，例如，参数 `n` 的 `toFixed` 方法接收一个可选的 `numer` 类型参数去计算数位：
```ts
function f(n: number) {
    console.log(n.toFixed()); // 0 arguments
    console.log(n.toFixed(3)); // 1 argument
}
```
我们可以在 TypeScript 中通过 `?` 把参数标记为可选的：
```ts
function f(x?: number) {
    // ...
}
f(); // OK
f(10); // OK
```
上面例子中参数被指定为 `number` 类型，并且是可选的。所以参数 `x` 实际上的类型为 `number | undefined`，因为在 JavaScript 中，没有传递的参数的值为 `undefined`。

你也可以提供参数默认值：
```ts
function f(x = 10) {
    // ...
}
```
现在 `f` 函数体中，`x` 为类型 `number` 类型，因为任何为 `undefined` 的参数将被替换为 `10`。

注意，当形参为可选时，调用者可以传递 `undefined` 模拟一个 "缺少" 实参：
```ts
declare function f(x?: number): void;
// cut
// All OK
f();
f(10);
f(undefined);
```
### 回调中的可选参数
了解可选参数和函数类型表达式，在编写回调函数调用时，很容易犯以下错误：
```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
    for (let i = 0; i < arr.length; i++) {
        callback(arr[i], i);
    }
}
```
人们在写索引时通常想要什么？作为一个可选参数，它们希望这两个调用都是合法的:
```ts
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```
但是调用 `callback` 可能只传递一个参数。函数可能是这么定义的：
```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
    for (let i = 0; i < arr.length; i++) {
        // 只传递一个参数
        callback(arr[i]);
    }
}
```
TypeScript 会强制执行这个含义，并发出错误信息：
```ts
myForEach([1, 2, 3], (a, i) => {
    console.log(i.toFixed());
    // Error：Object is possibly 'undefined'.
}
```
在 JavaScript 中，如果调用的函数实参比形参多，那么额外的参数就会被忽略。TypeScript 的行为也是一样的。而具有更少形参的函数总是可以取代具有更多形参的函数（相同参数类型）。

>在为回调函数编写函数类型时，永远不要编写可选参数，除非你打算在不传递实参的情况下调用该函数

## 函数重载
某些 JavaScript 函数可以在多种参数数量和类型下调用。如下例中的 `Date` 构造函数，接受时间戳（一个参数）或 月份/天/年（三个参数）。

在 TypeScript 中，我们可以通过编写重载签名来指定一个函数可以用不同方式调用。如，写一些函数签名（通常2个或更多），然后是函数体：
```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
    if (d !== undefined && y !== undefined) {
        return new Date(y, mOrTimestamp, d);
    } else {
        return new Date(mOrTimestamp);
    }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
// Error：没有重载需要2个参数，但是重载需要1个或3个参数。
```
在这个例子中，我们编写了两个重载：一个接受一个参数，另一个接受三个参数。前两个签名称为重载签名，

最后编写了一个具有兼容签名的函数实现，称为实现签名。但是，不能直接调用该函数，也不能用两个参数调用它！虽然我们在函数必需参数的后面写了两个可选形参。只能传递一个或三个参数（满足两个函数签名中的一个）
### 重载签名和实现签名
通常人们会写以下的代码，却不明白为什么会有错误：
```ts
function fn(x: string): void;
function fn() {
    // ...
}
fn();
// Error：期望1个参数，但是参数为 0
```
其实函数体的实现签名不能从外部 "看到" 的。

> 实现签名在外部是不可见的。在编写重载函数时，应该始终在实现签名上方有两个或多个签名。

实现签名还必须与重载签名兼容。例如，以下函数有错误，因为实现签名没有以正确的方式匹配重载：
```ts
function fn(x: boolean): void;
function fn(x: string): void;
// Error：这个重载签名与实现签名不兼容。
function fn(x: boolean) {}
```
```ts
function fn(x: string): string;
function fn(x: number): boolean;
// Error：这个重载签名与实现签名不兼容。
function fn(x: string | number) {
    return "oops";
}
```
### 编写良好的重载
与泛型一样，在使用函数重载时也应该遵循一些规则。遵循这些原则将使你的函数更容易调用、更容易理解和更容易实现。

假设一个函数返回一个字符串或数组的长度：
```ts
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
    return x.length;
}
```
这个函数很好；我们可以用字符串或数组调用它。但是，我们不能用值是 `string | number` 类型来调用它，因为 TypeScript 只能将函数调用解析为重载签名中的一个:
```ts
len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
// Error：没有重载匹配这个调用
```
因为两个重载有相同的参数数量和相同的返回类型，所以我们可以编写一个非重载版本的函数替代：
```ts
function len(x: any[] | string) {
    return x.length;
}
```
这非常好！和上面的重载函数版本一样的，可传递字符串或数组并返回长度。这么写的好处就是，我们不必去确定实现签名是否正确。

> 尽可能使用联合类型的参数，而不是重载

## 声明函数中的 `this`
TypeScript 通过代码流分析，推断 `this` 在函数中应该是什么。如下例子：
```ts
const user = {
    id: 123,
    admin: false,
    becomeAdmin: function () {
        this.admin = true;
    }
};
```
TypeSript 理解 `user.becomeAdmin` 函数有一个对应的 `this`，那就是外部对象 `user`。大部分情况下这已经足够，但是在很多情况下，你需要更多的控制 `this` 代表哪个对象。JavaScript 规范规定函数不能有称为 `this` 的形参，因此 TypeScript 使用这个语法空间，去让你声明函数体内 `this` 的类型。
```ts
interface User {
  id: number;
  admin: boolean;
}
declare const getDB: () => DB;
// ---cut---
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```
这种模式在回调风格的 API 中很常见，当调用函数时通常由另一个对象控制。注意，你需要去使用 `function` 而不是箭头函数去获得这种行为：
```ts
interface DB {
    filterUsers(filter: (this: User) => boolean): User[];
}
const db = getDB();
const admins = db.filterUsers(() => this.admin);
// this Error：箭头函数捕获 'this' 的全局值。
// admin Error：元素隐式的具有 'any' 类型，因为 'typeof gloalThis' 类型没有索引签名。
```
## 其它需要了解的类型
在使用函数类型时，还需要识别一些经常出现的额外类型。与所有类型一样，可以在任何地方使用它们，但它们在函数上下文中有特别意义。
### `void`
`void` 表示函数没有返回值。当函数没有任何 `return` 语句，或者没有从这些返回语句中返回任何显式值时，那么返回值会被推断类型 `void`：
```ts
// 推断返回类型是 void
function noop() {
    return;
}
```
在 JavaScript 中，一个函数没有返回任何值，将隐式的返回 `undefined` 值。然而 `void` 和 `undefined` 在 TypeScript 中不是相同的类型。本章末尾有更多的细节。

> `void` 和 `undefined` 不一样

### `object`
特殊类型 `object` 指的是：除了原始类型（`string`, `number`, `bigint`, `boolean`, `symbol`, `null`, 或 `undefined`）的所有值。与空对象类型 `{}` 不同，并且与全局类型 `Object` 也不同。你很可能永远不会使用 `Object`。

> object 不是 Object。最好总是使用 object！

注意，在 JavaScript 中，函数值是对象：它们有属性， 在它们的原型链上有 `Object.prototype`，`instanceof Object` 为 `true`，你可以对它们调用 `Object.keys`，等等。因此，在 TypeScript 中，函数类型被认为是 `Ojbect`

### `unknown`
`unknown`  类型表示任意值。和 `any` 类型类似，但更安全，因为对 `unknown` 值做任何操作都是不合法的：
```ts
function f1(a: any) {
    a.b(); // OK
}
function f2(a: unknown) {
    a.b();
    // Error：对象是 'unknown' 类型
}
```
`unknown` 在描述函数类型时很有用，因为可以描述接受任何值的函数，而函数体中没有 `any` 值。

你也可以描述一个返回未知类型值的函数：
```ts
declare const someRandomString: string;
// ---cut---
function safeParse(s: string): unknown {
    return JSON.parse(s);
}
// 需要小心使用 'obj'!
const obj = safeParse(someRandomString);
```
### `never`
有些函数从不返回值：
```ts
function fail(msg: string): never {
    throw new Error(msg);
}
```
`never` 类型表示从未观察到的值。`never` 在返回类型中，这意味着函数抛出异常或终止程序的执行。

当 TypeScript 控制流分析确定联合类型中没有东西时，`never` 也会出现。
```ts
function fn(x: string | number) {
    if (typeof x === "string") {
        // ...
    } else if (typeof x === "number") {
        // ...
    } else {
        x; // x 为 'never' 类型!
    }
}
```
### `Function`
全局类型 `Function` 描述像 `bind`，`call`，`apply` 等属性，以及其它出现在 JavaScript 中的所有函数值。它还有特殊的属性，即 `Function` 类型的值总是可以被调用；这些调用返回 `any` 类型：
```ts
function doSomething(f: Function) {
    return f(1, 2, 3);
}
```
这是一个无类型的函数调用，最好避免这么写，因为返回了不安全的 `any` 类型。

如果你需要接收一个任意函数，但是不打算调用它，`() => void` 类型通常更安全。

## 剩余行参和实参
### 剩余行参
除了使用可选参数或重载，使函数可以接受各种不固定的参数数量外，还可以使用'剩余行参'，定义接受无限制参数数量的函数。

剩余行参出现在所有其它行参之后，使用 `...` 语法：
```ts
function multiply(n: number, ...m: number[]) {
    return m.map((x) => n * x);
}

// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```
在 TypeScript 中，剩余行参的类型注释是隐性的 `any[]` 类型而不是 `any` 类型，并且给出的任何类型注释必须是 `Array<T>` 或 `T[]` 形式，或者是元组类型（我们将在后面学习）。
### 剩余实参
相反，我们可以使用数组的扩展语法，提供给函数可变数量的实参。例如，数组的 `push` 方法接受任意数量的参数：
```ts
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```
注意，通常情况下，TypeScript 并不认为数组是不可变的。这会导致一些令人惊讶的行为：
```ts
const args = [8, 5];
// args 推断类型为 number[] -- "具有零个或多个数值的数组",
const angle = Math.atan2(...args);
// Error：展开实参必须是元组类型或传递一个剩余参数。
```
> Math.atan2 定义：(method) Math.atan2(y: number, x: number): number

`Math.atan2` 只需要两个参数，所以报错

这种情况的最佳解决方案有点取决于你的代码，但一般来说，`const` 上下文是最直接的解决方案：
```ts
// 推断为长度为2的元组
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```
在目标为较旧的运行环境时，使用剩余参数可能需要打开[downlevelIteration](https://www.typescriptlang.org/tsconfig#downlevelIteration)。
### 参数解构
你可以使用参数解构来方便地将，作为参数的对象解构一个或多个局部变量到函数体中。在 JavaScript 中，它看起来是这样的：
```ts
function sum({ a, b, c }) {
    console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```
对象的类型注释紧跟在解构语法之后：
```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {
    console.log(a + b + c);
}
```
这看起来有点冗长，你也可以在这里使用类型别名:
```ts
// 和前面的例子一样
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
    console.log(a + b + c);
}
```
## 函数的可赋值性
### 返回 `void` 类型
函数的 `void` 返回类型可以产生一些不寻常的，但预期的行为。

返回类型为 `void` 的上下文类型不会强制函数不返回某些内容，换句话说，这是一个具有 `void` 返回类型的上下文函数类型（`type vf = () => void`），实现时，可以返回任何其他值，但它将被忽略。

因此，以下实现 `() => void` 类型是有效的：
```ts
type voidFunc = () => void;
const f1: voidFunc = () => {
    return true;
};
const f2: voidFunc = () => true;
const f3: voidFunc = function () {
    return true;
};
```
以上函数定义都是有效的，并且当这些函数的返回值赋值给另一个变量时，它将保留 `void` 类型：
```ts
const v1 = f1(); // const v1: void
const v2 = f2(); // const v2: void
const v3 = f3(); // const v3: void
```
这种行为的存在，使得以下代码是有效的，即使 `Array.prototype.push` 返回一个数值，而 `Array.prototype.forEach` 方法期望接收一个返回 `void` 类型的函数。
```ts
const src = [1, 2, 3];
const dst = [0];
src.forEach((el) => dst.push(el));
```
还有一种特殊情况需要注意，上面例子是用函数类型表达式定义的，当用字面量函数定义，具有 `void` 返回类型函数时，该函数不能返回任何东西。
```ts
function f2(): void {
    // Error：'boolean' 类型不能赋值给 'void' 类型
    return true;
}

const f3 = function (): void {
    // Error：'boolean' 类型不能赋值给 'void' 类型
    return true;
};
```
注意，例子中 `f1` 等变量会被 TypeScript 推断为函数类型表达式（`() => void`），对字面量函数声明使用 `typeof`，也被推断为函数类型表达式，所以下面例子是有效的：
```ts
function Fn(): void {
};
const f4: typeof Fn = function () {
  return true
}
```

有关 `void` 的更多信息，请参考以下其他文档:
-   [v1 handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html#void)
-   [v2 handbook](https://www.typescriptlang.org/docs/handbook/2/functions.html#void)
-   [FAQ - “Why are functions returning non-void assignable to function returning void?”](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)

感谢观看，如有错误，望指正

>官网地址： <https://www.typescriptlang.org/docs/handbook/2/functions.html>
>
>github 资料： <https://github.com/Mario-Marion/TS-Handbook>
