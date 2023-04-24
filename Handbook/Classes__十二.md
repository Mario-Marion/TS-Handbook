# Classes
TypeScript 支持所有 ES2015 中引入的的 class 关键字。
## 类成员
这是最基础的类——一个空类
```ts
class Point {}
```
### 字段
在类上声明一个公共可写的属性：
```ts
class Point {
    x: number;
    y: number;
}
const pt = new Point();
pt.x = 0;
pt.y = 0;
```
字段类型注释同样是可选的，但是没有指定类型，开启 [noImplicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) 配置将会报错，提示你这是隐性的 `any` 类型。
```ts
class Point {
    x 
   // Error：成员 'x' 具有隐性的 'any' 类型
}
```
字段同样有初始化程序,也就是默认值。当 class 实例化的时候将自动运行。
```ts
class Point {
    x = 0;
    y = 0;
}
const pt = new Point();
console.log(`${pt.x},${pt.y}`); // Prints 0,0
```
就像 `const`,`let`,和 `var` 一样，上例中，类属性的初始化程序将可以用于推断它的类型。
```ts
const pt = new Point();
pt.x = "0";
// Error: 'string' 类型不能赋值给 'number' 类型
```
#### --strictPropertyInitialization 配置
[strictPropertyInitialization](https://www.typescriptlang.org/tsconfig#strictPropertyInitialization) 设置能够控制类字段是否需要在构造函数中初始化。当 `strictPropertyInitialization` 设置为 `true` 时：
```ts
class UserAccount {
    name: string;
    accountType = "user";
    age!: number;
    email: string;
    //Error：'email' 属性没有初始化程序，并且没有在构造函数中明确赋值。
    address: string | undefined;
    constructor(name: string) {
        this.name = name;
    }
}
```
上面例子中：

- `name` 在构造函数有明确设置
- `accountType` 有设置默认值
- `age` 使用了非空断言操作符
- `address` 声明可能为 `undefined` 类型,意味着它可以不用在构造函数初始化
- `email` 以上情况都没有，所以得到一个错误

注意，字段只能在构造函数本身中初始化。TypeScript 不会分析你从构造函数调用的方法来检测初始化，因为派生类可能会覆盖这些方法使得初始化成员失败。

如果你打算通过构造函数以外的方式，明确的初始化一个字段（例如，也许一个外部库正在为你填充类的一部分），你可以使用明确的赋值断言运算符：`!`。
### Readonly 修饰符
字段可以用 `readonly` 修饰符作为前缀，这能阻止在构造函数外部对字段进行赋值。
```ts
class Greeter {
    readonly name: string = "word";
    constructor(otherName?: string) {
        if (otherName !== undefined) {
            this.name = otherName
        }
    }
    err() {
        this.name = "not ok";
        //Error： 不能赋值给 'name' ，因为它是只读属性。
    }
}
const g = new Greeter();
g.name = "also not ok";
// Error：不能赋值给 'name' ，因为它是只读属性。
```
### 构造函数
类构造函数和函数非常相似。你可以添加参数类型注释，默认值，和重载。
```ts
class Point {
    x: number;
    y: number;
    // 带默认值的普通签名
    constructor(x=0,y=0){
        this.x = x;
        this.y = y;
    }
}
```
```ts
class Point {
    // 重载
    constructor(x: number,y: string);
    constructor(s: string);
    constructor(xs: any,y?: any){
        // TBD
    }
}
```
类构造函数和函数之间的区别如下：

- 构造函数不能有类型参数-它们应在类外部声明（参考 [泛型类](#heading-29)）
- 构造函数不能返回注释类型-总是返回类实例类型

#### Super 的调用
和 JavaScript 一样，如果有基类，你需要在你派生类的构造函数体中，使用 `this.` 成员之前调用 `super()`：
```ts
class Base {
    k = 4;
}
 
class Derived extends Base {
    constructor() {
    // 在 ES5 中打印错误值；在 ES6 中抛出异常
    console.log(this.k);
    // Error：在派生类的构造函数中，'super' 必须在访问 'this' 之前调用
    super();
    }
}
```
在 JavaScript 中忘记调用 `super` 是一个容易犯的错误，但是 TypeScript 会在必要的时候告诉你。
### 方法
函数作为类的属性，称为方法。方法能使用与函数和构造函数相同的类型注释：
```ts
class Point {
    x = 10;
    y = 10;
    scale(n: number): void {
        this.x *= n;
        this.y *= n;
    }
}
```
除了标准类型注释，还有 默认值，重载，TypeScript 没有向方法添加任何其它新东西。

注意，方法和构造函数一样，不能有类型参数，类型参数应在类外部声明（参考 [泛型类](#heading-29)）。在方法体中，依然强制要求通过 `this.` 访问类字段和其它方法。没有使用 `this.` 的话访问的是作用域中的变量。
```ts
let x:number = 0;
class C {
    x: string = "hello";
    m() {
        // 方法 'm' 中没有变量 'x'，访问到了外部作用域(全局作用域)的变量 'x'
        x = "word";  
        // Error：'string' 类型不能赋值给 'number' 类型
    }
}
```
### 访问器 Getters / Setters

类同样也有访问器
```ts
class C {
    _length = 0;
    get length() {
        return this._length;
    }
    set length(value) {
        this._length = value;
    }
}
```
> 注意，如果不需要在 get/set 执行期间添加额外的逻辑，最好直接暴露为公共字段

TypeScript 对访问器有一些特殊的推断规则

- 如果只有 `get` 没有 `set`,那么属性变成 `readonly`
- 如果 setter 参数没有指定类型，那么会根据 getter 的返回值进行推断
- Getters 和 Setters 必须拥有一致的 [成员可见度](#heading-17)

自从 [TypeScript 4.3](https://devblogs.microsoft.com/typescript/announcing-typescript-4-3/)起,访问器 getters 和 setters 可以使用不同类型（意思就是访问器 setter 的参数类型不必和 getter 的返回类型一样，但是必须兼容 getter 的返回类型）
```ts
class Thing {
    _size = 0;
    get size(): number {
        return this._size;
    }
    
    set size(value: string | number | boolean) {
        let num = Number(value);
        // 不允许 NaN, Infinity, 等
        if (!Number.isFinite(num)) {
            this._size = 0;
            return;
        }
        this._size = num;
    }
}
```
### 索引签名
类能够用索引签名声明，与对象类型的 [索引签名](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures) 相同
```ts
class MyClass {
    [s: string]: boolean | ((s: string) => boolean);
    
    check(s: string) {
        return this[s] as boolean;
    }
}
```
因为索引签名类型还需要捕获方法的类型，有效的使用这些类型并不容易，最好将索引数据存储在其它地方，而不是类实例本身。
## Class 继承
像其它面向对象语言一样，JavaScript 的类也能继承自基类

### `implement` 关键字

你能使用 `implements` 关键字检查类是否满足特定的接口，如果类没能正确实现，将会发生错误
```ts
interface Pingable {
    ping(): void;
}
class Sonar implements Pingable {
    ping() {
        console.log("ping!");
    }
}
class Ball implements Pingable {
// Error：类 'Ball' 没有正确的实现 'Pingable' 接口。属性 'ping' 没有在类型 'Ball' 中，但是在 'Pingable' 接口中是必须的。
    pong() {
        console.log("pong!");
    }
}
```
类也能实现多个接口，例如：`class C implements A,B {}`

#### 注意事项

重要的是理解 `implements` 只检查类是否满足接口类型。不会改变类的类型或它的方法。常见的错误是认为 `implements` 会改变类类型：
```ts
interface Checkable {
    check(name: string): boolean;
}
class NameChecker implements Checkable {
    check(s) {
        //Error：参数 's' 拥有隐性的 'any' 类型
        return s.toLowercse() === "ok";
    }
}
```
在例子中，我们可能会认为 `s` 会受到 `name` 类型的影响：`check` 的参数为 `string`。其实不是，`implement` 不会改变类本身的检查方式或类型推断。

同样的，实现有可选属性的接口，并不会创建可选属性：
```ts
interface A {
    x: number;
    y?: number;
}
class C implements A {
    x = 0;
}
const c = new C();
c.y = 10;
// Error：属性 'y' 不存在类型 'C' 上
```
### `extends` 关键字
类可以从基类扩展，派生类拥有基类的所有属性和方法，还能定义额外成员。
```ts
class Animal {
    move() {
        console.log("Moving along!");
    }
}
class Dog extends Animal {
    woof(times: number) {
        for (let i = 0; i < times; i++) {
            console.log("woof!");
        }
    }
}
const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```

#### 覆盖方法

派生类也能够覆盖基类的字段和属性。你可以使用 `super.` 语法去访问基类的方法。注意，因为 JavaScript 类是一个简单的查找对象，这里没有 "super 字段" 的概念。

TypeScript 强制派生类总是基类的**子类型**

例如，下面覆盖方法是合法的：
```ts
class Base {
    greet() {
        console.log("Hello, world!");
    }
}
class Derived extends Base {
    greet(name?: string) {
        if (name === undefined) {
            super.greet();
        } else {
            console.log(`Hello, ${name.toUpperCase()}`);
        }
    }
}
const d = new Derived();
d.greet(); // Hello, world!
d.greet("reader"); // Hello, READER
```
通过基类去引用派生类实例是非常常见的
```ts
// 通过基类引用为派生类实例取别名
const b: Base = d;
b.greet();
```
派生类需要受到基类的限制，如果派生类没有遵循基类呢：
```ts
class Base {
    greet() {
        console.log("Hello, world!");
    }
}
class Derived extends Base {
    // 参数为必传的
    greet(name: string) {
    // 'Derived' 类型的 greet 属性不能赋值给基类 'Base' 类型的 greet 属性
    // (name: string) => void 类型不能赋值给 '() => void' 类型
        console.log(`Hello, ${name.toUpperCase()}`);
    }
}
```
如果错误了还编译这段代码，这个程序会崩溃:
```ts
const b: Base = new Derived();
// 因为 "name" 为 undefined 而崩溃
b.greet();
```

#### 纯类型字段声明

当 `tsconfig` 中 `target >= ES2022` 或 [useDefineForClassFields](https://www.typescriptlang.org/tsconfig#useDefineForClassFields) 为 `true` 时，类字段在父类构造函数完成后初始化，覆盖父类设置的任何值。当你只是想要对继承的字段重新声明更精确的类型时，这是一个问题。为了处理这些情况，你可以写 `declare` 来告诉 TypeScript 这个字段声明不应该有运行时作用。

```ts
interface Animal {
    dateOfBirth: any;
}

interface Dog extends Animal {
    breed: any;
}

class AnimalHouse {
    resident: Animal;
    constructor(animal: Animal) {
        this.resident = animal;
    }
}
class DogHouse extends AnimalHouse {
    // 没产生任何 JavaScript 代码
    // 只是确保了类型正确
    declare resident: Dog;
    constructor(dog: Dog) {
        super(dog);
    }
}
```
#### 初始化顺序
JavaScript 类的初始化顺序在某些例子中可能会让人惊讶，比如：
```ts
class Base {
    name = "base";
    constructor() {
        console.log("My name is " + this.name);
    }
}
class Derived extends Base {
    name = "derived";
}
// Prints "base", not "derived"
const d = new Derived();
```
这里发生了什么？初始化顺序为：

- 初始化基类字段
- 运行基类构造函数
- 初始化派生类字段
- 运行派生类构造函数

这意味着基类的构造函数看到的是自己的 `name` 值，因为派生类初始化还没有运行。

#### 继承内置类型

> 注意，如果你不打算继承内置类型，像 **Array**，**Error**，**Map**，等，或你编译目标明确设置为 ES6/ES2015 或更高，你可以跳过本部分。

在 ES6，构造函数返回一个对象，基类构造函数隐式的将 `this` 的值替换为 `super(...)` 的调用者。派生类构造函数必须捕获任何潜在的 `super(...)` 返回值，并将其替换为 `this`。

结果，子类化 `Error`，`Array` 和其它类，可能不像预期的那样工作。这是因为 `Error`、`Array` 等的构造函数使用了 ES6 的 `new.target` 去调整原型链。然而，当在 ES5 中调用构造函数时，并不能保证 `new.target` 的值。其它底层编译器通常具有相同的限制。

例如下面的子类在 ES5 中执行：
```ts
class MsgError extends Error {
    constructor(m: string) {
        super(m);
    }
    sayHello() {
        return "hello " + this.message;
    }
}
```
你会发现：

- 实例化 `MsgError`，实例化对象上的 `sayHello` 方法为 `undefiend`（`sayHello` 是原型方法），所以调用 `sayHello` 会出错。
- `MsgError` 和其实例之间的 `instanceof` 会断开，所以 `(new MsgError()) instanceof MsgError` 将返回 `false`。

当然你可以调用在 `super(...)` 之后立即手动调整原型
```ts
class MsgError extends Error {
    constructor(m: string) {
        super(m);
        // Set the prototype explicitly.
        Object.setPrototypeOf(this, MsgError.prototype);
    }
    sayHello() {
        return "hello " + this.message;
    }
}
```
但是，`MsgError` 的所有子类都得手动设置原型，而且运行时不支持 [Object.setPrototypeOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)，可以用 [\_\_proto\_\_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto) 替代。

但是，[IE10 及更低版本](https://learn.microsoft.com/en-us/archive/microsoft-edge/legacy/developer/?redirectedfrom=MSDN) 不支持这些修改方法，只能手动赋值原型方法到实例上（如：`MsgError.prototype` 到 `this` 上），但原型链本身无法修复。
## 成员可见度
你能使用 TypeScript 控制某个方法或属性在类外部代码是否可见

### `public`
默认类成员可见度是 `public`。`public` 成员可以在任何地方访问。
```ts
class Greeter {
    public greet() {
        console.log("hi!");
    }
}
const g = new Greeter();
g.greet();
```
因为 `public` 已经是默认的可见修饰符，所以你在类成员上不需要再显示编写该修饰符，但是为了风格和可读性，可以写上。

### `protected`
`protected` 成员只有在它们声明的类里和子类可见
```ts
class Greeter {
    public greet() {
        console.log("Hello, " + this.getName());
    }
    protected getName() {
        return "hi";
    }
}
class SpecialGreeter extends Greeter {
    public howdy() {
        // OK to access protected member here
        console.log("Howdy, " + this.getName());
    }
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
// Error：Property 'getName' is protected and only accessible within class 'Greeter' and its subclasses.
```
#### 暴露 `protected` 成员
派生类需要受基类限制，但是能选择暴露基类的子类型更多功能。包括把 `protected` 成员变成 `public`。
```ts
class Base {
    protected m = 10;
}
class Derived extends Base {
    //没有修饰器，所以默认是 'public'
    m = 15;
}
const d = new Derived();
console.log(d.m); // OK
```
注意，`Derived` 实例已经能够自由的读写属性 `m`，改变了 “安全性”。意味着在派生类中要注意 `protected` 成员的覆盖。如果要保持 "安全性" ，需重复 `protected` 修饰符。
```ts
class Base {
    protected m = 10;
}
class Derived extends Base {
    //没有修饰器，所以默认是 'public'
    protected m = 15;
}
const d = new Derived();
console.log(d.m);
// Error：Property 'm' is protected and only accessible within class 'Derived' and its subclasses.
```
#### 跨层次访问 `protected`
不同的 OOP 语言，对于基类是否能通过引用，访问 `protected` 成员存在分歧：
```ts
class Base {
    protected x: number = 1;
}
class Derived2 extends Base {
    f1(other: Derived2) {
        other.x = 10;
    }
    f2(other: Base) {
        other.x = 10;
        // Error：属性 'x' 是受保护的，只能通过 'Derived2' 类实例访问。这是一个 'Base' 类实例。
    }
}
```
Java 认为是合法的，C# 和 C++ 认为这代码是不合法的。

TypeScript 这里偏向 C# 与 C++，因为在 `Derived2` 中访问 `x` 应该只有 `Derived2` 子类才合法（）。例子中 `Base` 和 `Derived1` 是 `Derived2` 的父类，因此通过 `Base` 和 `Derived1` 的引用，访问 `x` 是非法的。

可参考 [为什么不能从派生类中访问 protected 成员](https://learn.microsoft.com/zh-cn/archive/blogs/ericlippert/why-cant-i-access-a-protected-member-from-a-derived-class) ，了解 c# 的推理。

### `private`
`private` 像 `protected`，但不允许子类访问成员：
```ts
class Base {
    private x = 0;
}
const b = new Base();
// 不能再类外部访问
console.log(b.x);
// Error：属性 'x' 是私有的，只能在 'Base' 类中能访问
```
```ts
class Derived extends Base {
    showX() {
        // 不能在子类访问
        console.log(this.x);
        // Error：property 'x' is private and only accessible within class 'Base'.
    }
}
```
因为 `private` 成员不能在派生类可见，所以派生类不能更改为可见：
```ts
class Base {
    private x = 0;
}
class Derived extends Base {
    // Error：'Derived' 类错误扩展了基类 'Base'。属性 'x' 在 'Base' 是私有的，但在 'Derived' 里不是。
    x = 1;
}
```
#### 跨实例访问 `private`
不同 OOP 语言对于，相同类不同实例是否可以互相访问 `private` 成员有不同的分歧。像 Java，C#，C++，Swift 和 PHP 允许，但是 Ruby 不允许。

TypeScript 允许跨实例访问 `private`
```ts
class A {
    private x = 10;
    public sameAs(other: A) {
        return other.x === this.x
    }
}
const a = new A();
const b = new A();
console.log(a.sameAs(b)) // true
```
### 警告
像 TypeScript 类型检查系统其它方面一样，`private` 和 `protected` [只在类型检查期间强制执行](https://www.typescriptlang.org/play?removeComments=true&target=99&ts=4.3.4#code/PTAEGMBsEMGddAEQPYHNQBMCmVoCcsEAHPASwDdoAXLUAM1K0gwQFdZSA7dAKWkoDK4MkSoByBAGJQJLAwAeAWABQIUH0HDSoiTLKUaoUggAW+DHorUsAOlABJcQlhUy4KpACeoLJzrI8cCwMGxU1ABVPIiwhESpMZEJQTmR4lxFQaQxWMm4IZABbIlIYKlJkTlDlXHgkNFAAbxVQTIAjfABrAEEC5FZOeIBeUAAGAG5mmSw8WAroSFIqb2GAIjMiIk8VieVJ8Ar01ncAgAoASkaAXxVr3dUwGoQAYWpMHBgCYn1rekZmNg4eUi0Vi2icoBWJCsNBWoA6WE8AHcAiEwmBgTEtDovtDaMZQLM6PEoQZbA5wSk0q5SO4vD4-AEghZoJwLGYEIRwNBoqAzFRwCZCFUIlFMXECdSiAhId8YZgclx0PsiiVqOVOAAaUAFLAsxWgKiC35MFigfC0FKgSAVVDTSyk+W5dB4fplHVVR6gF7xJrKFotEk-HXIRE9PoDUDDcaTAPTWaceaLZYQlmoPBbHYx-KcQ7HPDnK43FQqfY5+IMDDISPJLCIuqoc47UsuUCofAME3Vzi1r3URvF5QV5A2STtPDdXqunZDgDaYlHnTDrrEAF0dm28B3mDZg6HJwN1+2-hg57ulwNV2NQGoZbjYfNrYiENBwEFaojFiZQK08C-4fFKTVCozWfTgfFgLkeT5AUqiAA)。

这意味着 JavaScript 运行时，构造像 `in` 或简单的属性查找，仍能够访问 `private` 或 `protected` 成员。
```ts
class MySafe {
    private secretKey = 12345;
}
```
```
// 在 js 文件中...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
```
`private` 还允许在类型检查期间用方括号进行访问，使得 `private` 声明的字段可能更容易被单元测试之类的东西访问，缺点是这些字段是软私有的，并没有严格执行私有化。

```ts
class MySafe {
    private secretKey = 12345;
}
const s = new MySafe();
// 类型检查期间不能访问
console.log(s.secretKey);
// Error：property 'secretKey' is private and only accessible within class 'MySafe'。

// OK
console.log(s["secretKey"]);
```
不像 TypeScript 的 `private`，JavaScript 的 [private fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields)(`#`) 编译后仍然是私有的，并且不能像之前提到的用方括号访问，它们是硬私有的
```ts
class Dog {
    #barkAmount = 0;
    personality = "happy";
    constructor() {}
}
```
编译后
```ts
"use strict";
class Dog {
    #barkAmount = 0;
    personality = "happy";
    constructor() { }
}
```
当编译为 ES2021 或更低版本时，TypeScript 将使用 `WeakMap` 替代 `#`
```ts
"use strict";
var _Dog_barkAmount;
class Dog {
    constructor() {
        _Dog_barkAmount.set(this, 0);
        this.personality = "happy";
    }
}
_Dog_barkAmount = new WeakMap();
```
如果你需要保护类中的值不受恶意影响，应该使用运行时硬私有的机制，如，闭包，WeakMaps，或 私有字段。注意，添加这些私有化检查，运行期间可能会影响性能。

## 静态成员
类可以有静态成员，这些成员与类实例没有关联。它们可以通过类本身访问：
```ts
class MyClass {
    static x = 0;
    static printX() {
        console.log(MyClass.x);
    }
}
console.log(MyClass.x);
MyClass.printX();
```
静态成员同样能使用 `public`，`protected`，`private` 可见性修饰符：
```ts
class MyClass {
    private static x = 0;
}
console.log(MyClass.x);
// Error：Property 'x' is private and only accessible within class 'MyClass'.
```
静态成员也能继承：
```ts
class Base {
    static getGreeting() {
        return "Hello world";
    }
}
class Derived extends Base {
    myGreeting = Derived.getGreeting();
}
```
### 特殊静态名称
通常覆盖 `Function.prototype` 上的属性一般是不 安全的/合理的。因为类本身是一个可以用 `new` 调用的函数，所以某些静态名称不能使用，像函数属性 `name`，`length`，`call` 都不能定义成静态成员。
```ts
class S {
    static name = "S!";
    // Error：Static property 'name' conflicts with built-in property 'Function.name' of constructor function 'S'.
}
```
### 为什么没有静态类
TypeScript (和 JavaScript) 不像 C# 一样能构建 `static class`。之所以存在静态类，是因为这些语言强制所有数据和函数都在一个类中。而 TypeScript 不存在这个限制，所以不需要它们。

在 TypeScript 中不需要静态类语法，因为一个普通对象或顶阶函数也能做同样的工作：

```ts
class MyStaticClass {
    static doSomething() {}
}
function doSomething() {}
const MyHelperObject = {
    dosomething() {},
};
```
## 类中的静态块
静态块允许你编写自己作用域的语句序列，this 引用为类，并可以访问类中的静态私有字段。这意味着可以用写语句的方式来写初始化代码，不会泄露变量，并完全访问类的内部。
```ts
class Foo {
    static #count = 0;
    get count() {
        return Foo.#count;
    }
    static {
        try {
            const lastInstances = loadLastInstances();
            this.#count += lastInstances.length;
        }
        catch {}
    }
}
```
## 泛型类
类像接口一样也能用泛型。用 `new` 实例化泛型类的时候，类型参数的推断方法和函数调用相同：
```ts
class Box<Type> {
    contents: Type;
    constructor(value: Type) {
        this.contents = value;
    }
}
const b = new Box("hello!");
// b 类型：const b: Box<string>
```
类能可以像接口一样使用泛型约束和默认值。
### 在静态成员的类型参数
下面的代码是不合法的，并且原因可能不明显：
```ts
class Box<Type> {
    static defaultValue: Type;
    // Error：静态成员不能引用类的类型参数
}
```
记住，类型总是完全擦除的，运行时，例子中只有一个 `Box.defaultValue` 属性插槽。意味着设置 `Box<string>.defaultValue` (如果可能的话) 也能改为 `Box<number>.defaultValue`，这并不好。泛型类的静态成员永远不能引用类的类型参数。
## 运行时类中的 `this`
重要的是要记住 TypeScript 不会改变 JavaScript 的运行时行为，
某种程度上 JavaScript 以一些特殊的运行时行为而闻名。

JavaScript 对 `this` 处理确实不同寻常：
```ts
class MyClass {
    name = "MyClass";
    getName() {
        return this.name;
    }
}
const c = new MyClass();
const obj = {
    name: "obj",
    getName: c.getName,
};
// Prints "obj", not "MyClass"
console.log(obj.getName());
```
长话短说，默认情况下，函数内部 `this` 的值取决于函数的调用方式。在例子中，因为函数是通过 `obj` 引用调用的，`this` 的值是 `obj` 而不是类实例。

这可能不是你想要的！TypeScript 提供了一些方法来减轻或防止这类错误。
### 箭头函数
如果你函数调用经常丢失 `this` 上下文，可以使用箭头函数替换函数定义：
```ts
class MyClass {
    name = "MyClass";
    getName = () => {
        return this.name;
    };
}
const c = new MyClass();
const g = c.getName;
// Prints "MyClass" instead of crashing
console.log(g());
```
使用箭头函数，有以下一些取舍：
- `this` 保证运行时是正确的，但是 TypeScript 没有检查代码。
- 这将使用更多内存，因为每个类实例都有属于自己的函数定义副本（类初始化时使用箭头函数，只能定义在实例上）。
- 你不能在派生类使用 `super.getName`，因为 `super.getName` 是从原型链找的，而 `getName` 是定义在实例上的方法。
### `this` 参数
方法和函数有个初始化参数叫做 `this` ，对 TypeScript 来说的有特殊含义。而且这个参数在编译期间会被擦除。
```ts
// TypeScript 输入带有 'this' 参数
function fn(this: SomeType, x: number) {
    /* ... */
}
```
```js
// 编译后的 js
function fn(x) {
    /* ... */
}
```
TypeScript 检查带有 `this` 参数的函数调用是否使用了正确的上下文。不使用箭头函数，我们在方法定义添加一个 `this` 参数去强制方法被正确调用：
```ts
class MyClass {
name = "MyClass";
getName(this: MyClass) {
    return this.name;
}
}
const c = new MyClass();
// OK
c.getName();
// Error, 会崩溃
const g = c.getName;
console.log(g());
// Error：The 'this' context of type 'void' is not assignable to method's 'this' of type 'MyClass'.
```
这种方法和箭头函数相反：
- `this` 不能保证运行时是正确的，但是 TypeScript 会进行检查代码。
- 方法定义在类原型上，而不是每个类实例上
- 派生类可以通过 `super` 调用基类方法
## `this` 类型
在类中，有一种叫做 `this` 的特殊类型，动态引用当前类的类型。

让我们看看这有什么用:
```ts
class Box {
    contents: string = "";
    set(value: string) {
        // set 类型：(method) Box.set(value: string): this
        this.contents = value;
        return this;
    }
}
```
这里，TypeScript 推断 `set` 的返回类型为 `this` 而不是 `Box`。现在让我们创建 `Box` 的一个子类:
```ts
class ClearableBox extends Box {
    clear() {
        this.contents = "";
    }
}
const a = new ClearableBox();
const b = a.set("hello");
// b 类型：const b: ClearableBox
```
例子中，实例化子类 `ClearableBox`，并调用父类 `Box.prototype` 上的 `set` 方法，返回值 `this` 被推断为 `ClearableBox` 类型

你也可以在参数类型注释使用 `this`：
```ts
class Box {
    content: string = "";
    sameAs(other: this) {
        return other.content === this.content;
    }
}
```
这和编写 `other: Box` 不同-如果你有一个派生类，`sameAs` 方法现在只能接受同一派生类的实例对象：
```ts
class Box {
    content: string = "";
    sameAs(other: this) {
        return other.content === this.content;
    }
}
class DerivedBox extends Box {
    otherContent: string = "?";
}
const base = new Box();
const derived = new DerivedBox();
base.sameAs(derived); // OK
derived.sameAs(base);
// Error：
// 类型为 'Box' 的参数不能赋值给类型为 'DerivedBox' 的参数。
// 属性 "otherContent" 在类型 "Box" 中缺失，但在类型 "DerivedBox" 中是必须的。
```
例子中，`base.sameAs(derived)`，`this` 被推断为 `BOX`，`derived` 是 `DerivedBox` 的实例，类型为 `DerivedBox`，所以实例 `derived` 可以传递给 `sameAs` 方法。

而 `derived.sameAs(base)`，`this` 被推断为 `DerivedBox`，`base` 是 `Box` 的实例，类型为 `Box`，所以实例 `base` 传递给 `sameAs` 方法会抛出错误。

### `this`-基于类型保护
在类和接口中，方法的返回类型位置可以使用 `this is Type`，当与类型缩窄混合使用时（例如 if 语句），目标对象的类型将被缩窄为指定的类型。
```ts
class FileSystemObject {
    isFile(): this is FileRep {
        return this instanceof FileRep;
    }
    isDirectory(): this is Directory {
        return this instanceof Directory;
    }
    isNetworked(): this is Networked & this {
        return this.networked;
    }
    constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
    constructor(path: string, public content: string) {
        super(path, false);
    }
}

class Directory extends FileSystemObject {
    children: FileSystemObject[];
}

interface Networked {
    host: string;
}

const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

if (fso.isFile()) {
    fso.content;  // fso 类型：const fso: FileRep
} else if (fso.isDirectory()) {
    fso.children; // fso 类型：const fso: Directory
} else if (fso.isNetworked()) {
    fso.host; // fso 类型：const fso: Networked & FileSystemObject
}
```
一个基于 `this` 类型保护的常用用例，允许特定字段去延迟验证。例如，当 `hasValue` 被验证为真时，把 `box` 内的 `value` 移除 `undefined` 类型：
```ts
class Box<T> {
    value?: T;
    hasValue(): this is { value: T } {
        return this.value !== undefined;
    }
}
const box = new Box();
box.value = "Gameboy";
// box.value 类型：(property) Box<unknown>.value?: unknown
if (box.hasValue()) {
    box.value;
    // box.value 类型：(property) value: unknown
}
```

## 参数属性
TypeScript 提供了特殊的语法，用于将构造函数参数转换为具有相同名称和值的类属性。这些被称为参数属性，通过在构造函数参数前加可见性修饰符 `public`，`private`，`protected` 或 `readonly` 创建。
```ts
class Params {
    constructor(
        public readonly x: number,
        protected y: number,
        private z: number
    ) {
        // No body necessary
    }
}
const a = new Params(1, 2, 3);
console.log(a.x);
// a.x 类型：(property) Params.x: number
console.log(a.z);
// Error：Property 'z' is private and only accessible within class 'Params'.
```
## 类表达式
类表达式与类声明非常相似。但是类表达式不需要名字，我们可以通过它们最终绑定的任何标识符来引用它们:
```ts
const someClass = class<Type> {
    content: Type;
    constructor(value: Type) {
        this.content = value;
    }
};
const m = new someClass("Hello, world");
// m 类型：const m: someClass<string>
```
## 抽象类和成员
TypeScript 里的，类，方法，和字段可能是抽象的。

抽象方法或抽象字段没有提供实现。它们只能存在抽象类中，并且抽象类不能直接实例化。

抽象类的作用是作为基类，派生类要实现抽象类的所有抽象成员。当一个类没有任何抽象成员时，我们称它为具体类。

例子：
```ts
abstract class Base {
    abstract getName(): string;
    printName() {
        console.log("Hello, " + this.getName());
    }
}
const b = new Base();
// Error：不能创建抽象类的实例
```
我们不能用 `new` 实例化 `Base` ，因为它是抽象类。

我们需要创建一个派生类，并实现抽象类的所有抽象成员：
```ts
class Derived extends Base {
    getName() {
        return "world";
    }
}
const d = new Derived();
d.printName();
```
注意，如果派生类忘记实现抽象类的抽象成员，将会出错：
```ts
class Derived extends Base {
    // Error
}
```
### 抽象构造签名
有时你想接受一些类构造函数（该类继承自某个抽象类），并产生一个类实例。

你可能会写这种代码：
```ts
abstract class Base {
    abstract getName(): string;
    printName() {}
}
function greet(ctor: typeof Base) {
    const instance = new ctor();
     // Error：Cannot create an instance of an abstract class.
    instance.printName();
}
```
TypeScript 提示你，你正在尝试实例化一个抽象类。

而根据 `greet` 的定义，传入抽象类 `Base` 是完全合法的，这种写法很不好：
```ts
// Bad!
greet(Base);
```
更好的写法是 `greet` 接收带有构造签名的类型:
```ts
function greet(ctor: new () => Base) {
    const instance = new ctor();
    instance.printName();
}
greet(Derived);
greet(Base); 
// Error：
// Argument of type 'typeof Base' is not assignable to parameter of type 'new () => Base'. // Cannot assign an abstract constructor type to a non-abstract constructor type.
```
现在 TypeScript 正确地告诉你哪些类构造函数是可以被调用的—— `Derived` 可以，因为它是具体类，但 `Base` 不能。

### 抽象类继承抽象类

抽象类继承抽象类，两抽象类会进行合并。如果存在相同的抽象方法和字段，会有以下规则：

- 相同字段方法类型不同会报错
- 派生类相同字段方法类型为 `any`，则使用 `any`
- 派生类相同字段方法类型为父类相同字段方法类型的子类型，则使用子类型


```ts
abstract class Base {
  abstract getName(): string;
  abstract asd: string
  printName() {
    console.log("Hello, " + this.getName());
  }
}
abstract class Bag extends Base {
  abstract getName(): any;  // 相同抽象方法,返回值改为any，不报错
  abstract asd: '456' // 相同抽象字段，返回值改为 string 子类型，不报错
  abstract a: number
}
class Derived extends Bag {
  constructor(public asd: '456',public a:number) {
    super()
  }
  getName() {
    return ['World'];
  }
}
const d = new Derived('456',123);
d.printName();
```

## 类之间的关系
大多数情况下，TypeScript 中的类与其它类型一样，进行结构比较：

如下，两个类可以互相代替使用，因为它们结构是相同的:
```ts
class Point1 {
    x = 0;
    y = 0;
}
class Point2 {
    x = 0;
    y = 0;
}
// OK
const p: Point1 = new Point2();
```
类的子类型之间也是如此，即使它们没有显式继承：
```ts
class Person {
    name: string;
    age: number;
}
class Employee {
    name: string;
    age: number;
    salary: number;
}
// OK
const p: Person = new Employee();
```
这听起来很简单，但有一些其它奇怪的情况。

空类是没成员的，在结构类型系统中，没有成员的类型通常是其他类型的超类型。所以如果你写了一个空类，那么任何东西都可以用来代替它:
```ts
class Empty {}
function fn(x: Empty) {
    // 不能用 'x' 做任何事
}
// All OK!
fn(window);
fn({});
fn(fn);
```

感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/2/classes.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

