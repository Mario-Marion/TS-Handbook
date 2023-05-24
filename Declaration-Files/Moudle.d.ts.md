# 对比 JavaScript 与 .D.TS

## 常见的 CommonJS 模式

一个使用了 CommonJS 模式的模块，使用 `module.exports` 去描述导出值。

如下模块，导出了一个函数和一个数值常量。

```js
const maxInterval = 12;

function getArrayLength(arr) {
    return arr.length;
}

module.exports = {
    getArrayLength,
    maxInterval,
};
```

可以通过以下 `.d.ts` 文件去描述该模块。

```ts
export function getArrayLength(arr: any[]): number;
export const maxInterval: 12;
```

在 [TypeScript playground](https://www.typescriptlang.org/play?useJavaScript=true#code/GYVwdgxgLglg9mABAcwKZQIICcsEMCeAMqmMlABYAUuOAlIgN6IBQiiW6IWSNWAdABsSZcswC+zCAgDOURAFtcADwAq5GKUQBeRAEYATM2by4AExBC+qJQAc4WKNO2NWKdNjxFhFADSvFquqk4sxAA) 中可看到 `.d.ts` 和该 JavaScript 代码是对应的。

在 `.d.ts` 中声明类型的语法故意看起来像 [ES Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) 语法。ES Modules 是在 2019 年被 TC39 批准的，虽然已经通过编译器提供很长时间了。

如果你有以下使用 ES Modules 的 JavaScript 代码：

```js
export function getArrayLength(arr) {
    return arr.length;
}
```

对应 `.d.ts` 文件为：

```ts
export function getArrayLength(arr: any[]): number;
```

### 默认导出

在 CommonJS 中可以导出一个任意值为默认导出。

如下模块，导出一个正则表达式。

```js
module.exports = /hello( world)?/;
```

可通过以下 `.d.ts` 描述：

```ts
declare const helloWorld: RegExp;
export default helloWorld;
```

或导出一个数值：

```js
module.exports = 3.142;
```

对应 `.d.ts`

```ts
declare const pi: number;
export default pi;
```

CommonJS 的导出风格是导出一个函数。因为函数也是一个对象，所以可添加额外的字段并包含在导出中。

```js
function getArrayLength(arr) {
    return arr.length;
}

getArrayLength.maxInterval = 12;
module.exports = getArrayLength;
```

可通过以下 `.d.ts` 描述：

```ts
export default function getArrayLength(arr: any[]): number;
export const maxInterval: 12;
```

需要注意的是，在 `.d.ts` 文件中使用 `export default` 时，需要在项目 TSConfig 中开启 [`esModuleInterop: true`](https://www.typescriptlang.org/tsconfig#esModuleInterop)。如果没有配置 `esModuleInterop: true`，则必须使用 `export=` 语法代替。这种旧语法更难使用，但在任何地方都适用。

下面是使用 `export=` 编写上述示例的方式：

```ts
declare function getArrayLength(arr: any[]): number;
declare namespace getArrayLength {
    declare const maxInterval: 12;
}
export = getArrayLength;
```

有关其工作原理的详细信息，请参阅 [Module: Functions](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-function-d-ts.html) 和 [Modules reference](https://www.typescriptlang.org/docs/handbook/modules.html) 页面。

## 处理导入

以下是现代导入模块的方法：

```js
const fastify = require("fastify");

const { fastify } = require("fastify");

import fastify = require("fastify");

import * as Fastify from "fastify";

import { fastify, FastifyInstance } from "fastify";

import fastify from "fastify";

import fastify, { FastifyInstance } from "fastify";
```

涵盖所有这些情况需要 JavaScript 代码实际支持所有这些模式。为了支持这些模式，CommonJS 模块需要看起来像这样：

```ts
class FastifyInstance {}

function fastify() {
    return new FastifyInstance();
}

fastify.FastifyInstance = FastifyInstance;

// Allows for { fastify }
fastify.fastify = fastify;

// Allows for strict ES Module support
fastify.default = fastify;

// Sets the default export
module.exports = fastify;
```

## 模块中的类型

你可能希望为不存在的 JavaScript 代码提供类型：

```js
function getArrayMetadata(arr) {
    return {
        length: getArrayLength(arr),
        firstObject: arr[0],
    };
}

module.exports = {
    getArrayMetadata,
};
```

可描述为：

```ts
export type ArrayMetadata = {
    length: number;
    firstObject: any | undefined;
};
export function getArrayMetadata(arr: any[]): ArrayMetadata;
```

可以使用 [泛型](https://www.typescriptlang.org/docs/handbook/generics.html#generic-types) 去提供更丰富的类型信息：

```ts
export type ArrayMetadata<ArrType> = {
    length: number;
    firstObject: ArrType | undefined;
};

export function getArrayMetadata<ArrType>(
    arr: ArrType[]
): ArrayMetadata<ArrType>;
```

现在数组类型传进了 `ArrayMetadata` 元素里面。

被导出的类型可以被模块的使用者通过 TypeScript 代码或 [JSDoc imports](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html#import-types) 中的 `import` 或 `import type` 复用。

### 模块代码中的命名空间

试图描述 JavaScript 代码的运行时关系可能很棘手。当类 ES Modules 语法没有提供足够的工具来描述导出时，可以使用 `namespace`。

如下，你可能有足够复杂的类型，你可以选择在你的 `.d.ts` 去命名它们：

```ts
// 这表示在运行时可用的 JavaScript 类
export class API {
    constructor(baseURL: string);
    getInfo(opts: API.InfoRequest): API.InfoResponse;
}

// 此命名空间与 API 类合并，并允许消费者使用。
// 此文件拥有嵌套在它们自己的部分中的类型
declare namespace API {
    export interface InfoRequest {
        id: string;
    }

    export interface InfoResponse {
        width: number;
        height: number;
    }
}
```

可参考 [.d.ts  deep dive](https://www.typescriptlang.org/docs/handbook/declaration-files/deep-dive.html) 了解命名空间是如何工作的。

### 可选的全局用法

你可以使用 `export as namespace` 去声明你的模块，它将在 UMD 上下文中的全局作用域中可用。

```ts
export as namespace moduleName;
```

## 参考例子

为了了解所有这些部分是如何组合在一起的，这里有一个可参考的 `.d.ts`，可在创建新模块的时候参考。

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是一个模块模板文件，应该重命名为 index.d.ts
*~ 并将其放入与模块同名的文件夹中
*~ 例如, 如果你为 "super-greeter" 编写了一个文件
*~ 这个文件应该在 'super-greeter/index.d.ts'
*/

/*~ 如果这个模块是一个 UMD 模块
*~ 当在模块加载器环境之外加载，暴露了一个全局变量 'myLib'
*~ 在这声明这个全局变量
*~ 否则，删除这个声明
*/
export as namespace myLib;

/*~ 如果这个模块暴露了函数, 则像这样声明它们 */
export function myFunction(a: string): string;
export function myOtherFunction(a: number): number;

/*~ 可通过导入模块来声明可用的类型 */
export interface SomeType {
    name: string;
    length: number;
    extras?: string[];
}

/*~ 可以使用 const、let 或 var 来声明模块的属性 */
export const myField: number;
```

### 库文件布局

声明文件的布局应该反映库的布局。

一个库可以由多个模块组成，例如：

```ts
myLib
  +---- index.js
  +---- foo.js
  +---- bar
         +---- index.js
         +---- baz.js
```

这些可导入为：

```ts
var a = require("myLib");
var b = require("myLib/foo");
var c = require("myLib/bar");
var d = require("myLib/bar/baz");
```

声明文件应该为：

```ts
@types/myLib
  +---- index.d.ts
  +---- foo.d.ts
  +---- bar
         +---- index.d.ts
         +---- baz.d.ts
```

### 测试你的类型

如果你打算将这些更改提交给 DefinitelyTyped 以供每个人使用，那么我们建议你：

> 1. 在 node_modules/@types/[libname] 创建一个新文件夹
> 2. 在文件夹中创建 index.d.ts 文件，并将上面示例复制进去
> 3. 查看模块的使用在哪里中断，并开始填写 index.d.ts
> 4. 当你满意的时候，克隆 [DefinitelyTyped/DefinitelyTyped](https://github.com/DefinitelyTyped) 并按照 README 中的说明操作。

否则

> 1. 在源代码树的根目录下创建一个新文件：[libname].d.ts
> 2. 添加声明模块 "[libname]" { }
> 3. 在声明模块的大括号内添加模板，并查看你的用法在哪里中断

感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
