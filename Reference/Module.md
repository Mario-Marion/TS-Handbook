# Modules

从 ECMAScript 2015 开始，JavaScript 有了模块的概念，TypeSCRipt 也有这个概念。

模块运行它们自己的作用域，而不是全局作用域；这意味着在模块中的声明的变量，函数，类，等等，外部是无法访问的，除非明确使用了 [`export` forms](#heading-1) 其中的一种形式导出它们。相反，要使用来自不同模块导出的变量，函数，类，接口，等等。使用 [`import` forms](#heading-5) 其中的一种形式去导入。

模块是声明性的；模块之间的关系在文件级别通过导入和导出来指定。

模块之间使用模块加载器导入。在运行时，模块加载器负责在模块执行之前，定位和执行模块的所有依赖项。JavaScript 中常用的模块加载器有 Node.js 的 [CommonJS](https://wikipedia.org/wiki/CommonJS) 模块加载器 和 Web 应用程序中 [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md) 模块的 [RequireJS](http://requirejs.org/) 加载器。

在 TypeScript 中，就像在 ECMAScript 2015 中一样，任何包含顶级 `import` 或 `export` 的文件都被视为一个模块。相反，没有任何顶级 `import` 或 `export` 声明的文件被视为一个脚本，其内容在全局范围内可用（因此也适用于模块）。

## Export

### 导出声明

任何声明（例如变量、函数、类、类型别名或接口）都可以通过添加 `export` 关键字来导出。

**StringValidator.ts**

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

**ZipCodeValidator.ts**

```ts
import { StringValidator } from "./StringValidator";
export const numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
```

### 导出语句

当需要重命名导出时，可以使用导出语句，所以上面的例子可以写成：

```ts
class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```

### 重新导出

通常模块会扩展其他模块，并导出它们的部分功能。重新导出不会在本地导入它，也不会引入局部变量。

**ParseIntBasedZipCodeValidator.ts**

```ts
export class ParseIntBasedZipCodeValidator {
  isAcceptable(s: string) {
    return s.length === 5 && parseInt(s).toString() === s;
  }
}
// 直接导出并重命名
export { ZipCodeValidator as RegExpBasedZipCodeValidator } from "./ZipCodeValidator";
```

一个模块可以包装一个或多个模块，并使用 `export * from "module"` 语法组合它们的所有导出。

**AllValidators.ts**

```ts
export * from "./StringValidator"; // 导出 'StringValidator' 接口
export * from "./ZipCodeValidator"; // 导出 'ZipCodeValidator' 类 and 'numberRegexp' 常量值
export * from "./ParseIntBasedZipCodeValidator"; //  导出 'ParseIntBasedZipCodeValidator' 类
// 和重新导出，别名为 'RegExpBasedZipCodeValidator'，来自 'ZipCodeValidator.ts' 模块的 'ZipCodeValidator' 类
```

## import

导入与从模块导出一样简单。例如：

```ts
import { ZipCodeValidator } from "./ZipCodeValidator";
let myValidator = new ZipCodeValidator();
```

### 从模块导入单个导出

```ts
import { ZipCodeValidator } from "./ZipCodeValidator";
let myValidator = new ZipCodeValidator();
```

导入也可以重命名

```ts
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```

### 将整个模块导入到单个变量中。并且使用它访问模块导出

```ts
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```

### 只导入模块副作用

虽然不是很推荐的这种做法，但某些模块会设置一些可供其它模块使用的全局状态。这些模块可能没有任何导出，或者使用者对这些模块的任何导出都不感兴趣。要导入这些模块，请使用：

```ts
import "./my-module.js";
```

### 导入类型

在 TypeScript 3.8 版本之前可以使用 `import` 导入类型。 TypeScript 3.8 及之后可以使用 `import` 语句或使用 `import type` 导入类型。

```ts
// 导入值
import { APIResponseType } from "./api";
// 导入类型
import type { APIResponseType } from "./api";
// 同时导入 (getResponse) 值和 (APIResponseType) 类型 
import { getResponse, type APIResponseType} from "./api";
```

确保从你的 JavaScript 中擦除任何显式标记的 `type` 导入，并且像 Babel 这样的工具，可以通过设置 tsconfig 中的 [isolatedModules]() 标志，对你的代码做出更好的假设。可以在 [3.8 发布说明](https://devblogs.microsoft.com/typescript/announcing-typescript-3-8-beta/#type-only-imports-exports) 中阅读更多内容。

使用 TypeScript 4.5，可以在单个命名导入上使用类型修饰符。

```ts
import { someFunc, type BaseType } from "./some-module.js";
```

## 默认导出

每个模块都可以选择导出默认导出。默认导出使用关键字 `default` 标记；每个模块只能有一个默认导出。默认导出是使用不同的导入方式导入的。

默认导出真的很方便。例如，像 jQuery 这样的库可能默认导出 `jQuery` 或 `$`，我们如下导入。

**[JQuery.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/jquery/JQuery.d.ts)**

```ts
declare let $: JQuery;
export default $;
```

**App.ts**

```ts
import $ from "jquery";
$("button.continue").html("Next Step...");
```

类和函数声明可以直接编写为默认导出。默认导出的类和函数声明名称是可选的。

**ZipCodeValidator.ts**

```ts
export default class ZipCodeValidator {
    static numberRegexp = /^[0-9]+$/;
    isAcceptable(s: string) {
        return s.length === 5 && ZipCodeValidator.numberRegexp.test(s);
    }
}
```

**Test.ts**

```ts
import validator from "./ZipCodeValidator";
let myValidator = new validator();
```

或

**StaticZipCodeValidator.ts**

```ts
const numberRegexp = /^[0-9]+$/;
export default function (s: string) {
    return s.length === 5 && numberRegexp.test(s);
}
```

**Test.ts**

```ts
import validate from "./StaticZipCodeValidator";
let strings = ["Hello", "98052", "101"];
// Use function validate
strings.forEach((s) => {
    console.log(`"${s}" ${validate(s) ? "matches" : "does not match"}`);
});
```

默认导出也可以只是值：

**OneTwoThree.ts**

```ts
export default "123";
```

**Log.ts**

```ts
import num from "./OneTwoThree";
console.log(num); // "123"
```

## Export all as x

在 TypeScript 3.8 中，可以使用 `export * as ns` 的简写形式重新导出另一个具有名称的模块：

```ts
export * as utilities from "./utilities";
```

这从模块中获取所有依赖项并使其成为导出字段，可以如下导入它：

```ts
import { utilities } from "./index";
```

## `export =` 和 `import = require()`

CommonJS 和 AMD 通常都有一个 `exports` 对象的概念，它了包含一个模块的所有导出。

他们还支持用自定义单个对象替换 `exports` 对象。默认导出旨在替代此行为；然而，两者是不相容的。 TypeScript 支持 `export =` 来模拟传统的 CommonJS 和 AMD 工作流程。

`export =` 语法指定从模块导出的单个对象。可以是类、接口、命名空间、函数或枚举。

使用 `export =` 导出模块时，必须使用 TypeScript 的特有语法 `import module = require("module")` 来导入模块。

**ZipCodeValidator.ts**

```ts
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

**Test.ts**

```ts
import zip = require("./ZipCodeValidator");

let strings = ["Hello", "98052", "101"];

let validator = new zip();

strings.forEach((s) => {
    console.log(
    `"${s}" - ${validator.isAcceptable(s) ? "matches" : "does not match"}`
    );
});
```

## 模块代码的生成

根据编译期间指定的模块目标，编译器将为 Node.js ([CommonJS](http://wiki.commonjs.org/wiki/CommonJS))、require.js ([AMD](https://github.com/amdjs/amdjs-api/wiki/AMD))、[UMD](https://github.com/umdjs/umd)、[SystemJS](https://github.com/systemjs/systemjs) 或 [ECMAScript 2015 原生模块](http://www.ecma-international.org/ecma-262/6.0/#sec-modules) (ES6) 模块加载系统生成适当的代码。有关生成代码中的 `define`、`require` 和 `register` 调用的更多信息，请参阅每个模块加载器的文档。

这个简单的例子展示了，在导入和导出过程中使用的名称是如何被转换成模块加载代码的。

**SimpleModule.ts**

```ts
import m = require("mod");
export let t = m.something + 1;
```

**AMD / RequireJS SimpleModule.js**

```ts
define(["require", "exports", "./mod"], function (require, exports, mod_1) {
    exports.t = mod_1.something + 1;
});
```

**CommonJS / Node SimpleModule.js**

```ts
var mod_1 = require("./mod");
exports.t = mod_1.something + 1;
```

**UMD SimpleModule.js**

```ts
(function (factory) {
  if (typeof module === "object" && typeof module.exports === "object") {
    var v = factory(require, exports);
    if (v !== undefined) module.exports = v;
  } else if (typeof define === "function" && define.amd) {
    define(["require", "exports", "./mod"], factory);
  }
})(function (require, exports) {
  var mod_1 = require("./mod");
  exports.t = mod_1.something + 1;
});
```

**System SimpleModule.js**

```ts
System.register(["./mod"], function (exports_1) {
  var mod_1;
  var t;
  return {
    setters: [
      function (mod_1_1) {
        mod_1 = mod_1_1;
      },
    ],
    execute: function () {
      exports_1("t", (t = mod_1.something + 1));
    },
  };
});
```

**原生 ECMAScript 2015 模块 SimpleModule.js**

```ts
import { something } from "./mod";
export var t = something + 1;
```

## 简单示例

下面，我们整合了前面示例中使用的 Validator 实现，只从每个模块导出一个命名的导出。

要编译，我们必须在命令行上指定一个模块目标。对于 Node.js，使用 `--module commonjs`；对于 require.js，使用 `--module amd`。例如:

```shell
tsc --module commonjs Test.ts
```

编译时，每个模块将成为一个单独的 `.js` 文件。与引用标签一样，编译器将遵循 `import` 语句来编译依赖文件。

**Validation.ts**

```ts
export interface StringValidator {
  isAcceptable(s: string): boolean;
}
```

**LettersOnlyValidator.ts**

```ts
import { StringValidator } from "./Validation";
const lettersRegexp = /^[A-Za-z]+$/;
export class LettersOnlyValidator implements StringValidator {
  isAcceptable(s: string) {
    return lettersRegexp.test(s);
  }
}
```

**ZipCodeValidator.ts**

```ts
import { StringValidator } from "./Validation";
const numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
```

**Test.ts**

```ts
import { StringValidator } from "./Validation";
import { ZipCodeValidator } from "./ZipCodeValidator";
import { LettersOnlyValidator } from "./LettersOnlyValidator";

let strings = ["Hello", "98052", "101"];

let validators: { [s: string]: StringValidator } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

strings.forEach((s) => {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? "matches" : "does not match"
      } ${name}`
    );
  }
});
```

## 可选的模块加载和其他高级加载场景

在某些情况下，你可能只想在某些条件下加载模块。在 TypeScript 中，下面的模式可以实现这个情况和其他高级加载场景，直接调用模块加载器而不会失去类型安全性。

编译器会检测每个模块是否在发出的 JavaScript 中被使用。如果模块标识符只用作类型注释的一部分，而从不用作表达式，则不会为该模块发出 `require` 调用。省去未使用的引用是一种很好的性能优化，并且还允许可选地加载这些模块。

该模式的核心思想是 `import id = require("…")` 语句允许我们访问模块公开的类型。模块加载器是动态调用的(通过 `require`)，如下面的 `if` 块所示。这利用了引用省略优化，以便只在需要时加载模块。为了使这种模式工作，重要的是通过 `import` 定义的符号仅用于类型位置(即永远不要在将被发送到 JavaScript 中的位置)。

为了维护类型安全，我们可以使用 `typeof` 关键字。`typeof` 关键字在类型位置使用时会生成值的类型，在本例中为模块的类型。

**Dynamic Module Loading in Node.js**

```ts
declare function require(moduleName: string): any;
import { ZipCodeValidator as Zip } from "./ZipCodeValidator";
if (needZipValidation) {
  let ZipCodeValidator: typeof Zip = require("./ZipCodeValidator");
  let validator = new ZipCodeValidator();
  if (validator.isAcceptable("...")) {
    /* ... */
  }
}
```

**Sample: Dynamic Module Loading in require.js**

```ts
declare function require(
  moduleNames: string[],
  onLoad: (...args: any[]) => void
): void;
import * as Zip from "./ZipCodeValidator";
if (needZipValidation) {
  require(["./ZipCodeValidator"], (ZipCodeValidator: typeof Zip) => {
    let validator = new ZipCodeValidator.ZipCodeValidator();
    if (validator.isAcceptable("...")) {
      /* ... */
    }
  });
}
```

**Sample: Dynamic Module Loading in System.js**

```ts
declare const System: any;
import { ZipCodeValidator as Zip } from "./ZipCodeValidator";
if (needZipValidation) {
  System.import("./ZipCodeValidator").then((ZipCodeValidator: typeof Zip) => {
    var x = new ZipCodeValidator();
    if (x.isAcceptable("...")) {
      /* ... */
    }
  });
}
```

## 与其他 JavaScript 库一起工作

不是用 TypeScript 编写的库，描述该库的形状需要声明库公开的 API。

我们将不定义实现的声明称为 "环境"。通常在 `.d.ts` 文件中定义。如果你熟悉 C/C++，可以将这些视为 `.h` 文件。让我们看几个例子。

### 环境模块

在 Node.js 中，大多数任务都是通过加载一个或多个模块来完成的。我们可以在自己的 `.d.ts` 文件中，使用顶级导出声明定义每个模块，但将它们编写为一个大的 `.d.ts` 文件会更方便。为此，我们使用类似于环境命名空间的构造，但我们使用 `module` 关键字和模块的引用名称，这些名称将可用于以后的导入。例如：

**node.d.ts (简单的摘录)**

```ts
declare module "url" {
  export interface Url {
    protocol?: string;
    hostname?: string;
    pathname?: string;
  }
  export function parse(
    urlStr: string,
    parseQueryString?,
    slashesDenoteHost?
  ): Url;
}
declare module "path" {
  export function normalize(p: string): string;
  export function join(...paths: any[]): string;
  export var sep: string;
}
```

现在我们可以 `/// <reference> node.d.ts` 然后使用 `import url = require("url")` 或 `import * as URL from "url"` 加载模块；

```ts
/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("https://www.typescriptlang.org");
```

### 简写环境模块

如果你不想花时间在使用新模块之前写出声明，你可以使用简写模块。

**declarations.d.ts**

```ts
declare module "hot-new-module";
```

从简写模块导入的所有内容都是 `any` 类型。

```ts
import x, { y } from "hot-new-module";
x(y);
```

### 通配符模块声明

某些模块加载器（例如 [SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/module-types.md) 和 [AMD](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md)）允许导入非 JavaScript 内容。这些通常使用前缀或后缀来指示特殊的加载语义。通配符模块声明可用于涵盖这些情况。

```ts
declare module "*!text" {
  const content: string;
  export default content;
}

declare module "json!*" {
  const value: any;
  export default value;
}
```

现在您可以导入匹配 `"*!Text"` 或 `"json!*"`。

```ts
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```

### UMD 模块

一些库被设计用于许多模块加载器，或者没有模块加载（全局变量）。这些被称为 [UMD](https://github.com/umdjs/umd) 模块。可以通过导入或全局变量访问这些库。例如

**math-lib.d.ts**

```ts
export function isPrime(x: number): boolean;
export as namespace mathLib;
```

然后可以将该库用作模块内的导入:

```ts
import { isPrime } from "math-lib";
isPrime(2);
mathLib.isPrime(2);
ERROR: // 不能从模块内部使用全局定义
```

它也可以用作全局变量，但只能在脚本中使用。(脚本文件是没有导入或导出的。)

```ts
mathLib.isPrime(2);
```

## 构建模块指南

### 导出尽可能接近顶层

模块的使用者在使用导出的东西时应该尽可能地减少摩擦。添加太多层次的嵌套往往会很麻烦，因此请仔细考虑希望如何构建内容。

从你的模块导出命名空间就是添加过多的嵌套层。虽然命名空间有时有其用途，但它们在使用模块时增加了额外的间接级别。这很快就会成为用户的痛点，而且通常是没必要的。

导出类上的静态方法也有类似的问题 —— 类本身添加了一层嵌套。除非它以明显有用的方式增加表现力或目的，否则请考虑简单地导出辅助函数。

#### 如果你只导出了一个 `class` 或 `function`，请使用 `export default`

正如 "在顶层附近导出" 减少模块使用者的摩擦一样，引入默认导出也是如此。如果模块的主要目的是容纳一个特定的导出，那么你应该考虑将其导出为默认导出。这使得导入和实际使用导入变得更容易一些。例如：

正如“在顶层附近导出”减少模块消费者的摩擦一样，引入默认导出也是如此。如果模块的主要目的是容纳一个特定的导出，那么您应该考虑将其导出为默认导出。这使得导入和实际使用导入变得更容易一些。例如：

**MyClass.ts**

```ts
export default class SomeType {
  constructor() { ... }
}
```

**MyFunction.ts**

```ts
export default function getThing() {
  return "thing";
}
```

**Consumer.ts**

```ts
import t from "./MyClass";
import f from "./MyFunc";
let x = new t();
console.log(f());
```

这对使用者来说是最合适的。他们可以随心所欲地命名你的类型（在例子中为 `t`），并且可以轻易的找到你的对象。

#### 如果要导出多个对象，请将它们全部放在顶层

**MyThings.ts**

```ts
export class SomeType {
  /* ... */
}
export function someFunc() {
  /* ... */
}
```

#### 显式列出导入的名称

**Consumer.ts**

```ts
import { SomeType, someFunc } from "./MyThings";
let x = new SomeType();
let y = someFunc();
```

如果要导入大量内容，请使用命名空间导入模式

**MyLargeModule.ts**

```ts
export class Dog { ... }
export class Cat { ... }
export class Tree { ... }
export class Flower { ... }
```

**Consumer.ts**

```ts
import * as myLargeModule from "./MyLargeModule.ts";
let x = new myLargeModule.Dog();
```

## 重新导出扩展

通常需要在模块上扩展功能。常见的 JS 模式是用扩展原始对象，类似于 JQuery 扩展的工作方式。正如我们前面提到的，模块不像全局命名空间对象那样合并。推荐的解决方案是不要改变原始对象，而是导出一个提供新功能的新实体。

考虑在模块 `Calculator.ts` 中定义一个简单的计算器实现。该模块还导出一个辅助函数，通过传递输入字符串列表并在末尾写入结果来测试计算器功能。

**Calculator.ts**

```ts
export class Calculator {
  private current = 0;
  private memory = 0;
  private operator: string;
  protected processDigit(digit: string, currentValue: number) {
    if (digit >= "0" && digit <= "9") {
      return currentValue * 10 + (digit.charCodeAt(0) - "0".charCodeAt(0));
    }
  }
  protected processOperator(operator: string) {
    if (["+", "-", "*", "/"].indexOf(operator) >= 0) {
      return operator;
    }
  }
  protected evaluateOperator(
    operator: string,
    left: number,
    right: number
  ): number {
    switch (this.operator) {
      case "+":
        return left + right;
      case "-":
        return left - right;
      case "*":
        return left * right;
      case "/":
        return left / right;
    }
  }
  private evaluate() {
    if (this.operator) {
      this.memory = this.evaluateOperator(
        this.operator,
        this.memory,
        this.current
      );
    } else {
      this.memory = this.current;
    }
    this.current = 0;
  }
  public handleChar(char: string) {
    if (char === "=") {
      this.evaluate();
      return;
    } else {
      let value = this.processDigit(char, this.current);
      if (value !== undefined) {
        this.current = value;
        return;
      } else {
        let value = this.processOperator(char);
        if (value !== undefined) {
          this.evaluate();
          this.operator = value;
          return;
        }
      }
    }
    throw new Error(`Unsupported input: '${char}'`);
  }
  public getResult() {
    return this.memory;
  }
}
export function test(c: Calculator, input: string) {
  for (let i = 0; i < input.length; i++) {
    c.handleChar(input[i]);
  }
  console.log(`result of '${input}' is '${c.getResult()}'`);
}
```

下面是使用暴露的 `test` 函数对计算器进行的简单测试。

**TestCalculator.ts**

```ts
import { Calculator, test } from "./Calculator";
let c = new Calculator();
test(c, "1+2*33/11="); // prints 9
```

现在要扩展它，添加对输入非 10 为底数的数字支持，让我们创建 `ProgrammerCalculator.ts`

**ProgrammerCalculator.ts**

```ts
import { Calculator } from "./Calculator";
class ProgrammerCalculator extends Calculator {
  static digits = [
    "0",
    "1",
    "2",
    "3",
    "4",
    "5",
    "6",
    "7",
    "8",
    "9",
    "A",
    "B",
    "C",
    "D",
    "E",
    "F",
  ];
  constructor(public base: number) {
    super();
    const maxBase = ProgrammerCalculator.digits.length;
    if (base <= 0 || base > maxBase) {
      throw new Error(`base has to be within 0 to ${maxBase} inclusive.`);
    }
  }
  protected processDigit(digit: string, currentValue: number) {
    if (ProgrammerCalculator.digits.indexOf(digit) >= 0) {
      return (
        currentValue * this.base + ProgrammerCalculator.digits.indexOf(digit)
      );
    }
  }
}
// Export the new extended calculator as Calculator
export { ProgrammerCalculator as Calculator };
// Also, export the helper function
export { test } from "./Calculator";
```

新模块 `ProgrammerCalculator` 导出类似于原始 `Calculator` 模块的 API 形状，但不会增加原始模块中的任何对象。以下对我们的 ProgrammerCalculator 类的测试：

**TestProgrammerCalculator.ts**

```ts
import { Calculator, test } from "./ProgrammerCalculator";
let c = new Calculator(2);
test(c, "001+010="); // prints 3
```

## 不要在模块中使用命名空间

当第一次迁移到基于模块的组织时，常见的趋势是将导出包装在名称空间的附加层中。模块有自己的作用域，只有导出的声明才能在模块外部可见。考虑到这一点，在使用模块时，命名空间提供的价值很小或者没有价值。

在组织方面，命名空间可以方便地将全局作用域中逻辑相关的对象和类型分组在一起。例如，在 c# 中，你可以在 System.Collections 中找到所有的集合类型。通过将我们的类型组织成层次化的命名空间，我们为这些类型的用户提供了良好的“发现”体验。另一方面，模块必须已经存在于文件系统中。我们必须通过路径和文件名解析它们，因此有一个逻辑组织方案可供我们使用。我们可以有一个 /collections/generic/ 文件夹，里面有一个list模块。

命名空间对于避免全局作用域中的命名冲突非常重要。例如，你可能有`My.Application.Customer.AddForm` 和`My.Application.Order.AddForm`这两个类型，具有相同的名称，但不同命名空间。然而，在模块中，这不是问题。在一个模块中，没有理由让两个对象具有相同的名称。从使用者角度来看，任何给定模块，可以根据自己的需要为模块取一个独特的名称，因此不可能发生意外的命名冲突。

> 有关模块和名称空间的更多讨论，请参见 [名称空间和模块](https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html).

## 警告

以下所有内容都是模块结构的危险信号。如果你的文件符合以下情况，请检查你是否尝试对外部模块进行了命名空间处理：

* 一个文件的唯一顶层声明是 `export namespace Foo { ... }`(删除 `Foo` 并将所有内容向上移动一级)

* 多个文件在顶层具有 `export namespace Foo {` (不要认为这些文件会合并成一个 `Foo`!)

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/modules.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>