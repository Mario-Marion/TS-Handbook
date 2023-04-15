# Decorators

随着 TypeScript 和 ES6 中引入类，现在存在一些场景，需要额外的特性来支持注释或修改 类 和 类成员。装饰器提供了一种方法，为类声明和成员添加注释和元编程语法。装饰器是 JavaScript 的 [第二阶段提案](https://github.com/tc39/proposal-decorators)，并且是 TypeScript 的实验性特性。

> 注意，装饰器是一个实验性的特性，在将来的版本中可能会改变。

要启用对装饰器的实验性支持，必须在命令行或 `tsconfig.json` 中启用 [experimentalDecorators](https://www.typescriptlang.org/tsconfig#experimentalDecorators) 编译器选项:

**Command Line**

```shell
tsc --target ES5 --experimentalDecorators
```

**tsconfig.json**

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

## 装饰器

装饰器是一种特殊的声明类型，它可以附加到 [class declaration](https://www.typescriptlang.org/docs/handbook/decorators.html#class-decorators), [method](https://www.typescriptlang.org/docs/handbook/decorators.html#method-decorators), [accessor](https://www.typescriptlang.org/docs/handbook/decorators.html#accessor-decorators), [property](https://www.typescriptlang.org/docs/handbook/decorators.html#property-decorators), or [parameter](https://www.typescriptlang.org/docs/handbook/decorators.html#parameter-decorators)。装饰器使用 `@expression` 语法，`expression` 必须求值为一个函数，该函数将在运行时调用，并获取被装饰的声明的信息。

例如，给定装饰器 `@sealed`，我们可能如下编写 `sealed`：

```ts
function sealed(target) {
    // do something with 'target' ...
}
```

## 装饰器工厂

如果我们想要自定义一个装饰器应用于声明上，我们可以编写装饰器工厂。装饰器工厂是一个函数，它将返回一个表达式，在运行时被装饰器去调用。

我们可以按照以下方式编写装饰器工厂：

```ts
function color(value: string) {
    // this is the decorator factory, it sets up
    // the returned decorator function
    return function (target) {
        // this is the decorator
        // do something with 'target' and 'value'...
    };
}
```

## 装饰器位置

多个装饰器可以应用在一个声明，并且编写在同一行：

```ts
@f @g x
```

或多行写法：

```ts
@f
@g
x
```

当多个装饰符应用于单个声明时，它们的求值类似于 [数学中的函数组合](https://en.wikipedia.org/wiki/Function_composition)。在这个模型中，当 *f* 和 *g* 构成函数时，产生的组合 ( f ∘ g )( x ) 等效于 f( g( x ) )。

因此，在 TypeScript 中，单个声明上有多个装饰器，装饰器求值时会用以下步骤执行：

1.  每个装饰器的表达式都从上到下求值。
2.  然后将结果作为函数从下到上调用。

我们可以观察以下例子的求值顺序：

使用 [装饰器工厂](https://juejin.cn/post/7221750009140707365#heading-2)：

```ts
function first() {
    console.log("first(): factory evaluated");
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("first(): called");
    };
}

function second() {
    console.log("second(): factory evaluated");
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("second(): called");
    };
}

class ExampleClass {
    @first()
    @second()
    method() {}
}
```

打印输出到控制台：

```ts
first(): factory evaluated
second(): factory evaluated
second(): called
first(): called
```

使用装饰器：

```ts
function first(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("first(): called");
};

function second(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("second(): called");
};

class ExampleClass {
    @first
    @second
    method() {}
}
```

打印输出到控制台：

```ts
second(): called
first(): called
```

## 装饰器执行顺序

对于应用于类内部各种声明的装饰器，有以下顺序：

1.  实例成员，应用 (参数装饰器然后方法装饰器) 或 访问器 或 属性装饰器
2.  静态成员，应用 (参数装饰器然后方法装饰器) 或 访问器 或 属性装饰器
3.  构造函数，应用参数装饰器
4.  类，应用类装饰器

> 注意点
>
> 应用在同一方法中的装饰器，参数装饰器总是在方法装饰器之前调用
>
> 参数修饰器中，越往后的参数修饰器，越早调用。
> 
> 构造函数应用参数装饰器，参数装饰器第二个参数（方法名）为 undefined

例子太长，可到 [Playground](https://www.typescriptlang.org/zh/play?experimentalDecorators=true#code/GYVwdgxgLglg9mABBANgQwM4YCIFMJwBOaURAFAWBlISNEQFyIBi408YAlIgN4BQiZAgxwUuAHQo4AczIAiVJhz4iJInM4BuPgF9toSLASIADoTgnchKAE88BYqUJkwaALa4m1QjDDTu-IKEuFAghEgG7MZkUGiE0iFMaGA2ADSm5pbWNgDSuDZeNL7+vAKCQlSiElKyrh5aZTq6fJFGSCZx7iFW9qpOLl2FPn4BZcGh4YitHIgxcQlQSSnp0wgAcoOI3sWIAD6I4AAmuMC+uIfpvscAHkxgIG4ARlaj5RUiYpIyA-Xagk1NVZIDxQAAWcEOvUc5Dqni2RRGpSCITCETYbVmsXiiUQyTSGQsVlseQK8OG0nSxwwEB8JicTAACpkiXZcNTaU5XuVKB9qt9YQ1-tpAeiZmgIBA2SJCFC1M5YUNilzxqipqLoliFkt8WZCdkSYq-JS2TSYHTGIgmXrbHh2WbOUjucIql9al1BYgAXxvQABRRYWVOPj+jCIACi13cJjEAGF0FhSmUeTQ6P0fR1iCCeipoc4FMIU-RCAzOlnCBpTCBHigYBBcSYTABlKwAN1rcO2RsQ6dL3RlObl8mTtCLJczfYATBXpMEOwiSoFymCYBhxDPcLhEABeRDr3CNb2CH0g8GQgf9OTUEi1gCyIVPGjKV9gdYwJ4hZB746g2Ycg4A5M+tZjmW-7cC2aAoCAc7kgEnplD64qSlgRCBuQl6xC+ACCEpSuo3BAXWCyINcZBwSqkwAIwTgAzPBR66lkNrnuhhFWkxNgVoRiAwIcdwPM8hCHt2jEsmhea+FekC4OxLLTrOsB+Ia0jbog-57op0j-ghSF4f2f4XpJsTSThyHStOIQkWRvCIBRSDUXRTRHu+Z4GehRnJJKd5ghCj6CC5ZAQVBMHFOkX5dD++l9OQ-4edJIF9mBiBBdBE7KXBTQIaJ2TifIcWSrJ2RTtwe7Kc0YC4AA7uGkZuNGuBxkoZD-mgDbNoQbaSv+6TqbOYHaEAA) 参考
## 类装饰器

类装饰器在类声明之前声明。类装饰器应用于类的构造函数，并且用于观察、修改或替换类定义。类装饰器不能在声明文件 或 任何其他环境上下文中使用（例如在 `declare` 类上）。

类装饰器表达式，将在运行时作为函数被调用，被装饰的类的构造函数将作为其唯一参数。

如果类装饰器返回一个构造函数，那么它将替换类声明：

```ts
function reportableClassDecorator<T extends { new(...args: any[]): {} }>(constructor: T) {
  // 手动继承 BugReport 类，装饰器不会为你做这些
  return class extends constructor {
    reportingURL = "http://www...";
  };
}

@reportableClassDecorator
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}

const bug = new BugReport("Needs dark mode");
console.log(bug.title); // Prints "Needs dark mode"
console.log(bug.type); // Prints "report"

bug.reportingURL;
Error：// Property 'reportingURL' does not exist on type 'BugReport'.
```

> 注意
> 
> 这并不是装饰器工厂，装饰器工厂返回的是装饰器
> 
> 如果类装饰器返回一个新的构造函数，则必须注意维护原型链。在运行时应用装饰器的逻辑**不会**为你做这些。
> 
> 装饰器并不改变 TypeScript 类型，因此，对类型系统来说，新属性 `reportingURL` 是不存在的


下面是类（`BugReport`）应用的类装饰器（`@sealed`）的例子：

```ts
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}

@sealed
class BugReport {
    type = "report";
    title: string;
    constructor(t: string) {
        this.title = t;
    }
}
```

当 `@sealed` 被执行时，它将同时密封构造函数和它的原型，因此，将阻止运行时通过访问 `BugReport.prototype` 向该类添加或从移除的任何进一步功能，或者在 `BugReport` 类本身定义属性（注意，ES2015 的类，实际上只是基于原型的构造函数语法糖）。此装饰器不阻止类继承 `BugReport`。

```ts
BugReport.prototype.props = "props"
TS Error：// Property 'props' does not exist on type 'BugReport'.
JS Error：// Cannot add property asd, object is not extensible

BugReport.staticProps = 'staticProps'
TS Error：// Property 'staticProps' does not exist on type 'typeof BugReport'.
JS Error：// Cannot add property staticProps, object is not extensible

class SubClas extends BugReport { } // OK
```

## 方法装饰器

方法装饰器在方法声明之前声明，该装饰器应用于方法的属性描述符，并且用于观察，修改，或替换方法定义。方法装饰器不能在声明文件、重载 或 任何其他环境上下文中使用（例如在 `declare` 类上）。

方法装饰器表达式，将在运行时作为函数被调用，并且有以下三个参数：

1.  应用于静态成员时，为类的构造函数。 应用于实例成员时，为类的原型
2.  成员名称
3.  成员的属性描述符

> 注意，如果 tsconfig.json 中设置 target 低于 ES5 版本，那么属性描述符将为 undefined

如果方法装饰器有返回值，它将被用作方法的属性描述符：
```ts
function enumerable(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.log(descriptor)
  // {
  //   "writable": true,
  //   "enumerable": false,
  //   "configurable": true
  // } 
  return {
    "writable": false,
    "enumerable": false,
    "configurable": false
  }
};

class Greeter {
  constructor() { }
  @enumerable
  greet() { }
}

console.log(Object.getOwnPropertyDescriptor(Greeter.prototype, 'greet'))
// {
//   "writable": false,
//   "enumerable": false,
//   "configurable": false
// } 
```

> 注意，如果 tsconfig.json 中设置 target 低于 ES5 版本，那么返回值将被忽略

下面是类方法（`greet`）应用方法类装饰器（`@enumerable`）的例子：

```ts
function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}

class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }

  @enumerable(true)
  greet() {
    return "Hello, " + this.greeting;
  }
}
```

这里的 `@enumerable(false)` 装饰器是一个 [装饰器工厂](https://juejin.cn/post/7221750009140707365#heading-2)。当调用 `@enumerable(false)` 装饰器时，它会修改方法的属性描述符，`enumerable` 属性。

## 访问装饰器

访问装饰器在访问器声明之前声明，该装饰器应用于访问器的属性描述符，并且用于观察，修改，或替换访问器的定义。访问装饰器不能在声明文件 或 任何其他环境上下文中使用（例如在 `declare` 类上）。

> TypeScript 不允许装饰 get 和 set 访问器单个成员。而且，该成员的所有装饰器都必须应用于文档顺序中指定的第一个访问器。这是因为装饰器应用于属性描述符，它结合了 get 和 set 访问器，而不是分别为单独的声明。

例子：

**bad ❌**
```ts
function accessors(target: any, propertyKey: string, descriptor: PropertyDescriptor) { };

class Point {
  @accessors
  get x() {
    return 'asd'
  }
  @accessors
  set x(x) { }
  Error：// Decorators cannot be applied to multiple get/set accessors of the same name.
}
```
**Great ✔**
```ts
function accessors(target: any, propertyKey: string, descriptor: PropertyDescriptor) { };

class Point {
  @accessors
  get x() {
    return 'asd'
  }
  set x(x) { }
}
// 或
class Point2 {
  get x() {
    return 'asd'
  }
  @accessors
  set x(x) { }
}
```



访问装饰器表达式，将在运行时作为函数被调用，并且有以下三个参数：

1.  应用于静态成员时，为类的构造函数。 应用于实例成员时，为类的原型
2.  成员名称
3.  成员的属性描述符

> 注意，如果 tsconfig.json 中设置 target 低于 ES5 版本，那么属性描述符将为 undefined

如果访问装饰器有返回值，它将被用作成员的属性描述符：
```ts
function accessors(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.log(descriptor)
  // {
  //   "set": undefined,
  //   "enumerable": false,
  //   "configurable": true
  // } 
  return {
    "enumerable": false,
    "configurable": false
  }
};

class Point {

  @accessors
  get x() {
    return 'asd'
  }
}
console.log(Object.getOwnPropertyDescriptor(Point.prototype, "x"))
// {
//   "set": undefined,
//   "enumerable": false,
//   "configurable": false
// } 
```

> 注意，如果 tsconfig.json 中设置 target 低于 ES5 版本，那么返回值将被忽略

下面是类成员（`x` 和 `y`）应用访问装饰器（`@configurable`）的例子：

```ts
function configurable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.configurable = value;
  };
}

class Point {
  private _x: number;
  private _y: number;
  constructor(x: number, y: number) {
    this._x = x;
    this._y = y;
  }

  @configurable(false)
  get x() {
    return this._x;
  }

  @configurable(false)
  set y(num: number) {
    this._y = num
  }
}
```

## 属性装饰器

属性装饰器在属性声明之前声明。属性装饰器不能在声明文件 或 任何其他环境上下文中使用（例如在 `declare` 类上）。

属性装饰器表达式，将在运行时作为函数被调用，并且有以下两个参数：

1.  应用于静态成员时，为类的构造函数。 应用于实例成员时，为类的原型
2.  成员名称

> 注意，由于 TypeScript 中属性装饰器的初始化方式，属性描述符不会作为属性装饰器的参数。这是因为在定义 prototype 的成员时，目前没有描述实例属性的机制，也没有方法来观察或修改属性的初始化。返回值也会被忽略。因此，属性装饰器只能用于观察类是否声明了某个属性。

如下例子，我们可以使用这些信息来记录关于属性的元数据：

```ts
import "reflect-metadata";
const formatMetadataKey = Symbol("format");
function format(formatString: string) {
  return Reflect.metadata(formatMetadataKey, formatString);
}
function getFormat(target: any, propertyKey: string) {
  return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}

class Greeter {
    @format("Hello, %s")
    greeting: string;
    
    constructor(message: string) {
        this.greeting = message;
    }

    greet() {
        let formatString = getFormat(this, "greeting");
        return formatString.replace("%s", this.greeting);
    }
}
```

`@format("Hello, %s")` 装饰器是一个装饰器工厂。当 `@format("Hello, %s")` 被调用，它使用 `reflect-metadata` 库的 `Reflect.metadata` 方法为属性添加元数据。当 `getFormat` 被调用，它会获取并格式化属性元数据。

> 注意，本例需要使用 `reflect-metadata` 库。有关 `reflect-metadata` 库的更多信息，请参阅 [Metadata](https://juejin.cn/post/7221750009140707365#heading-10)。

## 参数装饰器

参数装饰器在参数声明之前声明，该装饰器应用于类构造函数或方法声明。参数装饰器不能在声明文件、重载 或 任何其他环境上下文中使用（例如在 `declare` 类上）。

参数装饰器表达式，将在运行时作为函数被调用，并且有以下三个参数：

1.  应用于静态成员时，为类的构造函数。 应用于实例成员时，为类的原型
2.  方法名称
3.  函数参数列表中该参数索引。

> 参数装饰器只能用于观察方法是否声明了某个参数。

参数装饰器的返回值会被忽略

下面是类成员（`print`）应用参数装饰器（`@required`）的例子：

```ts
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }

  @validate
  print(@required verbose: boolean) {
    if (verbose) {
      return `type: ${this.type}\ntitle: ${this.title}`;
    } else {
      return this.title;
    }
  }
}
```

以下是 `@required` 参数装饰器和 `@validate` 方法装饰器的定义：

```ts
import "reflect-metadata";
const requiredMetadataKey = Symbol("required");
 
function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  let existingRequiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
  existingRequiredParameters.push(parameterIndex);
  Reflect.defineMetadata( requiredMetadataKey, existingRequiredParameters, target, propertyKey);
}
 
function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<Function>) {
  let method = descriptor.value!;
 
  descriptor.value = function () {
    let requiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyName);
    if (requiredParameters) {
      for (let parameterIndex of requiredParameters) {
        if (parameterIndex >= arguments.length || arguments[parameterIndex] === undefined) {
          throw new Error("Missing required argument.");
        }
      }
    }
    return method.apply(this, arguments);
  };
}
```

`@required` 装饰器添加元数据，将参数标记为必需的。然后`@validate` 装饰器将现有的 `greet` 方法包装在一个函数中，该函数会在调用原始方法之前验证参数。

> 注意，本例需要使用 `reflect-metadata` 库。有关 `reflect-metadata` 库的更多信息，请参阅 [Metadata](https://juejin.cn/post/7221750009140707365#heading-10)。

## 其它

以下写法是不合法的：

```ts
function parameterDecorator (target: any, functionName: string, index: number) {
    console.log(functionName);
}
class ExampleClass {
  method2 = function (@parameterDecorator value: string) { }
  
  Error：// Decorators are not valid here.
}
new ExampleClass();
```

构造函数只能使用参数装饰器：

```ts
function methodDecorator(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.log(target);
};

class ExampleClass {
    @methodDecorator
    constructor(public appService: string) { }
    
    Error：// Decorators are not valid here.
}
new ExampleClass('appService');
```

## 元数据

一些例子使用了 `reflect-metadata` 库，它为 [实验性的元数据API](https://github.com/rbuckton/reflect-metadata) 添加了一个 polyfill。这个库还不是 ECMAScript (JavaScript) 标准的一部分。然而，一旦装饰器被正式采用为 ECMAScript 标准的一部分，这些扩展将被提议采用。

可以通过 npm 下载这个库：

```shell
npm i reflect-metadata --save
```

TypeScript 包含该实验性支持，可以为带有装饰器的声明发出特定类型的元数据。要启用这种实验性支持，你必须在命令行或 `tsconfig.json` 中设置 [emitDecoratorMetadata](https://www.typescriptlang.org/tsconfig#emitDecoratorMetadata) 编译器选项:

**Command Line**

```shell
tsc --target ES5 --experimentalDecorators --emitDecoratorMetadata
```

**tsconfig.json**

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}
```

启用后，只要导入了 `reflect-metadata` 库，就会在运行时暴露额外的设计时类型信息。

例如：

```ts
import "reflect-metadata";
 
class Point {
  constructor(public x: number, public y: number) {}
}
 
class Line {
  private _start: Point;
  private _end: Point;
 
  @validate
  set start(value: Point) {
    this._start = value;
  }
 
  get start() {
    return this._start;
  }
 
  @validate
  set end(value: Point) {
    this._end = value;
  }
 
  get end() {
    return this._end;
  }
}
 
function validate<T>(target: any, propertyKey: string, descriptor: TypedPropertyDescriptor<T>) {
  let set = descriptor.set!;
  
  descriptor.set = function (value: T) {
    let type = Reflect.getMetadata("design:type", target, propertyKey);
 
    if (!(value instanceof type)) {
      throw new TypeError(`Invalid type, got ${typeof value} not ${type.name}.`);
    }
 
    set.call(this, value);
  };
}
 
const line = new Line()
line.start = new Point(0, 0)
 
// @ts-ignore
// line.end = {}
 
// Fails at runtime with:
// > Invalid type, got object not Point
```

TypeScript 编译器将使用 `@Reflect.metadata` 装饰器注入设计时类型信息。你可以认为它等同于以下 TypeScript：

```ts
class Line {
  private _start: Point;
  private _end: Point;
  @validate
  @Reflect.metadata("design:type", Point)
  set start(value: Point) {
    this._start = value;
  }
  get start() {
    return this._start;
  }
  @validate
  @Reflect.metadata("design:type", Point)
  set end(value: Point) {
    this._end = value;
  }
  get end() {
    return this._end;
  }
}
```
> 注意，装饰器元数据是一个实验性的特性，在将来的版本中可能会引入破坏性的更改。

感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/decorators.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
