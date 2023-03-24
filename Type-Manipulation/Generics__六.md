---
highlight: vs2015
---
# Generics
软件工程的最主要部分是构建组件，这些组件不仅要有定义明确且一致的 API，还要能复用。组件既能处理当前数据又能处理将来的数据，将为你构建大型软件系统提供最灵活性的能力。

在 C# 和 Java 等语言中，用于创建可复用组件的工具箱中，主要工具之一是泛型，也就是说，创建的组件可以处理多种类型而不是单一类型。允许用户使用这些组件时使用他们自己的类型。
## 泛型之 Hello World
假设有一个 `identity` 函数，它会返回传入的任何东西。

如果没有泛型，我们要么必须给 `identity` 函数一个特定的类型:
```ts
function identity(arg: number): number {
    return arg;
}
```
或者，我们可以使用 `any` 类型：
```ts
function identity(arg: any): any {
    return arg;
}
```
虽然使用 `any` 肯定是通用的，但会导致函数接受 `arg` 的任意类型，丢失了函数返回时的类型信息。比如我们传入一个数值，那么只会返回 `any` 类型，并不知道函数返回值类型信息。

我们需要一种方法来捕获参数的类型，这样我们就可以用它来表示返回什么类型。在这里，我们将使用类型变量，这是一种特殊的变量，它作用于类型而不是值。
```ts
function identity<Type>(arg: Type): Type {
    return arg;
}
```
现在，我们向 `identity` 函数添加了一个类型变量 `Type`。这个 `Type` 允许我们捕获用户提供的类型，以便我们以后使用该信息。这里，我们再次使用 `Type` 作为返回类型。通过检查，我们现在可以看到参数和返回类型是相同的类型。这允许我们将该类型信息在函数中使用。

这个版本的 `identity` 函数也是通用的，它适用于任何类型。与使用 `any` 不同（即，它不会丢失任何信息），它与第一个使用数值作为参数和返回类型的 `identity` 函数一样精确。

泛型 `identity` 函数，我们可以有两种方式来调用它。第一种方法是将所有参数传递给函数，包括类型参数：
```ts
let output = identity<string>("myString");
// output 类型：let output: string
```
这里显式地将 `Type` 设置为 `string` ，作为函数调用的参数之一，在参数周围使用 `<>` 而不是 `()` 表示。

第二种方式也是很常见的。使用类型参数推断——编译器会根据我们传入的参数类型自动为我们设置 `Type` 的值：
```ts
let output = identity("myString");
```
注意，我们不必显式地在尖括号中传递类型(`<>`)；编译器只查看值 `"myString"`，并将 `Type` 设置为它的类型。虽然类型参数推断有助于保持代码更简短、可读性更高，但当编译器无法推断类型时（在更复杂的示例中可能会发生这种情况），那么你就需要使用第一种调用方式了。
## 使用泛型类型变量
当你开始使用泛型时，你会注意到，创建像 `identity` 这样的泛型函数时，编译器会强制你正确使用函数体中的任何泛型类型参数。你实际上可以将这些参数视为任意类型。

让我们用之前的 `identity` 函数：
```ts
function identity<Type>(arg: Type): Type {
    return arg;
}
```
如果我们想在每次调用时，记录参数 `arg` 的长度到控制台呢？我们可能会这样写：
```ts
function loggingIdentity<Type>(arg: Type): Type {
    console.log(arg.length);
    // Error：'length' 属性不存在 'Type' 类型上
    return arg;
}
```
当我们这样做时，编译器会给我们一个错误：`arg` 参数没有 `length` 成员。请记住，我们前面说过，这些类型变量代表任意类型，因此调用者可能传入一个没有 `length` 成员的 `number` 类型。

我们假设这个函数处理的是 `Type` 元素的数组类型，而不是直接处理 `Type` 类型。由于我们处理的 `arg` 为数组类型，那么 `.length` 成员是可用的。例子：
```ts
function loggingIdentity<Type>(arg: Type[]): Type[] {
    console.log(arg.length);
    return arg;
}
```
你可以将 `loggingIdentity` 的类型解读为："泛型函数 `loggingIdentity`，接受一个类型参数 `Type` 和一个 `arg` 参数（类型为 `Type` 元素数组），并返回一个 `Type` 元素数组类型。" 如果我们传入一个数值数组，那么函数会返回一个数值数组，因为 `Type` 类型参数会绑定为 `number`。这允许我们使用泛型类型变量 `Type` 作为我们正在处理的类型的一部分，而不是整个类型，给我们提供了更大的灵活性。

上面例子的另一种写法:
```ts
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
    console.log(arg.length);
    return arg;
}
```
你可能已经在其他语言中熟悉了这种类型风格。在下一节中，我们将介绍如何创建自己的泛型类型，如 `Array<Type>`。
## 泛型类型
在前面的小节中，我们创建了适用任意类型的泛型 `identity` 函数。在本节中，我们将探讨函数本身的类型以及如何创建泛型接口。

泛型函数就像非泛型函数声明一样，只是多个了类型参数：
```ts
function identity<Type>(arg: Type): Type {
    return arg;
}
let myIdentity: <Type>(arg: Type) => Type = identity;
```
泛型函数也可以写成对象字面量类型的调用签名：
```ts
function identity<Type>(arg: Type): Type {
    return arg;
}
let myIdentity: { <Type>(arg: Type): Type } = identity;
```
泛型函数也可以声明成接口：
```ts
interface GenericIdentityFn {
    <Type>(arg: Type): Type;
}
function identity<Type>(arg: Type): Type {
    return arg;
}
let myIdentity: GenericIdentityFn = identity;
```
> 之所以能声明成对象，是因为函数也是对象，可参考 [函数调用签名](https://juejin.cn/post/7210369671663616055#heading-2)

我们可能希望将泛型参数移动为整个接口的参数。这可以让我们看到哪些类型是泛型的（例如 `Dictionary<string>` 而不仅仅是 `Dictionary`）。也使得类型参数对接口的所有其他成员可见。
```ts
interface GenericIdentityFn<Type> {
    (arg: Type): Type;
}
function identity<Type>(arg: Type): Type {
    return arg;
}
let myIdentity: GenericIdentityFn<number> = identity;
```
请注意，现在我们没有了泛型函数，而是有一个非泛型函数签名，它是泛型类型的一部分。当我们使用 `GenericIdentityFn` 时，我们现在还需要指定相应的类型参数（这里是 `number`），锁定下面的调用签名将使用什么类型。理解何时将类型参数直接放在调用签名上，和何时将它放在接口本身上，将有助于描述类型的哪些方面是泛型的。

除了泛型接口，我们还可以创建泛型类。注意，不能创建泛型枚举和命名空间。

## 泛型类
泛型类与泛型接口有相似的形状。泛型类的类名后面跟随一个在尖括号（`<>`），其中是泛型类型参数列表。
```ts
class GenericNumber<NumType> {
    zeroValue: NumType;
    add: (x: NumType, y: NumType) => NumType;
}
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
    return x + y;
};
```
这是对 `GenericNumber` 类的字面用法，但你可能已经注意到，没有限制它只能使用 `number` 类型。我们可以使用 `string` 或者更复杂的对象。
```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
return x + y;
};
console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```
就像接口一样，将泛型类型参数放在类本身上，可以确保类的所有属性都可以使用泛型参数类型。

需要注意的是：泛型类的类型参数仅作用于实例成员，和原型成员，不能使用于静态成员。（可参考 [泛型类](https://www.typescriptlang.org/docs/handbook/2/classes.html#generic-classes)）

## 泛型约束
如果你还记得前面的示例，你可能有时想要编写一个使用于类型集的泛型函数，你了解该类型集将具有哪些功能。但是在我们的 `loggingIdentity` 示例中，我们希望能够访问参数 `arg` 的 `length` 属性，但是编译器觉得不是每种类型都有 `length` 属性，所以它警告我们不能做这样的假设。
```ts
function loggingIdentity<Type>(arg: Type): Type {
    console.log(arg.length);
    // length 属性不存在 'Type' 类型上
    return arg;
}
```
我们希望约束这个函数来处理具有 `length` 属性的任意类型，而不是使用任意类型。只要类型有这个成员，我们就允许它传入。所以，我们必须将 `Type` 约束为我们需要的类型范围。

在这里，我们将创建一个有 `length` 属性的接口，然后使用该接口和 `extends` 关键字来约束泛型类型参数 `Type`：
```ts
interface Lengthwise {
    length: number;
}
function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
    console.log(arg.length); // Now we know it has a .length property, so no more error
    return arg;
}
```
由于泛型函数类型参数现在受到约束，它将不再适用于任意类型：
```ts
loggingIdentity(3);
// Error：'number' 类型的实参不能赋值给 'Lengthwise' 类型的行参
```
我们需要传入符合 `Lengthwise` 类型的值：
```ts
loggingIdentity({ length: 10, value: 3 });
```

## 在泛型约束中使用类型参数
泛型中的类型参数可以受到另一个类型参数的约束。例如，这里我们想从给定名称的对象中获取一个属性。为了确保不会获取一个不存在 `obj` 上的属性，我们需要在这两个类型参数之间放置一个约束：
```ts
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
    return obj[key];
}
let x = { a: 1, b: 2, c: 3, d: 4 };
getProperty(x, "a");
getProperty(x, "m");
// Error：'m' 类型的实参不能赋值给 '"a" | "b" | "c" | "d"' 类型的行参
```
## 在泛型中使用类类型
在 TypeScript 中使用泛型创建工厂函数时，需要通过构造函数来引用类类型。例如：
```ts
function create<Type>(c: { new (): Type }): Type {
    return new c();
}
```
一个更高级的示例，使用 prototype 属性来推断和约束构造函数和类类型的实例之间的关系。
```ts
class BeeKeeper {
    hasMask: boolean = true;
}
class ZooKeeper {
    nametag: string = "Mikle";
}
class Animal {
    numLegs: number = 4;
}
class Bee extends Animal {
    keeper: BeeKeeper = new BeeKeeper();
}
class Lion extends Animal {
    keeper: ZooKeeper = new ZooKeeper();
}
function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}
createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```
该模式用于支持 [mixins](https://www.typescriptlang.org/docs/handbook/mixins.html) 设计模式


感谢观看，如有错误，望指正

>官网地址： <https://www.typescriptlang.org/docs/handbook/2/generics.html>
>
>github 资料： <https://github.com/Mario-Marion/TS-Handbook>

