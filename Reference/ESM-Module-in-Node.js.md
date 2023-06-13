# ECMAScript Modules in Node.js

在过去的几年里，Node.js 一直致力于支持运行 ECMAScript 模块(ESM)。这是一个非常难以支持的特性，因为 Node.js 生态系统的基础是建立在一个叫做 CommonJS (CJS) 的不同模块系统上的。

两个模块系统之间的互操作带来了巨大的挑战，需要处理许多新特性；然而，现在 Node.js 中对 ESM 的支持已经实现了。

这就是为什么 TypeScript 在 `module` 和 `moduleResolution` 中引入了两个新的设置：`Node16` 和 `NodeNext`。

```json
{
    "compilerOptions": {
        "module": "NodeNext",
        "moduleResolution": "NodeNext"
    }
}
```

我们将探讨这些新模式带来的一些高级特性。

## `package.json` 中的 `type` 和新扩展

Node.js [在 package.json 中支持了一个叫做 `"type"` 的新设置](https://nodejs.org/api/packages.html#packages_package_json_and_file_extensions)。`"type"` 可以设置为 `"module"` 或 `"commonjs"`。

```json
{
    "name": "my-package",
    "type": "module",
    
    "//": "...",
    "dependencies": {
    }
}
```


这个设置控制 `.js` 和 `.d.ts` 文件被解释为 ES 模块 或 CommonJS 模块，未设置时默认为 CommonJS。当一个文件被认为是一个 ES 模块时，与 CommonJS 相比，一些不同的规则开始发挥作用：

- 可以使用 `import/export` 语句和顶层 `await`
- 相对导入路径需要完整的扩展名（例如，`import "./foo.js"` 而不能是 `import "./foo"`）
- 导入的解析可能与 `node_modules` 中的依赖项不同
- 某些类似全局的值(特定于模块的全局变量)，如 `require()` 和 `__dirname` 不能直接使用
- 导入 CommonJS 模块会遵循某些特殊规则

当 `.ts` 文件被编译为 ES 模块时，ECMAScript `import`/`export` 语法在 `.js` 输出文件中被单独保留；当它被编译为 CommonJS 模块时，将产生与 [module](https://www.typescriptlang.org/tsconfig#module)：`CommonJS` 相同的输出。

这也意味着 `.ts` 文件作为 ES 模块和作为 CJS 模块时，路径解析是不同的。例如：

```ts
// ./foo.ts
export function helper() {
  // ...
}

// ./bar.ts
import { helper } from "./foo"; // 只适用于 CJS

helper();
```

这段代码可以在 CommonJS 模块中工作，但在 ES 模块中会失败，因为 ES 模块中相对导入路径需要使用扩展名。因此，必须重写它，使用 `foo.ts` 的输出文件扩展名 —— 所以使用 `./foo.js` 导入。

```ts
// ./bar.ts
import { helper } from "./foo.js"; // 适用于 ESM & CJS

helper();
```

一开始你可能会觉得有点麻烦，但 TypeScript 工具(如，自动导入和路径完成)通常会帮你完成这些。

另一方面，这也适用于 `d.ts` 文件。当 TypeScript 找到一个 `.d.ts` 它是作为 ESM 文件还是 CommonJS 文件处理，取决于所包含的包。

## 新文件扩展名

`package.json` 中的 `type` 字段很棒，因为它允许我们继续使用 `.ts` 和 `.js` 文件扩展名，这很方便；但是，你可能需要编写的文件类型不同于 `type` 字段指定的模块，或希望文件类型始终是明确的。

Node.js 支持两个扩展名来帮助解决这个问题：`.mjs` 和 `.cjs`。 `.mjs` 文件始终是 ES 模块，而 `.cjs` 文件始终是 CommonJS 模块，并且没有办法覆盖它们。

而 TypeScript 支持两个新的源文件扩展名：`.mt`s 和 `.cts`。当 TypeScript 将它们编译为 JavaScript 文件时，它会分别将它们编译为 `.mjs` 和 `.cjs`。

此外，TypeScript 还支持两个新的声明文件扩展名：`.d.mts` 和 `.d.cts`。当 TypeScript 生成 `.mts` 和 `.cts` 的声明文件时，它们对应的扩展名将是 `.d.mts` 和 `.d.cts`。

使用这些扩展名完全是可选的，即使你不选择它们用作主要工作流程的一部分，它们通常也是有用的。

## CommonJS 交互操作

Node.js 允许 ES 模块导入 CommonJS 模块，就像它们是带有默认导出的 ES 模块一样。

```ts
// @filename: helper.cts
export function helper() {
  console.log("hello world!");
}

// @filename: index.mts
import foo from "./helper.cjs";

// prints "hello world!"
foo.helper();
```

在某些情况下，Node.js 还会从 CommonJS 模块中合成命名的导出，这样会更方便。在这些情况下，ES 模块可以使用"命名空间风格"的导入(例如，`import * as foo from "…"`)或命名导入(例如，`import { helper } from "…"`)。

```ts
// @filename: helper.cts
export function helper() {
  console.log("hello world!");
}

// @filename: index.mts
import { helper } from "./helper.cjs";

// prints "hello world!"
helper();
```

在某些情况下，TypeScript 无法确定这些命名导入是否会被合成。但是，TypeScript 倾向于保守处理，并从明确是 CommonJS 模块的文件中，进行导入时使用一些试探性规则。

关于互操作性，有一个 TypeScript 特定的语法需要注意：

```ts
import foo = require("foo");
```

在 CommonJS 模块中，这仅仅是一个 `require()` 调用，而在 ES 模块中，这些导入会使用 [createRequire](https://nodejs.org/api/module.html#module_module_createrequire_filename) 来实现相同的功能。这样做会使代码在不支持 `require()` 的运行时（比如浏览器）上的可移植性降低，但在实现互操作性方面通常是很有用的。因此，你可以使用以下语法来编写上述示例：

```ts
// @filename: foo.cts
export function helper() {
  console.log("hello world!");
}

// @filename: index.mts
import foo = require("./foo.cjs");

foo.helper()
```

值得注意的是，从 CommonJS（CJS）模块中导入 ESM（ES 模块）文件的唯一方式是使用动态的 `import()` 调用。这可能会带来一些挑战，但这是当前在 Node.js 中的行为。

你可以在这里阅读更多关于 [ESM/CommonJS 在Node.js中的互操作](https://nodejs.org/api/esm.html#esm_interoperability_with_commonjs)。

## `package.json` 导出，导入，和自引用

Node.js 在 `package.json` 中支持一个新的字段来定义入口点，称为 ["exports"](https://nodejs.org/api/packages.html#packages_exports)。这个字段是 `"main"` 字段更强大的替代方式，可以控制将包的哪些部分暴露给使用者。

下面是一个 `package.json` 示例，支持 CommonJS 和 ESM 的分开入口点：

```json
// package.json
{
  "name": "my-package",
  "type": "module",
  "exports": {
      ".": {
          // 在 ESM 中，import "my-package" 的入口点
          "import": "./esm/index.js",
          // 在 CJS 中， require("my-package") 的入口点
          "require": "./commonjs/index.cjs",
      },
  },
  // 兼容旧版 Node.js 的 CJS 使用
  "main": "./commonjs/index.cjs",
}
```

这个功能有很多内容，你可以在 [Node.js 文档](https://nodejs.org/api/packages.html) 中阅读更多详细信息。这里我们将重点介绍 TypeScript 如何支持它。

在 TypeScript 的原始 Node 支持中，它会查找 `"main"` 字段，然后查找与该入口对应的声明文件。例如，如果 `"main"` 指向 `./lib/index.js`，TypeScript 会查找名为 `./lib/index.d.ts` 的文件。包的作者可以通过指定一个名为 `"types"` 的单独字段来覆盖这一行为（例如，`"types": "./types/index.d.ts"`）。

新的支持与 [导入条件](https://nodejs.org/api/packages.html) 工作方式类似。默认情况下，TypeScript 使用 导入条件 覆盖相同的规则 —— 如果你从 ES 模块导入，它将查找 `import` 字段；如果从 CommonJS 模块导入，它将查找 `require` 字段。如果找到它们，它将查找一个相同位置的声明文件。如果你需要指定不同的位置来存放类型声明文件，可以添加一个 `"types"` 的 导入条件。

```json
// package.json
{
  "name": "my-package",
  "type": "module",
  "exports": {
      ".": {
          // 在 ESM 中，import "my-package" 的入口点
          "import": {
              // TypeScript 将要找的路径
              "types": "./types/esm/index.d.ts",
              // Node.js 将要找的路径
              "default": "./esm/index.js"
          },
          // 在 CJS 中， require("my-package") 的入口点
          "require": {
              // TypeScript 将要找的路径
              "types": "./types/commonjs/index.d.cts",
              // Node.js 将要找的路径
              "default": "./commonjs/index.cjs"
          },
      }
  },
  // 兼容旧版本 TypeScript 的回退
  "types": "./types/index.d.ts",
  // 兼容旧版 Node.js 的 CJS 回退
  "main": "./commonjs/index.cjs"
}
```

> "types" 条件应该总是出现在 "default" 的前面

需要注意的是，CommonJS 入口点和 ES 模块入口点各自需要自己的声明文件，即使它们之间的内容相同。每个声明文件根据其文件扩展名和 `package.json` 中的 `"type"`字段，被解释为 CommonJS 模块或 ES 模块，而这个检测到的模块类型必须与 Node 对应的 JavaScript 文件的模块类型匹配，以确保类型检查正确。如果试图使用单个 `.d.ts` 文件来为 ES 模块入口点和 CommonJS 入口点两者提供类型，TypeScript 会认为只存在其中一个入口点，从而导致使用该包的用户出现编译错误。

TypeScript 还以类似的方式支持 `package.json `的 ["imports"](https://nodejs.org/api/packages.html#packages_imports) 字段（查找相应文件旁边的声明文件），并支持 [包自引用](https://nodejs.org/api/packages.html#packages_self_referencing_a_package_using_its_name)。这些特性通常不太使用，但也支持。


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/esm-node.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

