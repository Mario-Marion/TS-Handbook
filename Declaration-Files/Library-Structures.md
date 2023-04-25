# Library Structures

一般来说，构造声明文件的方式依赖于库是如何使用的。有许多提供给 JavaScript 使用的库方法，你需要编写与之匹配的声明文件。

本章节主要帮助理解常用的库结构，以及如何编写每种结构的适当声明文件。如果你正在编辑现有文件，那么大概不需要阅读本章节。强烈建议新声明文件的作者阅读本章节，理解库结构是如何影响声明文件编写的。

如果你已经知道你的库结构是什么，请参阅 [后面章节](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html) 。在后面章节中，将有许多声明文件，它们可以作为编写新声明文件的起点。
每种类型的主要库结构模式在后面章节都有相应的文件。你可以从这些模板开始，以帮助你快速声明。

## 识别库的种类

首先，我们将回顾 TypeScript 声明文件可以表示的库类型。我们将简单的展示每种库类型的使用方式和编写方式，并列举一些常见的库示例。

识别库的结构是编写库声明文件的第一步。我们将提示如何根据用法和代码来识别结构。根据库的文档和组织，其中一个可能比另一个更容易。我们建议你选择更舒适的。

## 你应该寻找什么

1. 如何获取库？

    是通过 npm 获取的，还是 CDN 获取的？

2. 如何导入库？

    如何添加全局对象？是使用 `require` 还是 `impot/export` 语法？

## 针对不同类型库的较小示例

### 模块化库

几乎所有现代 Node.js 库都属于模块家族。这些类型库只能在带有模块加载器的 JS 环境使用。例如， `express` 只能在 Node.js 使用，并且必须使用 CommonJS 的 `require` 函数加载。

ECMAScript 2015（也就是 ECMAScript 6、ES6），CommonJS，和 RequireJS 有类似的导入模块概念。在 CommonJS（Node.js）中，你可以这么写：

```js
var fs = require("fs");
```

在 TypeScript 或 ES6，`import` 关键字有相同作用：

```js
import * as fs from "fs";
```

你通常会在模块化库的文档中看到以下代码：

```js
var someLib = require("someLib");
```

或

```js
define(..., ['someLib'], function(someLib) {

});
```

与全局模块一样，你可能会在 [UMD](https://juejin.cn/post/7225886242259927097#heading-11) 模块的文档中看到这些示例，所以请务必检查代码或文档。

#### 从代码识别模块化库

模块化库至少具有以下一些用法：

- 无条件调用 `require` 或 `define`
- 声明 `import * as a from 'b';` 或 `export c;`
- 给 `exports` 或 `module.exports` 赋值

它们很少有：

- 赋值给 `window` 或 `global` 的属性

#### 模块模板

模块有四个可用的模板：[module.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html), [module-class.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-class-d-ts.html), [module-function.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-function-d-ts.html) and [module-plugin.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-plugin-d-ts.html).

你应该第一个阅读 [module.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html) 了解它们的工作方式。

如果你的模块可以像函数一样被调用，可以使用 [module-function.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-function-d-ts.html) 模板

```js
const x = require("foo");
// Note: calling 'x' as a function
const y = x(42);
```

如果你的模块可以使用 `new` 被构造，可以使用 [module-class.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-class-d-ts.html) 模板

```js
const x = require("bar");
// Note: using 'new' operator on the imported variable
const y = new x("hello");
```

如果你的模块在导入时对其他模块做了修改，可以使用 [module-plugin.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-plugin-d-ts.html) 模板

```js
const jest = require("jest");
require("jest-matchers-files");
```

### 全局库

全局库是可以从全局作用域访问的库（不使用任何形式的导入）。许多库提供了一个或多个全局变量。例如，如果你正在使用 [jQuery](https://jquery.com/), `$` 变量可以直接使用：

```jequery
$(() => {
    console.log("hello!");
});
```

你通常会在全局库的文档中看到，如何在 HTML 脚本标签中使用库的指导：

```js
<script src="http://a.great.cdn.for/someLib.js"></script>
```

现在，大多数流行的全局可访问库，实际上都被编写为 UMD 库（见下文）。UMD 库文档很难与全局库文档区分开来。在编写全局声明文件之前，请确保库实际上不是 UMD。

#### 从代码识别全局库

全局库代码通常非常简单。一个全局的 "Hello, world" 库可能是这样的：

```js
function createGreeting(s) {
    return "Hello, " + s;
}
```

或

```js
// Web
window.createGreeting = function (s) {
  return "Hello, " + s;
};
// Node
global.createGreeting = function (s) {
  return "Hello, " + s;
};
// Potentially any runtime
globalThis.createGreeting = function (s) {
  return "Hello, " + s;
};
```

在全局库代码中，你经常会看到：

- 顶级 `var` 语句或 `function` 声明
- 一个或多个 `window.someName` 赋值
- 假设存在原始 DOM 对象 `docuemnt` 或 `window`

你不会看到：

- 检查像 `require` 或 `define` 这样的模块加载器用法
- CommonJS/Node.js 风格的导入 `var fs = require("fs");`
- 调用 `define(...)`
- 文档描述如何 `require` 或导入库

#### 全局库例子

因为将全局库转换为 UMD 库通常很容易，所以很少有流行的库仍然以全局风格编写。然而，较小且需要 DOM (或没有依赖项）的库可能仍然是全局的。

#### 全局库模板

模板文件 [global.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-plugin-d-ts.html) 定义了一个示例库 `myLib`。一定要阅读 ["防止名称冲突" 的脚注](https://www.typescriptlang.org/docs/handbook/declaration-files/library-structures.html#preventing-name-conflicts)。

### ***UMD***

UMD 模块既可以用作模块（通过导入）库，也可以用作全局（在没有模块加载器的环境中运行时）。许多流行库（如 [Moment.js](https://momentjs.com/)）都是以这种方式编写的。例如，在 Node.js 中或者使用 RequireJS ，你可以这样写：

```js
import moment = require("moment");
console.log(moment.format());
```

而在一个普通的浏览器环境中，你可以这样写:

```js
console.log(moment.format());
```

#### 识别 UMD 库

[UMD 模块](https://github.com/umdjs/umd) 检查是否存在模块加载器环境。这是一个很容易发现的模式，看起来像这样：

```js
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(["libName"], factory);
    }
    else if (typeof module === "object" && module.exports) {
        module.exports = factory(require("libName"));
    }
    else {
        root.returnExports = factory(root.libName);
    }
}(this, function (b) {}));
```

如果你在库的代码中看到 `typeof define`、`typeof window`或`typeof module` 的测试，特别是在文件的顶部，那么它大概率是一个 UMD 库。

UMD 库的文档也经常会展示一个 "在 Node.js 中使用" 的例子，使用 `require`，以及一个 "在浏览器中使用" 的例子，使用`<script>` 标签来加载脚本。

#### UMD 库例子

大多数流行的库现在都以 UMD 包的形式提供。包括 [jQuery](https://jquery.com/), [Moment.js](https://momentjs.com/), [lodash](https://lodash.com/), 等等。

#### 模板

使用 [module-plugin.d.ts](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-plugin-d-ts.html) 模板

## 依赖关系

你的库可能有几种依赖关系。这部分将展示如何将它们导入到声明文件中。

### 依赖于全局库

如果你的库依赖于一个全局库，可使用 `/// <reference types="..." />` 指令：

```ts
/// <reference types="someLib" />
function getThing(): someLib.thing;
```

### 依赖于模块

如果你的库依赖于模块，可使用 `import` 语句：

```ts
import * as moment from "moment";
function getThing(): moment;
```

### 依赖于 UMD 库
#### 来自全局库

如果你的全局库依赖于 UMD 模块，可使用 `/// <reference types` 指令：

```ts
/// <reference types="moment" />
function getThing(): moment;
```

#### 来自模块或 UMD 库

如果你的模块或 UMD 库依赖于 UMD 库，可使用 `import` 语句：

```ts
import * as someLib from "someLib";
```

不要使用 `/// <reference` 指令来声明对 UMD 库的依赖！

## 注脚
### 防止命名冲突

注意，当编写全局声明文件时，它可能在全局作用域定义许多类型。我们强烈反对这样做，因为当一个项目中有许多声明文件时，它可能导致无法解决的命名冲突。

要遵循的一个简单规则，库定义的任何全局变量声明为命名空间类型。例如，如果库定义了全局值 "cats"，你应该这样写：

```ts
declare namespace cats {
  interface KittySettings {}
}
```

不能是：

```ts
// at top-level
interface CatsKittySettings {}
```

该指南还可以确保库在不破坏声明文件用户的情况下转换为 UMD。

### ES6 对模块调用签名的影响

许多流行库（如 Express），在导入时将自身暴露为可调用的函数。例如，典型的 Express 用法是这样的：

```js
import exp = require("express");
var app = exp();
```

在 ES6 兼容的模块加载器中，顶级对象（这里导入为 `exp`）只能有属性；顶层模块对象永远不能被调用。

这里最常见的解决方案是定义为 可调用/可构造 对象的默认导出；模块加载器通常会自动检测这种情况，并使用 `default` 导出替换顶级对象。在 tsconfig.json 设置 ["esModuleInterop": true](https://www.typescriptlang.org/tsconfig/#esModuleInterop) ，TypeScript 可以帮你处理这个问题



感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-files/library-structures.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>


