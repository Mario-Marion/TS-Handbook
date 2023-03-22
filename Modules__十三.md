---
highlight: vs2015
---

# Modules
JavaScript 在处理模块化代码方面有着悠久的历史。TypeScript 从 2012 年首次发布就已经实现了对很多这些格式的支持，但是随着时间的推移，社区和 JavaScript 规范已经趋同于一种叫做 ES模块 的格式（ES6 模块）。你可能知道它的语法 `import` / `export`。

ES模块 在 2015 年被添加到 JavaScript 规范中，到 2020 年 ES模块 在大多数 web 浏览器和 JavaScript 运行时得到了广泛的支持。

本手册将涵盖 ES模块 和它流行之前的 CommonJS（`module.exports =` 语法），并且你可以在官网 Reference 下的 [Modules](https://www.typescriptlang.org/docs/handbook/modules.html) 找到关于其他模块模式的信息。
## JavaScript 模块是如何定义的
在 TypeScript 中，和 ECMAScript 2015 定义一样，任何包含顶级 `import` 或 `export` 声明的文件都被视为模块。

如果没有任何顶级 `import` 或 `export` 声明的文件都被视为脚本，其内容在全局作用域中可用的（也可用于模块）。

模块在它们自己的作用域中执行，而不是在全局作用域中执行。这意味着在模块中声明的变量、函数、类等，在模块外部是不可见的。除非使用模块的导出形式明确的导出它们。并且，要使用不同模块导出的变量、函数、类、接口等，必须在使用的地方使用模块的导入形式进行导入。
## 非模块
在开始之前，了解 TypeScript 对模块的定义是很重要的。JavaScript 规范声明，JavaScript 文件没有 `export` 或顶级 `await` 声明的都应该被视为脚本，而不是模块。

在脚本文件中，变量和类型被声明在共享的全局作用域。假设使用 [outFile](https://www.typescriptlang.org/tsconfig#outFile) 编译器选项，将多个输入文件连接到一个输出文件中，或者在 HTML 中使用多个 `<script>` 标签来加载这些文件（以正确的顺序!）。

如果你有一个文件，没有任何 `import` 的东西或 `export` 的东西，但你想将它视为一个模块，可添加这行：
```ts
export {};
```
它将把文件更改为一个不导出任何内容的模块。无论你的模块目的是什么，此语法都有效。
## TypeScript 中的模块
在 TypeScript 中编写模块代码时，主要考虑三件事情：
- **语法**：要使用什么语法来导入和导出内容？
- **模块解析**：模块名称（或路径）和磁盘上的文件之间是什么关系?
- **模块输出目标**：输出的 JavaScript 模块应该是什么样子?

### ES 模块语法
一个文件可以通过 `export default` 声明一个主要导出：
```ts
// @filename: hello.ts
export default function helloWorld() {
    console.log("Hello, world!");
}
```
然后通过以下方式导入：
```ts
import helloWorld from "./hello.js";
helloWorld();
```
除了默认导出，你可以省略 `default`，通过 `export` 导出多个变量和函数：
```ts
// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;

export class RandomNumberGenerator {}

export function absolute(num: number) {
if (num < 0) return num * -1;
    return num;
}
```
通过 `import` 语法，在其它文件中导入上面导出的变量和函数，就可以使用它们:
```ts
import { pi, phi, absolute } from "./maths.js";

console.log(pi);
const absPhi = absolute(phi);
// absPhi 类型：const absPhi: number
```
### 其他导入语法
可以使用 `import {old as new}` 格式重命名导入：
```ts
import { pi as π } from "./maths.js";
console.log(π);
// π 类型：
// (alias) var π: number 
// import π
```
你可以在单独的 `import` 声明中，混合和匹配上面的语法：
```ts
// @filename: maths.ts
export const pi = 3.14;
export default class RandomNumberGenerator {}

// @filename: app.ts
import RandomNumberGenerator, { pi as π } from "./maths.js";

RandomNumberGenerator;
// RandomNumberGenerator 类型：
// (alias) class RandomNumberGenerator
// import RandomNumberGenerator

console.log(π);
// π 类型：
// (alias) const π: 3.14
// import π
```
你可以使用 `* as name` 将所有导出的对象放到单个命名空间中：
```ts
// @filename: app.ts
import * as math from "./maths.js";

console.log(math.pi);
const positivePhi = math.absolute(math.phi);
// positivePhi 类型：const positivePhi: number
```
通过 `import "./file"` 语法可以导入一个文件，并且不包含任何变量到你当前模块中：
```ts
// @filename: app.ts
import "./maths.js";

console.log("3.14");
```
在本例中，`import` 没有导入任何东西。然而，会执行 `maths.ts` 中的所有代码，这可能会引发，影响其他对象的副作用。
### TypeScript 特定的 ES 模块语法
类型可以使用与 JavaScript 值相同的导出和导入语法：
```ts
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
export interface Dog {
    breeds: string[];
    yearOfBirth: number;
}

// @filename: app.ts
import { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
```
TypeScript 扩展了 `import` 语法，使用了两个概念来声明类型的导入：
#### 导入类型
这是一个 `import` 语句，只能导入类型：
```ts
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
export type Dog = { breeds: string[]; yearOfBirth: number };
export const createCatName = () => "fluffy";

// @filename: valid.ts
import type { Cat, Dog } from "./animal.js";
export type Animals = Cat | Dog;

// @filename: app.ts
import type { createCatName } from "./animal.js";
const name = createCatName();
// Error：'createCatName' 不能作为值使用，因为它是使用 'import type' 导入的
```
#### 内联类型导入
TypeScript 4.5 还允许单独的导入（`type` 为前缀），以表明导入的引用是一种类型:
```ts
// @filename: app.ts
import { createCatName, type Cat, type Dog } from "./animal.js";

export type Animals = Cat | Dog;
const name = createCatName();
```
这些可以促使非 typescript 编译器（如 Babel、swc 或 esbuild）了解哪些导入可以安全的删除。
### ES 模块语法与 CommonJS 行为
TypeScript 有 ES 模块语法，直接与 CommonJS 和 AMD 的 `require` 相互关联。使用 ES 模块的导入，在大多数情况下与这些环境中的 `request` 相同，但这种语法确保在你的 TypeScript 文件中，与 CommonJS 输出有 1 对 1 的匹配：
```ts
import fs = require("fs");
const code = fs.readFileSync("hello.ts", "utf8");
```
您可以在 [模块参考页面](https://www.typescriptlang.org/docs/handbook/modules.html#export--and-import--require). 了解关于此语法的更多信息。
### CommonJS 语法
CommonJS 是 npm 中大多数模块都使用的格式。即使你使用的是上面的 ES Modules 语法，对 CommonJS 语法的工作原理有一个简单的了解，也会帮助你更容易调试。
#### Exporting
通过设置全局变量 `module` 上的 `exports` 属性来导出。
```ts
function absolute(num: number) {
    if (num < 0) return num * -1;
    return num;
}

module.exports = {
    pi: 3.14,
    squareTwo: 1.41,
    phi: 1.61,
    absolute,
};
```
然后可以通过 `require` 语句导入这些文件：
```ts
const maths = require("maths");
maths.pi;
// maths 类型为any，所以它的所有属性都为 any
```
或者你可以使用 JavaScript 中的解构功能来简化：
```ts
const { squareTwo } = require("maths");

squareTwo;
// squareTwo 类型：any
```
### CommonJS 和 ES 模块互操作
CommonJS 和 ES 模块，在 "默认导入" 和 "模块命名空间对象导入" 的区分上，存在一些不匹配的特性。TypeScript 有一个 [esModuleInterop](https://www.typescriptlang.org/tsconfig#esModuleInterop) 编译器标志来减少两个不同集合之间的分歧。
### TypeScript 的模块解析选项
模块解析是从 `import` 或 `require` 语句中获取字符串，并确定该字符串引用什么文件的过程。

TypeScript 包含两种解析策略: Classic 和 Node。当编译器选项 [module](https://www.typescriptlang.org/tsconfig#module) 不是 `commonjs` 时，Classic 策略是默认值，用于向后兼容。Node 策略复制了 Node.js 在 CommonJS 模式下的工作方式。

以下的 TSConfig 标志会影响 TypeScript 中的模块策略：
[moduleResolution](https://www.typescriptlang.org/tsconfig#moduleResolution)，[baseUrl](https://www.typescriptlang.org/tsconfig#baseUrl)，[paths](https://www.typescriptlang.org/tsconfig#paths)，[rootDirs](https://www.typescriptlang.org/tsconfig#rootDirs)

有关这些策略如何工作的完整细节，可以参考 [Module Resolution](https://www.typescriptlang.org/docs/handbook/module-resolution.html)。
### TypeScript的模块输出选项
有两个选项会影响发出的 JavaScript 输出：
- [target](https://www.typescriptlang.org/tsconfig#target) 它决定了哪些 JS 特性会被降级（转换为在旧的 JavaScript 运行时运行），哪些 JS 特性保持不动。
- [module](https://www.typescriptlang.org/tsconfig#module) 它决定了模块之间使用什么代码进行交互。

使用哪个 [target](https://www.typescriptlang.org/tsconfig#target) 取决于你希望运行在 TypeScript 代码中的 JavaScript 运行时特性。特性可能来自：你支持的最旧的 Web 浏览器，你希望去运行 Node.js 最低版本或运行可能来自你运行时独特的约束——例如 Electron。

模块之间的所有通信都通过模块加载器进行，编译器选项 [module](https://www.typescriptlang.org/tsconfig#module) 决定使用哪一个。在运行时，模块加载器负责在执行模块之前定位并执行模块的所有依赖。

例如，下面是一个使用 ES 模块语法的 TypeScript 文件，展示了 [module](https://www.typescriptlang.org/tsconfig#module) 的几个不同选项：
```ts
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```
**ES2020**
```tS
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```
**CommonJS**
```ts
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.twoPi = void 0;
const constants_js_1 = require("./constants.js");
exports.twoPi = constants_js_1.valueOfPi * 2;
```
**UMD**
```ts
(function (factory) {

    if (typeof module === "object" && typeof module.exports === "object") {

        var v = factory(require, exports);
        if (v !== undefined) module.exports = v;

    } else if (typeof define === "function" && define.amd) {

        define(["require", "exports", "./constants.js"], factory);
    }

})(function (require, exports) {

    "use strict";
    Object.defineProperty(exports, "__esModule", { value: true });
    exports.twoPi = void 0;
    const constants_js_1 = require("./constants.js");
    exports.twoPi = constants_js_1.valueOfPi * 2;

});
```
> 注意，ES2020 是有效的，相当于原始的 **index.ts**

你可以在 TSConfig 参考的 [module](https://www.typescriptlang.org/tsconfig#module) 中看到所有可用选项以及它们编译后的 JavaScript 代码。
### TypeScript 命名空间
TypeScript有自己的模块格式，称为 `namespace`，它早于 ES 模块标准。这个语法对于创建复杂的定义文件，有很多有用的特性，在 [DefinitelyTyped](https://www.typescriptlang.org/dt) 中仍能看到活跃的使用。虽然没有被弃用，但命名空间中的大多数特性都存在于 ES 模块，我们建议你使用它时与  JavaScript 的保持一致。你可以在 [命名空间参考页面](https://www.typescriptlang.org/docs/handbook/namespaces.html) 中了解更多信息。


感谢观看，如有错误，望指正

>官网地址： <https://www.typescriptlang.org/docs/handbook/2/modules.html>
>
>github 资料： <https://github.com/Mario-Marion/TS-Handbook>
