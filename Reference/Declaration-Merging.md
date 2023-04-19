# Declaration Merging

TypeScript 中的一些独特概念，在类型级别描述了 JavaScript 对象的形状。 TypeScript 特别独特的一个例子是 "声明合并" 的概念。理解这个概念将在使用现有 JavaScript 时更有优势。它也为更高级的抽象概念打开了大门。

这篇文章目的是介绍 "声明合并"，指编译器将两个具有相同名称的独立声明合并为一个定义。合并后的定义具有两个原始声明的特性。可以合并任意数量的声明；不只是局限于两个声明。

## Basic Concepts

在 TypeScript 中，声明至少在三个组之一创建实体：命名空间（namespace）、类型（type）或值（value）。命名空间创建声明会创建一个命名空间，其中包含的名称可以使用 点符号（.） 访问。类型创建声明是这样的：它们创建一个类型，该类型与声明的形状都是可见的，并绑定到给定的名称。最后，值创建声明会创建值，它在输出的 JavaScript 是可见的。

> 实体（Entity）：创建某种类型的实体叫做声明，如变量、函数、class、interface、enum、module 等关键字来定义。可以用来创建新的对象、定义接口和模块化代码等。

![Snipaste\_2023-04-17\_15-44-25.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/270f4c3734004b818c711d03171054ae\~tplv-k3u1fbpfcp-watermark.image?)

理解使用每个声明创建的内容，将有助于理解在执行声明合并时合并的内容。

## 合并接口

最简单，也可能是最常见的声明合并类型是接口合并。在最基本的级别上，合并机制将两个声明的成员连接到具有相同名称的单个接口中。

```ts
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

let box: Box = { height: 5, width: 6, scale: 10 };
```

接口的非函数成员应该是唯一的。如果它们不是唯一的，则它们必须是相同的类型。如果两个接口都声明了同名但类型不同的非函数成员，编译器将发出错误。

对于函数成员，每个同名的函数成员都被视为描述同一函数的重载。同样值得注意的是，在接口 `A` 与后面的接口 `A` 合并的情况下，第二个接口的优先级将高于第一个接口。

例子：

```ts
interface Cloner {
    clone(animal: Animal): Animal;
}

interface Cloner {
    clone(animal: Sheep): Sheep;
}

interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
}
```

这三个接口将合并创建一个单独的声明，如下所示：

```ts
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
    clone(animal: Sheep): Sheep;
    clone(animal: Animal): Animal;
}
```

请注意，每个组的元素保持相同的顺序，但是组本身与后面优先排序的'重载集'合并。简单说就是，合并的同名接口越晚声明，重载声明优先级更高。

这个规则的有一个例外，'专门化签名'。如果一个签名有一个参数，其类型是单个字符串字面量类型（例如：不是字符串字面量的联合类型），那么它将被冒泡到合并重载列表的顶部。

例子：

```ts
interface Document {
    createElement(tagName: any): Element;
}

interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
}

interface Document {
    createElement(tagName: string): HTMLElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

合并后的 `Document` 声明如下所示：

```ts
interface Document {
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
}
```

## 合并命名空间

在 TypeScript 中，命名空间是一种将相关代码分组到一个通用名称下的方法，类似于模块。但是，与模块不同的是，名称空间还会创建与名称空间同名的值。该值是一个对象，包含命名空间的所有成员，例如函数、变量和其他命名空间。

例子：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a08282845cb34bb58ccf6b9cdd9226fc\~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ff9f83af22a4f69a8977314ca14e8a4\~tplv-k3u1fbpfcp-watermark.image?)

> 类可作为 type 和 value，可参考之前图片

与接口类似，同名的名称空间也将合并其成员。因为声明命名空间时，创建名称空间和值，所以我们需要理解两者是如何合并的。

要合并命名空间，需要合并每个命名空间中**导出**接口声明的类型定义，从而形成一个单独的名称空间，其中包含合并的接口定义。

要合并命名空间值，在每个声明点上，如果已经存在具有给定名称的名称空间，则通过获取现有名称空间并将第二个名称空间的导出成员添加到第一个名称空间，进一步扩展该名称空间。

例子：

```ts
namespace Animals {
    export class Zebra {}
}

namespace Animals {
    export interface Legged {
        numberOfLegs: number;
    }
    export class Dog {}
}
```

等同于：

```ts
namespace Animals {
    export interface Legged {
        numberOfLegs: number;
    }
    
    export class Zebra {}
    export class Dog {}
}
```

但我们还需要了解非导出成员会发生什么。非导出成员仅在原始（未合并的）名称空间中可见。这意味着合并后，来自其他声明的合并成员不能看到未导出的成员。

例子：

```ts
namespace Animal {
    let haveMuscles = true;
    
    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

namespace Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles; Error：// because haveMuscles is not accessible here
    }
}
```

因为没有导出 `haveMuscles`，所以只有和同一命名空间的 `animalsHaveMuscles` 函数才能看到该符号。即使 `doAnimalsHaveMuscles` 函数它是合并的 `Animal` 名称空间的一部分，也不能看到这个未导出的成员。

## 合并类、方法和枚举命名空间

命名空间足够灵活，可以与其他类型声明合并。为此，命名空间声明必须紧跟与之合并的声明。生成的声明具有这两种声明类型的属性。TypeScript 使用这个功能去模仿 JavaScript 和其他编程语言中的一些模式。

## 合并类命名空间

这为用户提供了一种描述内部类的方法。

```ts
class Album {
    label: Album.AlbumLabel | undefined;
}

namespace Album {
    export class AlbumLabel {}
}
```

合并成员的可见规则和 [合并命名空间]() 部分中描述的相同，所以我们必须导出 `AlbumLable` 类，才能在合并的类中看到它。最终结果是在另一个类中管理一个类。你也可以使用命名空间向现有类添加更多静态成员。

除了内部类的模式之外，你可能还熟悉创建函数，然后通过向函数添加属性来进一步扩展函数，这样的 JavaScript 实践。TypeScript 使用声明合并，以类型安全的方式构建这样的定义。

```ts
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export let suffix = "";
    export let prefix = "Hello, ";
}

console.log(buildLabel("Sam Smith"));
```

同样的，命名空间可以用来扩展带有静态成员的枚举类型：

```ts
enum Color {
    red = 1,
    green = 2,
    blue = 4,
}

namespace Color {
    export function mixColor(colorName: string): number|void {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        } else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        } else if (colorName == "magenta") {
            return Color.red + Color.blue;
        } else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```

## 不允许合并

在 TypeScript 中并不是所有的合并都被允许。目前，类也可以和接口合并，但是不能与其它类或变量合并。有关模仿类合并的信息，请参阅 TypeScript 中的 [Mixins](https://juejin.cn/post/7222840421542133818) 部分。

## 模块扩展

尽管 JavaScript 模块不支持合并，但你可以通过导入并更新现有对象来给它们打补丁。

例子：

```ts
// observable.ts
export class Observable<T> {
    // ... implementation left as an exercise for the reader ...
}

// map.ts
import { Observable } from "./observable";
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
};
Error：// Property 'map' does not exist on type 'Observable<any>'.
```

这在 TypeScript 中也能正常工作，但编译器不知道 `Observable.prototype.map`。你可以使用模块扩展来告诉编译器：

```ts
// observable.ts
export class Observable<T> {
    // ... implementation left as an exercise for the reader ...
}

// map.ts
import { Observable } from "./observable";
declare module "./observable" {
    interface Observable<T> {
        map<U>(f: (x: T) => U): Observable<U>;
    }
}
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
};

// consumer.ts
import { Observable } from "./observable";
import "./map";
let o: Observable<number> = new Observable();
o.map((x) => x.toFixed());
```

模块名的解析方式与 `import`/`export` 中的模块说明符相同。可参阅 [模块](https://www.typescriptlang.org/docs/handbook/modules.html) 了解更多信息。合并扩展中的声明，就像它们在原始文件中声明一样。

有两个限制要记住：

1.  不能在扩展中声明新的顶级声明 —— 只能对现有声明进行补丁。
2.  默认的导出也不能扩展，只能扩展命名的导出（因为你需要通过导出的名称来扩展一个导出，而 `default` 是一个保留字 —— 详细信息请参阅 [#14080](https://github.com/Microsoft/TypeScript/issues/14080)）
    ```ts
    // observable.ts
    export default class Observable<T> {
        // ... implementation left as an exercise for the reader ...
    }

    // map.ts
    import Observable from "./observable";
    declare module "./observable" {
        interface Observable<T> {
            map<U>(f: (x: T) => U): Observable<U>;
        }
    }
    Observable.prototype.map = function (f) {
        // ... another exercise for the reader
    };
    Error：// Property 'map' does not exist on type 'Observable<any>'.

    // consumer.ts
    import Observable from "./observable";
    import "./map";
    let o: Observable<number> = new Observable();
    o.map((x) => x.toFixed());
    Error：// Property 'map' does not exist on type 'Observable<any>'.
    ```

## 全局扩展

你也可以在模块内部向全局作用域添加声明：

```ts
// observable.ts
export class Observable<T> {
    // ... still no implementation ...
}

declare global {
    interface Array<T> {
        toObservable(): Observable<T>;
    }
}

Array.prototype.toObservable = function () {
    // ...
};
```

全局扩展拥有和模块扩展相同的行为和限制。

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-merging.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
