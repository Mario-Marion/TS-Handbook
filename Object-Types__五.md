---
highlight: vs2015
---

# Object Types
在 JavaScript 中，数据进行组合传递的基本方式是通过对象。在 TypeScript 中，我们用 `object` 类型来表示它们。

它们可以是匿名的：
```ts
function greet(person: { name: string; age: number }) {
    return "Hello " + person.name;
}
```
也可以通过使用接口来命名它们：
```ts
interface Person {
    name: string;
    age: number;
}
function greet(person: Person) {
    return "Hello " + person.name;
}
```
或类型别名：
```ts
type Person = {
    name: string;
    age: number;
};

function greet(person: Person) {
    return "Hello " + person.name;
}
```
在以上三个示例中，我们编写的函数接受一个对象，包含属性 `name`（必须是 `string` 类型）和 `age`（必须是 `number` 类型）。
## 属性修饰符
对象类型中的每个属性都可以指定一些东西：属性类型，属性是否是可选的，以及属性是否是可写的。

### 可选属性
我们可以通过在属性名称的末尾添加问号（`?`），将该属性标记为可选的：
```ts
interface PaintOptions {
    shape: Shape;
    xPos?: number;
    yPos?: number;
}

function paintShape(opts: PaintOptions) {
    // ...
}

const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```
在本例中，`xPos` 和 `yPos` 都是可选的。我们可以选择是否提供它们，因此以上对 `paintShape` 的调用都是有效的。所有的属性最好有指定类型，没指定类型将会是隐性的 `any` 类型，[noImplicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) 开启的情况下会报错。

当我们在 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 开启的情况下读取这些可选属性时，TypeScript 会告诉我们它们可能是 `undefined`。
```ts
function paintShape(opts: PaintOptions) {
    let xPos = opts.xPos; // 类型：(property) PaintOptions.xPos?: number | undefined
    let yPos = opts.yPos; // 类型：(property) PaintOptions.yPos?: number | undefined
}
```
在 JavaScript 中，即使属性不存在，我们仍然可以访问它，并且返回我们一个 `undefined`。为了排除 `undefined` 我们可以这么处理：
```ts
function paintShape(opts: PaintOptions) {
    let xPos = opts.xPos === undefined ? 0 : opts.xPos;
    // 类型：let xPos: number
    let yPos = opts.yPos === undefined ? 0 : opts.yPos;
    // 类型：let yPos: number
}
```
也可以设置默认值：
```ts
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
    console.log("x coordinate at", xPos);
    // xPos 类型：(parameter) xPos: number
    console.log("y coordinate at", yPos);
    // yPos 类型：(parameter) yPos: number
}
```
这里 `paintShape` 参数使用了 [解构模式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)，并且为 `xPos` 和 `yPos` 提供了默认值。现在 `xPos` 和 `yPos` 在 `paintShape` 函数体中不可能为 `undefined` 了。

> 请注意，目前没有办法在解构模式中放置类型注释。这是因为下面的语法在 JavaScript 中已经有了不同的含义。

```ts
function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
    render(shape);
    // Error：不能找到名称 'shape'，你是想找 'Shpae' 吗？
    render(xPos);
    // Error：不能找到名称 `xPos`
}
```
在对象解构模式中，`shape: Shape` 意味着定义了一个叫 `Shape` 的局部变量，值为参数的 `shape` 属性的值。

同样的，`xPos` 也意味着定义了一个叫 `number` 的局部变量，值为参数的 `xPos` 属性的值，如果没有 `xPos` 属性，则为默认值 `100`。

使用 [映射修饰符](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers) ，可移除 `?` 修饰符。

### `readonly` 属性
在 TypeScript 中属性可以被标记为 `readonly`。虽然它不会在运行时改变任何行为，但在类型检查期间，标记为 `readonly` 的属性将不能改写。

```ts
interface SomeType {
    readonly prop: string;
}

function doSomething(obj: SomeType) {
    // 可以读取 'obj.prop'.
    console.log(`prop has the value '${obj.prop}'.`);
    // 但不能重新赋值.
    obj.prop = "hello";
    // Error：不能给 'prop' 属性赋值，因为它为只读的属性。
}

```
使用 `readonly` 修饰符并不一定意味着值是完全不可变的，它只是意味着属性不能被重新赋值，和 `const` 常量类似。
```ts
interface Home {
    readonly resident: { name: string; age: number };
}

function visitForBirthday(home: Home) {
    // 可以读取和更新 'home.resident' 的属性
    console.log(`Happy birthday ${home.resident.name}!`);
    home.resident.age++;
}

function evict(home: Home) {
    //  但不能重新给 'home.resident' 属性赋值
    home.resident = {
    // Error：不能给 'resident' 属性赋值，因为它为只读的属性。
        name: "Victor the Evictor",
        age: 42,
    };
}
```
TypeScript 在检查两种对象类型是否兼容时，不会考虑这两种类型的属性是否为只读的，所以可通过以下方法绕过 TypeScript 检查：
```ts
interface Person {
    name: string;
    age: number;
}

interface ReadonlyPerson {
    readonly name: string;
    readonly age: number;
}

let writablePerson: Person = {
    name: "Person McPersonface",
    age: 42,
};

// works
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```
实际上变量 `readonlyPerson` 和 `writablePerson` 指向的是同个对象，`writablePerson` 改变对象属性值，`readonlyPerson` 自然跟着变。（TypeScript 检查类型是否见兼容，只检查类型结构是否相同，可参考 [类之间的关系](https://juejin.cn/post/7202898170613907517#heading-40)，不会考虑这两种类型的属性是否为只读的）

使用 [映射修饰符](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers) ，可移除 `readonly` 修饰符。

## 索引签名
有时候你不能提前知道对象属性的所有名称，但是你知道值的形状。

在这些情况下，你可以使用索引签名去描述，例如：
```ts
interface StringArray {
    [index: number]: string;
}
const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
// secondItem 类型：const secondItem: string
```
上面，我们有一个带有索引签名的 `StringArray` 接口。这个索引标签声明了，当 `StringArray` 索引为数值时，它将返回一个字符串。

索引签名属性允许类型为：`string`，`number`，`symbol`，模板字符串模式，以及由这些组成的联合类型。

注意！同时存在两种类型的索引器，是有可能的…

如下例子，同时存在 `number` 和 `string` 两种类型的索引器，但是  `number` 类型索引器的返回类型，必须是 `string` 类型索引器的返回类型的子类型（或同类型）。这是因为当索引为数值时，JavaScript 实际上会在索引进入对象之前将其转换为字符串。这意味着索引 `100` （`number`类型）与索引 `"100"`（`string`类型）是一样的。
```ts
interface Animal {
    name: string;
}

interface Dog extends Animal {
    breed: string;
}

interface NotOkay {
    [x: number]: Dog;
    [x: string]: Animal;
}
const arr: NotOkay = { 'asd': { name: 'ss' } };
// 注意，初始化时不能用数值，就算是 字符串数值（'0'），变量（为数值和字符串数值）也不行
arr[0] =  { name: 'ss',breed: 'str' }
```
存在索引签名，那么所有属性声明的返回类型，都要兼容索引签名的返回类型。如以下例子中，`name` 的类型与字符串索引的类型不匹配，类型检查器会给出一个错误：
```ts
interface NumberDictionary {
    [index: string]: number;
    length: number; // ok
    name: string;
    // Error：'name' 属性为 'string' 类型，不能赋值给字符串索引签名的属性类型 'number'
}
```
如果索引签名的返回类型为联合类型，则属性声明的返回类型就可以有多种类型：
```ts
interface NumberOrStringDictionary {
    [index: string]: number | string;
    length: number; // ok, length is a number
    name: string; // ok, name is a string
}
```
最后，你可以将索引签名设为只读，防止给属性重新赋值：
```ts
interface ReadonlyStringArray {
    readonly [index: number]: string;
}

let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
// Error：ReadonlyStringArray 中的索引签名是只读的
```
你不能设置 `myArray[2]`，因为索引签名是 `readonly`
## 扩展类型
类型可能是其他类型的更具体版本，是非常常见的。例如，我们可能有一个 `BasicAddress` 类型，它描述了在美国发送信件和包裹所需的字段
```ts
interface BasicAddress {
    name?: string;
    street: string;
    city: string;
    country: string;
    postalCode: string;
}
```
但是，一个地址上可能有门牌号。所以我们再声明一个接口 `AddressWithUnit`：
```ts
interface AddressWithUnit {
    name?: string;
    unit: string;  // 比 'BasicAddress' 多一字段
    street: string;
    city: string;
    country: string;
    postalCode: string;
}
```
这是没问题的，但缺点是：我们只是添加了一个新字段，就得必须重复 `BasicAddress` 中的所有其他字段。

其实我们可以扩展原始的 `BasicAddress` 类型，只添加 `AddressWithUnit` 中新增的字段。
```ts
interface BasicAddress {
    name?: string;
    street: string;
    city: string;
    country: string;
    postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
    unit: string;
}
```
`interface` 上的 `extends` 关键字帮助我们有效地复制其他命名类型成员，然后加入我们想要的任何新成员。减少了重复声明属性。如上面例子中，`AddressWithUnit` 不需要去重复 `street` 属性，因为可以使用 `BasicAddress` 类型 `street` 属性。

接口还可以一次扩展多个，并且可以扩展其它对象类型。
```ts
interface Circle {
  radius: number;
}
class Shape {
  constructor(public shape: string) { }
}
type Colorful = {
  color: string;
}

interface ColorfulCircle extends Circle, Shape, Colorful { }
const b: ColorfulCircle = {
  radius: 1,
  shape: 'str',
  color: 'str',
}
```
注意，扩展接口，存在同名属性时有以下几种情况：

1. 扩展接口（`ColorfulCircle`）和被扩展接口 （`Circle`）有同名属性，扩展接口同名属性类型必须是被扩展接口同名属性类型的**相同类型/子类型/缩窄类型**：

    Error ❌
    ```ts
    interface Circle {
      radius: number;
      color: string;
    }

    interface ColorfulCircle extends Circle {
        color: string | number;
    }
    // Error：
    // 接口 'ColorfulCircle' 错误的扩展了 'Circle' 接口。
    // 'color' 属性类型是不兼容的，类型 'string | number' 不能赋值给 'string' 类型
    // 类型 'number' 不能赋值给 'string' 类型
    ```
    Good ✔
    ```ts
    interface Circle {
      radius: number;
      color: string | number;
    }

    interface ColorfulCircle extends Circle {
        color: string;
    }
    ```
2. 多个被扩展接口 （`Circle`，`Colorful`）之间有同名属性，那么同名属性类型之间必须**相同**：

    Error ❌
    ```ts
    interface Circle {
      radius: number;
      color: string | number;
    }
    interface Colorful{
      color: string
    }

    interface ColorfulCircle extends Circle, Colorful { }
    // Error：
    // 接口 'ColorfulCircle' 不能同时扩展 'Circle' 和 'Colorful' 类型
    // 'Circle' and 'Colorful' 类型的 'color' 属性类型 不一致
    ```
    Good ✔
    ```ts
    interface Circle {
      radius: number;
      color: string | number;
    }
    interface Colorful{
      color: string | number;
    }

    interface ColorfulCircle extends Circle, Colorful { }
    ```
3. 如果多个被扩展接口 （`Circle`，`Colorful`）和扩展接口（`ColorfulCircle`）之间都有同名属性，那么被扩展接口同名属性类型之间**不必相同**，只要互相兼容即可。但是，扩展接口同名属性类型必须是**所有**被扩展接口同名属性类型的**相同类型/子类型/缩窄类型**：

    Error ❌
    ```ts
    interface Circle {
      radius: number;
      color: number;
    }
    interface Colorful {
      color: string | number;
    }

    interface ColorfulCircle extends Circle, Colorful {
      color: 'str'
    }
      // Error：
      // 'ColorfulCircle' 接口，不正确的扩展了 'Circle' 接口
      // 'color' 属性不兼容
      // 'str' 类型不能赋值给 'number' 类型
    ```
    `Circle` 和 `Colorful` 的 `color` 属性虽然兼容，但是扩展接口 `ColorfulCircle` 的 `color` 属性类型，并不是所有被扩展接口 `color` 属性的**相同类型/子类型/缩窄类型**，和 `Circle` 接口的 `color` 属性冲突了。
    
    Good ✔
    ```ts
    interface Circle {
      radius: number;
      color: string;
    }
    interface Colorful {
      color: string | number;
    }

    interface ColorfulCircle extends Circle, Colorful {
      color: 'str'
    }
    ```
## 交叉类型
上面 `extends` ，**作用于接口**上，用于**扩展对象类型**。

TypeScript 还提供了另一个称为 交叉类型 的结构，使用 `&` 操作符定义。**作用于类型别名**上，可用于**组合对象类型和原始类型**。

组合对象类型：
```ts
interface Circle {
  color: string;
}

class Shape {
  constructor(public shape: string) { }
}

type Colorful = {
  radius: number;
}

type ColorfulCircle = Circle & Shape & Colorful & {};
const obj: ColorfulCircle = {
  radius: 1,
  shape: 'str',
  color: 'str',
}
```
在这里，我们交叉了 `Colorful`，`Shape` 和 `Circle` 类型，生成了一个新类型，其中包含了 `Colorful`，`Shape` 和 `Circle` 的所有成员。

组合原始类型：
```ts
type A = string
type B = 'asd'
type C = A & B
```
`C` 为 `'asd'`，因为 `'asd'` 字符串字面量类型 为 `string` 的子类型。相当于从 `string` 类型缩窄成了 `'asd'` 字符串字面量类型。

> 注意，原始类型和对象类型不要进行组合
```ts
type Colorful = {
  radius: number;
}
type A = string
type ColorfulCircle = Colorful & A;

// --------cut-----------
const color:ColorfulCircle = 'asd';
// Error：
// 'string' 类型不能赋值给 'ColorfulCircle' 类型
// 'string' 类型不能赋值给 'Colorful' 类型

const objColor:ColorfulCircle = {radius: 23};
// Error：
// '{ radius: number; }' 类型不能赋值给 'ColorfulCircle' 类型
// '{ radius: number; }' 类型不能赋值给 'string' 类型
```
会出现很奇怪的现象，没报错，也不为 `never` 类型。既为对象类型也为 `string` 类型，无法赋值，因为 JavaScript 没有这种类型的值。


注意，**组合对象类型**时，存在同名属性，有以下几种情况：
1. 交叉类型（`ColorfulCircle`）与被交叉类型（`Circle`）存在同名属性，交叉类型的同名属性类型必须是被交叉类型同名属性类型的**相同类型/子类型/缩窄类型**：

    Bad ❌
    ```ts
    interface Circle {
      radius: number;
      color: string;
    }

    type ColorfulCircle = Circle & {
      color: number;
    }
    // 不报错！'color' 属性为 'never' 类型

    // ----------cut--------------
    const obj: ColorfulCircle = {
      radius: 123,
      color: 'str'
      // Error：'string' 类型不能赋值给 'never' 类型
    }
    ```
    例子中，`ColorfulCircle` 与 `Circle` 的同名属性 `color` 类型冲突了，但是没报错，`color` 属性会变为 `never` 类型，导致无法赋值。
    
    Good  ✔
    ```ts
    interface Circle {
      radius: number;
      color: string;
    }

    type ColorfulCircle = Circle & {
      color: 'str';
    }
    // ----------cut-------------- 
    const obj: ColorfulCircle = {
      radius: 123,
      color: 'str'
    }
    ```
2. 多个被交叉类型 （`Circle`，`Colorful`）之间有同名属性，那么同名属性类型之间必须**兼容**：
    
    Bad ❌
    ```ts
    interface Circle {
      color: boolean;
    }

    type Colorful = {
      radius: number;
      color: 'str' | number;
    }

    type ColorfulCircle = Circle & Colorful & {};
    // 不报错！所有属性都变成 `never` 类型
    // -------------cut--------------
    const obj: ColorfulCircle = {
      radius: 1,
      color: 'str',
    }
    // Error：
    // 'number' 类型不能赋值给 'never' 类型
    // 'string' 类型不能赋值给 'never' 类型
    ```
    所有属性都变成 `never` 类型
    
    Good  ✔
    ```ts
    interface Circle {
      color: string;
    }

    type Colorful = {
      radius: number;
      color: 'str' | number;
    }
    // -------------cut--------------
    type ColorfulCircle = Circle & Colorful & {};
    const obj: ColorfulCircle = {
      radius: 1,
      color: 'str',
    }
    ```
    因为 `'str'` 为 `string` 的子类型，所以 `'str' | number` 类型和 `string` 兼容，所以 `color` 属性为字面量字符串 `'str'`。

注意，**组合原始类型**，不兼容时，类型为 `never`：
```ts
type A = string
type B = number
type C = A & B
```
`A` 和 `B` 不兼容，所以 `C` 为 `never` 类型

## 扩展 VS 交叉
> 我们刚刚看到了两种组合类型的方法，它们很相似，但实际上有微妙的不同。对于接口，我们可以使用 `extends` 从其他类型进行扩展。对于类型别名，我们可以使用交叉进行类似的操作。两者之间的主要区别在于如何处理同名属性类型冲突，这种差异通常是选择接口扩展或类型别名交叉的主要原因之一。
## 泛型对象类型
假设有个 `Box` 类型，它能接收任意值：
```ts
interface Box {
    contents: any;
}
```
现在，`contents` 属性的类型是 `any`，但可能会导致后续事故（相当于没了类型检查）。

为了安全，我们可以使用 `unknown` 替代，但这意味着你已经知道了 `contents` 属性的类型，然后进行预防性检查，或者使用容易出错的类型断言。
```ts
interface Box {
    contents: unknown;
}

let x: Box = {
    contents: "hello world",
};

// 检查 'x.contents'
if (typeof x.contents === "string") {
    console.log(x.contents.toLowerCase());
}

// 或者使用类型断言
console.log((x.contents as string).toLowerCase());
```
另一种类型安全的方法是，构建多个不同的 `Box` 类型，并且 `contents` 属性类型也不同。
```ts
interface NumberBox {
    contents: number;
}

interface StringBox {
    contents: string;
}

interface BooleanBox {
    contents: boolean;
}
```
但这意味着我们必须创建不同的函数，或者重载函数来操作这些类型。
```ts
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
    box.contents = newContents;
}
```
这是一大堆引用。而且我们以后可能需要引入新的类型和重载，因为我们的盒子类型和重载都是一一对应的。

为了解决以上问题，我们可以创建一个泛型 `Box` 类型，它声明一个类型参数。
```ts
interface Box<Type> {
    contents: Type;
}
```
你可以把它理解为 "`Box` 的 `Type` 就是 `contents` 的 `Type`"。当我们引用 `Box` 时，我们必须给出一个类型参数来代替 `Type`。
```ts
let box: Box<string>;
```
可以把 `Type` 看作占位符，在上面例子中，当 TypeScript 看到 `Box<string>`，它将替换 `Box<Type>` 中的每一个 `Type` 为 `string`，所以 `Box<string>` 为 `{ contents: string }`。和之前 `StringBox` 类型是一样的。
```ts
interface Box<Type> {
    contents: Type;
}

interface StringBox {
    contents: string;
}

let boxA: Box<string> = { contents: "hello" };
boxA.contents; // 类型：(property) Box<string>.contents: string

let boxB: StringBox = { contents: "world" };
boxB.contents; // 类型：(property) StringBox.contents: string
```
`Box<Type>` 是可重用的，因为 `Type` 可以替换为任何类型。这意味着当我们需要一个新的 `Box` 类型时，我们根本不需要声明一个新的 `Box` 类型。
```ts
interface Box<Type> {
    contents: Type;
}

interface Apple {
    // ....
}

// 等同于 '{ contents: Apple }'.
type AppleBox = Box<Apple>;
```
这也意味着我们可以使用 [泛型函数](https://www.typescriptlang.org/docs/handbook/2/functions.html#generic-functions) 来替换重载函数。
```ts
function setContents<Type>(box: Box<Type>, newContents: Type) {
    box.contents = newContents;
}
```
类型别名也可以使用泛型：
```ts
type Box<Type> = {
    contents: Type;
};
```
`interface` 只能描述对象，类型别名不仅可以描述对象类型，还可以描述其它类型，所以可以使用它来编写其他类型的泛型类型。
```ts
type OrNull<Type> = Type | null;
type OneOrMany<Type> = Type | Type[];

type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
// OneOrManyOrNull 类型：type OneOrManyOrNull<Type> = OneOrMany<Type> | null

type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
// OneOrManyOrNull 类型：type OneOrManyOrNullStrings = OneOrMany<string> | null
```
### 数组类型
泛型对象类型通常是某种容器类型，其工作独立于它们包含的元素类型。数据结构以这种方式工作是非常理想的，这样它们就可以跨不同的数据类型重用。在本手册中，我们一直在使用类似的类型：`Array` 类型。

每当我们编写像 `number[]` 或 `string[]` 这样的类型时，实际上只是 `Array<number>` 和 `Array<string>` 的缩写。
```ts
function doSomething(value: Array<string>) {
    // ...
}

let myArray: string[] = ["hello", "world"];

doSomething(myArray);
doSomething(new Array("hello", "world"));
```
`Array` 本身也是一个泛型类型。
```ts
interface Array<Type> {
    /**
    * 获取或设置数组的长度。
    */
    length: number;
    
    /**
    * 移除数组最后一个元素并返回它
    */
    pop(): Type | undefined;
    
    /**
    * 将新元素追加到数组元素末尾，并返回数组的新长度。
    */
    push(...items: Type[]): number;
    // ...
}
```
现代 JavaScript 还提供了其它数据结构，TypeScript 也定义了相应的泛型，如 `Map<K, V>`，`Set<T>` 和 `Promise<T>`。由于 `Map`、`Set` 和 `Promise` 的行为方式，这意味着，它们可以处理任何类型集合。
### 只读数组类型
`ReadonlyArray` 是一种特殊类型，它描述了数组不应该被更改。
```ts
function doStuff(values: ReadonlyArray<string>) {
    // 我们可以读取 "values" 的元素
    const copy = values.slice();
    console.log(`The first value is ${values[0]}`);
    
    // 但不能改变 "values"
    values.push("hello!");
    // Error：属性 'push' 不存在 'readonly string[]' 类型上。
}
```
很像 `readonly` 属性修饰符。当我们看到函数返回 `ReadonlyArray`，就表示不需要改变返回值内容。而当我们看到函数参数为 `ReadonlyArray` 时，则表示，我们可以将任何数组传递给这个函数，而不用担心它会改变其内容。

与 `Array` 不同，`ReadonlyArray` 没有构造函数供我们使用。
```ts
new ReadonlyArray("red", "green", "blue");
// Error：'ReadonlyArray' 只是一个类型，但在这里被用作一个值。
```
我们可以将常规数组分配给 `ReadonlyArrays`。
```ts
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```
就像 TypeScript 为 `Type[]` 提供的简写语法：`Array<Type>` 一样，它同样为 `ReadonlyArray<Type>` 提供了简写语法 `readonly Type[]`
```ts
function doStuff(values: readonly string[]) {
    // 我们可以读取 "values" 的元素
    const copy = values.slice();
    console.log(`The first value is ${values[0]}`);
    
    // 但不能改变 "values"
    values.push("hello!");
     // Error：属性 'push' 不存在 'readonly string[]' 类型上。
}
```
最后需要注意的一点是，TypeScript 在检查两种对象类型是否兼容时，不会考虑这两种类型的属性是否为只读的（可回顾上面的 [readonly 属性](#heading-3)）。而常规的 `Array` 类型和 `ReadonlyArray` 类型之间的赋值不是双向的，也就是说数值检查兼容时是有考虑 `readonly` 的。
```ts
let x: readonly string[] = [];
let y: string[] = [];

x = y; // ok
y = x; // error
// Error：'readonly string[]' 类型是 'readonly'，不能赋值给可变的 'string[]' 类型。
```
可变数组可以赋值给只读数组，而反过来不行
### 元组类型
元组类型也是 `Array` 类型的一种，它明确知道数组包含多少元素，以及元素的位置和类型。
```ts
type StringNumberPair = [string, number];
```
例子中，`StringNumberPair` 是 `string` 和 `number` 元组类型。像 `ReadonlyArray` 一样，不会影响代码运行时行为，但对 TypeScript 来说很重要。在类型系统中，`StringNumberPair` 描述了一个数组，索引 `0` 为字符串元素，索引 `1` 为数值元素。
```ts
function doSomething(pair: [string, number]) {
    const a = pair[0]; // a 类型：const a: string
    const b = pair[1]; // b 类型：const b: number
    // ...
}
doSomething(["hello", 42]);
```
如果我们尝试索引超出元素的数量，就会得到一个错误。
```ts
function doSomething(pair: [string, number]) {
    // ...
    const c = pair[2];
    // Error：'[string, number]' 元组类型长度为 '2'，没有索引为 '2' 的元素
}
```
我们还可以使用 JavaScript 的 [数组解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring) 来解构元组。
```ts
function doSomething(stringHash: [string, number]) {
    const [inputString, hash] = stringHash;
    console.log(inputString); // inputString 类型：const inputString: string
    console.log(hash); // hash 类型：const hash: number
}
```
> 元组类型在大量基于约定的 API 中非常有用，其中每个元素的含义都是"明显的"。这给了我们在解构变量时任意命名的灵活性。在上面的例子中，我们可以将元素 **0** 和 **1** 命名为任何我们想要的名称。
>
然而，由于不是每个用户都持有相同的看法，因此可能需要考虑使用具有描述性属性名（见名知意）的对象是否适合你的 API。

抛开长度检查不谈，简单的元组类型等价于，指定元素为特定类型的数组类型，并使用数值字面量类型声明 `length` 属性，例如：
```ts
interface StringNumberPair {
    // 专门的属性
    length: 2;
    0: string;
    1: number;
    // 其它和 'Array<string | number>' 成员一样...
    slice(start?: number, end?: number): Array<string | number>;
}
```
元组可以通过修饰符 `?` 来声明可选属性（在元素类型之后附加 `?`）。可选的元组元素只能出现在末尾，并且会影响 `length` 的类型。
```ts
type Either2dOr3d = [number, number, number?, number? ];
function setCoordinate(coord: Either2dOr3d) {
    const [x, y, z] = coord;
    // z 类型：const z: number | undefined
    
    console.log(`Provided coordinates had ${coord.length} dimensions`);
    // length 类型：(property) length: 2 | 3 | 4
}
```
元组也可以使用"剩余"元素（和剩余参数类似），剩余元素的类型必须是 数组或元组类型：
```ts
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```
- `StringNumberBooleans` 描述了一个数组，前两个元素分别为 `string` 和 `number` 类型，但是后面可能有任意数量的 `boolean` 类型元素。
- `StringBooleansNumber` 描述了一个数组，第一个元素为 `string` 类型，然后后面可能有任意数量的 `boolean` 类型元素，最后还有一个 `number` 类型元素。
- `BooleansStringNumber` 描述了一个数组，开头可能有任意数量的 `boolean` 类型元素，然后最后还有一个 `string` 类型元素和一个 `number` 类型元素。

具有剩余元素的元组类型没有固定的"长度"——它只有一组位于不同位置的已知元素。
```ts
type StringNumberBooleans = [string, number, ...boolean[]];
// ---cut---
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```
可以用参数列表来对应元组。元组类型可用于 [剩余形参和实参](https://juejin.cn/post/7210369671663616055#heading-25)。因此，当你希望使用剩余参数去不限制参数的最大数量，只限制参数的最少数量，但又不想引入中间变量时，以下方法非常方便：
```ts
function readButtonInput(...args: [string, number, ...boolean[]]) {
    const [name, version, ...input] = args;
    // ...
}
```
相当于:
```ts
function readButtonInput(name: string, version: number, ...input: boolean[]) {
    // ...
}
```
### 只读元组类型
元组类型还有最后一个注意事项——元组类型有 `readonly` 元素，可以通过在它们前面附加一个 `readonly` 修饰符来指定——就像数组简写语法一样。
```ts
function doSomething(pair: readonly [string, number]) {
    // ...
}
```
TypeScript 不允许你改变 `readonly` 元组的任何属性。
```ts
function doSomething(pair: readonly [string, number]) {
    pair[0] = "hello!";
    // Error：不能给索引 '0' 赋值，因为它是只读属性
}
```
在大多数代码中，元组往往被创建而不被修改，因此'只读'元组是一个很好的默认值类型注释。还有一点也很重要，带有 `const` 断言的数组将被推断为'只读'字面量元组类型。
```ts
let point = [3, 4] as const;
function distanceFromOrigin([x, y]: [number, number]) {
    return Math.sqrt(x ** 2 + y ** 2);
}
distanceFromOrigin(point);
// Error：
// 'readonly [3,4]' 类型的实参不能赋值给 '[number, number]' 类型的行参。
// 'readonly [3,4]' 类型是只读的，并且不能赋值给可变的 '[number, number]' 类型
```
例子中，`distanceFromOrigin` 从不修改参数的元素，但期望接收一个可变的元组类型 `[number, number]`。由于 `point` 的类型被推断为 `readonly [3,4]`，是不可变的，所以与 `[number, number]` 不兼容。

感谢观看，如有错误，望指正

>官网地址： <https://www.typescriptlang.org/docs/handbook/2/objects.html>
>
>github 资料： <https://github.com/Mario-Marion/TS-Handbook>
