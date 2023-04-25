# Declaration Reference

当我们只有底层库的示例来指导我们时，我们经常面临编写声明文件的问题。本章主要针对的是 TypeScript 新手，可能还不熟悉 TypeScript 中的每个语言结构。教授如何编写高质量的定义文件。会模拟一些 API 的文档，以及该 API 的示例用法，并解释如何编写相应的声明。

这些例子的排列顺序是按照复杂度大致递增的。

## 具有属性的对象

*Documentation*

> 全局变量 myLib 有一个 makeGreeting 函数，用于创建问候语的，还有一个 numberOfGreetings 属性，表示到目前为止创建的问候语的数量。

*Code*

```ts
let result = myLib.makeGreeting("hello, world");
console.log("The computed greeting is:" + result);

let count = myLib.numberOfGreetings;
```

*Declaration*

使用 `declare namespace` 去描述通过点符号访问的类型或值。

```ts
declare namespace myLib {
    function makeGreeting(s: string): string;
    let numberOfGreetings: number;
}
```

## 重载函数

*Documentation*

> `getWidget` 函数接收一个数值并返回一个 `Widget` 类型，或接收一个字符串并返回一个 `Widget[]` 数组类型。

*Code*

```ts
let x: Widget = getWidget(43);

let arr: Widget[] = getWidget("all of them");
```

*Declaration*

```ts
declare function getWidget(n: number): Widget;
declare function getWidget(s: string): Widget[];
```

## 可复用类型（Interfaces）

*Documentation*

> 当指定问候语时，必须传递一个 `GreetingSettings` 对象。这个对象拥有以下属性：
> 
> 1 - greeting：字符串
> 
> 2 - duration：可选时间长度（毫秒）
> 
> 3 - color：可选字符串，例如：'#ff00ff'

*Code*

```ts
greet({
    greeting: "hello world",
    duration: 4000
});
```

*Declaration*

使用 `interface` 去定义带有属性的类型：

```ts
interface GreetingSettings {
    greeting: string;
    duration?: number;
    color?: string;
}

declare function greet(setting: GreetingSettings): void;
```

## 可复用类型（Type Aliases）

*Documentation*

> 在任何需要问候语的地方，你可以提供一个字符串、一个返回字符串的函数 或 一个 `Greeter` 实例。

*Code*

```ts
function getGreeting() {
    return "howdy";
}

class MyGreeter extends Greeter {}
greet("hello");
greet(getGreeting);
greet(new MyGreeter());
```

*Declaration*

你可以使用类型别名来简化类型：

```ts
type GreetingLike = string | (() => string) | MyGreeter;

declare function greet(g: GreetingLike): void;
```

## 组织类型

*Documentation*

> `greeter` 对象可以记录到文件或显示警报，你可以给 .log(...) 提供 `记录选项`，给 .alert(...) 提供警告选项

*Code*

```ts
const g = new Greeter("Hello");
g.log({ verbose: true });
g.alert({ modal: false, title: "Current Greeting" });
```

*Declaration*

使用命名空间去组织类型：

```ts
declare namespace GreetingLib {
    interface LogOptions {
        verbose?: boolean;
    }
    interface AlertOptions {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

你也可以在一个声明中创建嵌套的命名空间：

```ts
declare namespace GreetingLib.Options {
    // Refer to via GreetingLib.Options.Log
    interface Log {
        verbose?: boolean;
    }
    interface Alert {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

## 类

*Documentation*

> 你可以通过实例化 `Greeter` 对象创建一个迎宾器，或扩展 `Greeter` 创建一个自定义的迎宾器。

*Code*

```ts
const myGreeter = new Greeter("hello, world");
myGreeter.greeting = "howdy";
myGreeter.showGreeting();

class SpecialGreeter extends Greeter {
    constructor() {
        super("Very special greetings");
    }
}
```

*Declaration*

使用 `declare class` 去描述一个类或类似类对象。类可以具有属性和方法以及构造函数。

```ts
declare class Greeter {
    constructor(greeting: string);
    greeting: string;
    showGreeting(): void;
}
```

## 全局变量

*Documentation*

> 全局对象 foo 包含当前小部件的数量。

*Code*

```ts
console.log("Half the number of widgets is " + foo / 2);
```

*Declaration*

使用 `declare var` 去声明变量。如果变量是只读的，可以使用 `declare const`。如果变量是块级作用域，可以使用 `declare let`。

```ts
/** The number of widgets present */
declare var foo: number;
```

## 全局函数

*Documentation*

> 你可以使用字符串调用函数 `greet`，向用户显示问候语。

*Code*

```ts
greet("hello, world");
```

*Declaration*

使用 `declare function` 去声明函数。

```ts
declare function greet(greeting: string): void;
```

感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-files/by-example.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

