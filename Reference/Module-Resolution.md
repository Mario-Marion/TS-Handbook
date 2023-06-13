> 本节假设读者对模块有一些基本的了解。有关更多信息，请参阅 [Modules](https://www.typescriptlang.org/docs/handbook/modules.html) 文档。

# Module Resolution

模块解析 是编译器用来确定导入引用什么的过程。假设导入语句 `import { a } from "module"`；为了检查 `a` 的用法，编译器需要明确的知道它代表什么，并且需要检查它在 `moduleA` 的定义 。

此时，编译器会问 “ `moduleA` 的形状是什么？” 虽然听起来很简单，但是 `moduleA` 可能在你自己的 `.ts\.tsx` 文件中定义，或你项目依赖的 `.d.ts` 中定义。

首先，编译器将尝试去找到代表导入模块的文件。为此，编译器遵循两种不同策略之一：[Classic](#heading-3) 或 [Node](#heading-4)。这些策略告诉编译器去哪里寻找 `moduleA`。

如果不起作用，并且模块名称是非相对的（如上面的假设 `"moduleA"` ，它是非相对的），那么编译器将尝试找到 [环境模块声明](https://www.typescriptlang.org/docs/handbook/modules.html#ambient-modules)，接下来我们将讨论非相对导入。

最后，如果编译器没能解析该模块，它将记录一个错误。错误类似于：`error TS2307: Cannot find module 'moduleA'`

## 相对  vs. 非相对 模块导入

模块引用是相对的还是非相对的，会导致模块导入解析方式的不同。

相对导入是以 `/`，`./` 或 `../` 开头的。例如：

*   `import Entry from "./components/Entry";`
*   `import { DefaultHeaders } from "../constants/http";`
*   `import "/mod";`

其它导入为 非相对导入。例如：

*   `import * as $ from "jquery";`
*   `import { Component } from "@angular/core";`

相对导入是相对于导入文件解析的，不能解析环境模块声明。你应该使用相对导入你自己的模块，它能确保在运行时保持它们的相对位置。

非相对导入可以相对于 [baseUrl](https://www.typescriptlang.org/tsconfig#baseUrl) 解析，或通过路径映射解析，将在下面介绍。还可以解析 [环境模块声明](https://www.typescriptlang.org/docs/handbook/modules.html#ambient-modules)。在导入任何外部依赖时请使用非相对路径。

## 模块解析策略

模块解析有两种策略：[Node](#heading-4) 和 [Classic](#heading-3)。 你可以使用 `tsconfig.json` 中的 [moduleResolution](https://www.typescriptlang.org/tsconfig#moduleResolution) 选项去指定模块解析策略。如果没有明确指定，默认为 `--module commonjs` 的 `Node`，否则为 `Classic` 。（包括当 [module](https://www.typescriptlang.org/tsconfig#module) 设置为 `amd`，`system`，`umd`，`es2015`，`esnext`，等的时候）

> 注意：`node` 模块解析式是 TypeScript 社区最常用的，并被推荐用于大多数项目。如果你在 TypeScript 中使用 `import` 和 `export` 有解析问题，尝试设置 `moduleResolution: "node"`, 看看它是否能解决这个问题

### Classic

这曾经是 TypeScript 的默认解析策略。目前，这种策略主要用于向后兼容。

相对导入，将相对于导入文件进行解析。所以在源文件 `/root/src/folder/A.ts` 中，`import { b } from "./moduleB"` 将导致以下查询：

1.  `/root/src/folder/moduleB.ts`
2.  `/root/src/folder/moduleB.d.ts`

然而，对于非相对模块导入，编译器会从包含导入文件的目录开始沿着目录树往上走，试图找到匹配的定义文件。

例如：

在源文件 `/root/src/folder/A.ts` 中，非相对导入 `moduleB` 模块：`import { b } from "moduleB"`，将会尝试以下位置中找寻 `"moduleB"`：

1.  `/root/src/folder/moduleB.ts`
2.  `/root/src/folder/moduleB.d.ts`
3.  `/root/src/moduleB.ts`
4.  `/root/src/moduleB.d.ts`
5.  `/root/moduleB.ts`
6.  `/root/moduleB.d.ts`
7.  `/moduleB.ts`
8.  `/moduleB.d.ts`

### Node

这个解析策略试图模仿 [Node.js](https://nodejs.org/) 运行时的模块解析机制。完整的 Node.js 解析算法在 [Node.js 模块文档](https://nodejs.org/api/modules.html#modules_all_together) 中有概述。

#### Node.js 如何解析模块

为了理解 TS 编译器将遵循哪些步骤，需要了解 Node.js 模块。传统上，Node.js 中的导入是通过调用一个名为 `require` 的函数来执行的。Node.js 的行为会根据 `require` 是相对路径还是非相对路径而有所不同。

相对路径是相当简单的。例如，文件 `/root/src/ moduleA.js`，它包含 `import var x = require("./moduleB");` Node.js 按照以下顺序解析该导入：

1.  询问名为 `/root/src/moduleB.js` 的文件是否存在。
2.  询问文件夹 `/root/src/moduleB` 是否包含一个名为 `package.json` 的文件，并指定 `"main"` 模块。在我们的例子中，如果 Node.js 发现文件 `/root/src/moduleB/package.js` ，并包含 `{"main": "lib/mainModule.js"}`，那么 Node.js 将指向 `/root/src/moduleB/lib/mainModule.js`。
3.  询问文件夹 `/root/src/moduleB` 是否包含一个名为 `index.js` 的文件。该文件被隐式地视为该文件夹的 `"main"` 模块。

你可以在 Node.js 文档中阅读更多关于 [文件模块](https://nodejs.org/api/modules.html#modules_file_modules) 和 [文件夹模块](https://nodejs.org/api/modules.html#modules_folders_as_modules) 的内容。

但是，[非相对模块名](#heading-4) 的解析执行方式不同。Node 将在名为 `node_modules` 的特殊文件夹中查找模块。`node_modules` 文件夹可以与当前文件处于同一级别，或者存在更高的目录链中。Node 将沿着目录链往上走，查找每个 `node_modules`，直到找到你尝试加载的模块。

根据上面的示例，把 `/root/src/moduleA.js` 中的导入改为非相对路径：`var x = require("moduleB");`。那么 Node 将尝试将 `moduleB` 解析到每个位置，直到有一个成功为止。

1.  `/root/src/node_modules/moduleB.js`
2.  `/root/src/node_modules/moduleB/package.json`（如果它指定了一个 `"main"` 属性）
3.  `/root/src/node_modules/moduleB/index.js`
4.  `/root/node_modules/moduleB.js`
5.  `/root/node_modules/moduleB/package.json`（如果它指定了一个 `"main"` 属性）
6.  `/root/node_modules/moduleB/index.js`
7.  `/node_modules/moduleB.js`
8.  `/node_modules/moduleB/package.json`（如果它指定了一个 `"main"` 属性）
9.  `/node_modules/moduleB/index.js`

注意，Node.js 在步骤 (4) 和 (7) 中跳转到上一个目录。

你可以在 Node.js 文档中阅读更多关于 [从 node\_modules 加载模块](https://nodejs.org/api/modules.html#modules_loading_from_node_modules_folders) 的过程。

#### TypeScript 如何解析模块

TypeScript 将模仿 Node.js 运行时解析策略，以便在编译时定位模块的定义文件。为了实现这一点，TypeScript 将 TypeScript 源文件扩展名（`.ts、.tsx` 和 `.d.ts`）覆盖在 Node 的解析逻辑上。 TypeScript 还将使用 `package.json` 中名为 `types` 的字段来反映 `"main"` -- 编译器将使用它来查找 `"main"` 定义文件，以进行查询。

例如，在 `/root/src/moduleA.ts` 中 `import { b } from "./moduleB"`  这样的导入语句将导致在以下位置查询：

1.  `/root/src/moduleB.ts`
2.  `/root/src/moduleB.tsx`
3.  `/root/src/moduleB.d.ts`
4.  `/root/src/moduleB/package.json` (如果它指定了 `types` 属性)
5.  `/root/src/moduleB/index.ts`
6.  `/root/src/moduleB/index.tsx`
7.  `/root/src/moduleB/index.d.ts`

回想一下就会发现，Node.js 查找的是一个名为 `moduleB.js` 的文件，然后应用 `package.json`，然后是 `index.js`，和 TypeScript 是差不多的。

类似地，非相对导入也遵循 Node.js 解析逻辑，首先查找文件，然后查找适用的文件夹。因此，源文件 `/root/src/ moduleA.ts` 中的 `import { b } from "moduleB"` 将导致以下查找：

1.  `/root/src/node_modules/moduleB.ts`
2.  `/root/src/node_modules/moduleB.tsx`
3.  `/root/src/node_modules/moduleB.d.ts`
4.  `/root/src/node_modules/moduleB/package.json` (如果它指定了 `types` 属性)
5.  `/root/src/node_modules/@types/moduleB.d.ts`
6.  `/root/src/node_modules/moduleB/index.ts`
7.  `/root/src/node_modules/moduleB/index.tsx`
8.  `/root/src/node_modules/moduleB/index.d.ts`
9.  `/root/node_modules/moduleB.ts`
10. `/root/node_modules/moduleB.tsx`
11. `/root/node_modules/moduleB.d.ts`
12. `/root/node_modules/moduleB/package.json` (如果它指定了 `types` 属性)
13. `/root/node_modules/@types/moduleB.d.ts`
14. `/root/node_modules/moduleB/index.ts`
15. `/root/node_modules/moduleB/index.tsx`
16. `/root/node_modules/moduleB/index.d.ts`
17. `/node_modules/moduleB.ts`
18. `/node_modules/moduleB.tsx`
19. `/node_modules/moduleB.d.ts`
20. `/node_modules/moduleB/package.json` (如果它指定了 `types` 属性)
21. `/node_modules/@types/moduleB.d.ts`
22. `/node_modules/moduleB/index.ts`
23. `/node_modules/moduleB/index.tsx`
24. `/node_modules/moduleB/index.d.ts`

不要被这里的步骤数量吓到 —— TypeScript 仍然只在步骤 (9) 和 (17) 中跳转到上一个目录。这实际上并不会比 Node.js 本身所做的复杂。

## 额外模块解析标志

项目 源布局 有时与 输出布局 不匹配。通常一组构建步骤会生成最终输出。其中包括将 `.ts` 文件编译成 `.js` 文件，以及将依赖项从不同的源位置复制到单个输出位置。最终结果是模块在运行时的名称可能与包含其定义的源文件不同。或者最终输出中的模块路径在编译时可能与其对应的源文件路径不匹配。

TypeScript 编译器有一组额外的标志来通知编译器预期发生在源上的转换，以生成最终输出。

重要的是要注意编译器不会执行任何这些转换；它只是使用这些信息来指导将模块导入解析为其定义文件的过程。

### Base URL

使用 [baseUrl](https://www.typescriptlang.org/tsconfig#baseUrl) 是使用 AMD 模块加载器的应用程序中的常见做法，其中模块在运行时 "部署" 到单个文件夹。这些模块的源代码可以位于不同的目录中，但是构建脚本会将它们放在一起。

在 `tsconfig.json` 设置 `baseUrl` 通知编译器在哪里可以找到模块。所有具有非相对名称的模块导入都被假设为相对于 `baseUrl`。

baseUrl 的值被确定为：

*   baseUrl 命令行参数的值（如果给定路径是相对的，则根据当前目录计算）
*   `tsconfig.json` 中 baseUrl 属性的值（如果给定路径是相对的，则根据 `tsconfig.json` 的位置计算）

请注意，相对模块导入不受设置 baseUrl 的影响，因为它们始终相对于其导入文件进行解析。

您可以在 [RequireJS](http://requirejs.org/docs/api.html#config-baseUrl) 和 [SystemJS](https://github.com/systemjs/systemjs/blob/main/docs/api.md) 文档中找到有关 baseUrl 的更多信息。

> 此功能旨在与浏览器中的 AMD 模块加载器结合使用，不建议在任何其他情况下使用。从 TypeScript 4.1 开始，使用 paths 时不再需要设置 baseUrl。

### 路径映射

有时模块不直接位于 baseUrl 下。例如，对模块 `"jquery"` 的导入将在运行时转换为 `"node_modules/jquery/dist/jquery.slim.min.js"`。加载程序使用映射配置在运行时将模块名称映射到文件，请参阅 [RequireJs documentation](http://requirejs.org/docs/api.html#config-paths) 文档和 [SystemJS documentation](https://github.com/systemjs/systemjs/blob/main/docs/import-maps.md) 文档。

TypeScript 编译器支持使用 `tsconfig.json` 文件中的 [paths](https://www.typescriptlang.org/tsconfig#paths) 属性，声明此类映射。下面是如何为 `jquery` 指定 [paths](https://www.typescriptlang.org/tsconfig#paths) 属性的示例。

```json
{
    "compilerOptions": {
        "baseUrl": "." // 如果 "paths" 是，则必须指定。
        "paths": {
            "jquery": ["node_modules/jquery/dist/jquery"] // 这个映射是相对于 "baseUrl" 的
        }
    }
}
```

请注意，[paths](https://www.typescriptlang.org/tsconfig#paths) 是相对于 [baseUrl](https://www.typescriptlang.org/tsconfig#baseUrl) 解析的。将 `baseUrl` 设置为 `"."` 以外的值时，即 `tsconfig.json` 的目录，必须相应地更改映射。比如说，你在上面的例子中设置了 `"baseUrl"："./src"`，那么 `jquery` 应该映射到 `"../node_modules/jquery/dist/jquery"`。

使用 `paths` 还允许更复杂的映射，包括多个回退位置。假设一个项目配置，其中只有一些模块在一个位置可用，其余模块在另一个位置。构建步骤会将它们全部放在一个地方。项目布局可能如下所示：

```ts
projectRoot
├── folder1
│   ├── file1.ts (导入 'folder1/file2' and 'folder2/file3')
│   └── file2.ts
├── generated
│   ├── folder1
│   └── folder2
│       └── file3.ts
└── tsconfig.json
```

对应的 `tsconfig.Json` 看起来像这样：

```json
{
    "compilerOptions": {
        "baseUrl": ".",
        "paths": {
            "*": ["*", "generated/*"]
        }
    }
}
```

这告诉编译器任何匹配模式 `"*"`（即所有值）的模块导入，在以下两个位置查找：

1.  `"*"`：意思是相同的名称不变，所以映射 `<moduleName>` => `<baseUrl>/<moduleName>`
2.  `"generated/*"` 意思是模块名称加上前缀 "generated"，所以映射  `<moduleName>` => `<baseUrl>/generated/<moduleName>`

按照这个逻辑，编译器将尝试这样解析两个导入：

导入 'folder1/file2'：

1.  匹配模式 `"*"`，通配符捕获整个模块名称
2.  尝试列表中的第一个替换：`"*"` -> `folder1/file2`
3.  替换的结果是非相对名称 -- 将其与 baseUrl 组合 -> `projectRoot/folder1/file2.ts`
4.  文件存在，完成了。

导入 'folder2/file3'：

1.  匹配模式 `"*"`，通配符捕获整个模块名称
2.  尝试列表中的第一个替换：`"*"` -> `folder2/file3`
3.  替换的结果是非相对名称 -- 将其与 baseUrl 组合 -> `projectRoot/folder2/file3.ts`
4.  文件不存在，移动到第二个替换
5.  第二个替换 `"generated/*"` -> `generated/folder2/file3.ts`
6.  替换的结果是非相对名称 -- 将其与 baseUrl -> `projectRoot/generated/folder2/file3.ts` 组合。
7.  文件存在，完成了。

### 带有 `rootDirs` 的虚拟目录

有时候，在编译时来自多个目录的项目源全部组合在一起生成一个输出目录。这可以看作是一组源目录创建一个 "虚拟" 目录。

使用 `rootDirs`，你可以通知编译器构成这个 "虚拟" 目录的根目录；因此，编译器可以解析这些 "虚拟" 目录中的相关模块导入，就好像它们被合并到一个目录中一样。

例如考虑如下项目结构：

```ts
 src
 └── views
     └── view1.ts (imports './template1')
     └── view2.ts

 generated
 └── templates
         └── views
             └── template1.ts (imports './view2')
```

在 `src/views` 中的文件是一些 UI 控件的用户代码。 `generated/templates` 中的文件是模板生成器作为，构建的一部分自动生成的 UI 模板绑定代码。构建步骤会将 `/src/views` 和 `/generated/templates/views` 中的文件复制到输出中的同一目录。在运行时，视图可以期望它的模板存在于它旁边，因此应该使用相对名称 `"./template"` 来导入它。

要向编译器指定此关系，请使用 [rootDirs](https://www.typescriptlang.org/tsconfig#rootDirs)。 `rootDirs` 指定一个根列表，其内容预计在运行时合并。因此，根据我们的示例，`tsconfig.json` 文件应如下所示：

```json
{
  "compilerOptions": {
    "rootDirs": ["src/views", "generated/templates/views"]
  }
}
```

每次编译器在其中一个 `rootDirs`, 的子文件夹中看到相关模块导入时，它都会尝试在 `rootDirs` 的每个条目中查找此导入。

`rootDirs` 的灵活性不限于指定逻辑合并的物理源目录列表。提供的数组可能包含任意数量的临时、任意目录名称，无论它们是否存在。这允许编译器以类型安全的方式捕获复杂的绑定和运行时特性，例如，条件包含 和 项目特定的加载程序插件。

假设有一个国际化场景，其中构建工具通过插入一个特殊的路径标记（比如 `#{locale}`）作为相对模块路径的一部分（例如 `./#{locale}/messages`），来自动生成特定于区域设置的包。在这个假设的设置中，该工具枚举支持的语言环境，将抽象路径映射到 `./zh/messages`、`./de/messages` 等。

假设这些模块中的每一个都导出一个字符串数组。例如 `./zh/messages` 可能包含：

```ts
export default ["您好吗", "很高兴认识你"];
```

通过利用 `rootDirs`，我们可以将此映射通知编译器，从而允许它安全地解析 `./#{locale}/message`，即使该目录永远不存在。例如，使用下面的 \`tsconfig.json：

```ts
{
  "compilerOptions": {
    "rootDirs": ["src/zh", "src/de", "src/#{locale}"]
  }
}
```

出于工具目的，编译器现在解析 `import messages from './#{locale}/messages'` 为 `import messages from './zh/messages'`，允许在不依赖特定语言环境的情况下进行开发，不会影响设计时的支持。

## 跟踪模块解析

如前所述，编译器在解析模块时可以访问当前文件夹之外的文件。当诊断模块解析失败或解析为错误定义时，这可能会造成困扰。通过启用编译器模块解析跟踪功能，使用 [traceResolution](https://www.typescriptlang.org/tsconfig#traceResolution) 选项可以提供有关模块解析过程中发生的情况的见解。

假设我们有一个使用 typescript 模块的示例应用程序。如，`app.js` 拥有导入为：`import * as ts from "typescript"`。

```ts
│   tsconfig.json
├───node_modules
│   └───typescript
│       └───lib
│               typescript.d.ts
└───src
        app.ts
```

用 [traceResolution](https://www.typescriptlang.org/tsconfig#traceResolution) 调用编译器

```ts
tsc --traceResolution
```

结果输出如下:

```ts
======== Resolving module 'typescript' from 'src/app.ts'. ========
Module resolution kind is not specified, using 'NodeJs'.
Loading module 'typescript' from 'node_modules' folder.
File 'src/node_modules/typescript.ts' does not exist.
File 'src/node_modules/typescript.tsx' does not exist.
File 'src/node_modules/typescript.d.ts' does not exist.
File 'src/node_modules/typescript/package.json' does not exist.
File 'node_modules/typescript.ts' does not exist.
File 'node_modules/typescript.tsx' does not exist.
File 'node_modules/typescript.d.ts' does not exist.
Found 'package.json' at 'node_modules/typescript/package.json'.
'package.json' has 'types' field './lib/typescript.d.ts' that references 'node_modules/typescript/lib/typescript.d.ts'.
File 'node_modules/typescript/lib/typescript.d.ts' exist - use it as a module resolution result.
======== Module name 'typescript' was successfully resolved to 'node_modules/typescript/lib/typescript.d.ts'. ========
```

#### 注意事项

*   导入的名称和位置

```ts
======== Resolving module 'typescript' from  'src/app.ts'. ========
```

*   编译器遵循的策略

```ts
Module resolution kind is not specified, using  'NodeJs'.
```

*   从npm包中加载类型

```ts
'package.json' has 'types' field './lib/typescript.d.ts' that references 'node_modules/typescript/lib/typescript.d.ts'.
```

*   最终结果

```ts
======== Module name 'typescript' was 'successfully resolved' to 'node_modules/typescript/lib/typescript.d.ts'. ========
```

## 使用 `--noResolve`

通常，编译器会在开始编译过程之前尝试解析所有模块导入。每次成功解析到文件的 `import` 时，该文件都会添加到编译器稍后将处理的文件集中。

[noResolve](https://www.typescriptlang.org/tsconfig#noResolve) 编译器选项，命令编译器不要将任何未在命令行上传递的文件 "添加" 到编译中。它仍然会尝试将模块解析为文件，但如果未指定文件，则不会包含该文件。

例如：

**app.js**

```ts
import * as A from "moduleA"; // OK, 'moduleA' passed on the command-line

import * as B from "moduleB"; // Error TS2307: Cannot find module 'moduleB'.
```

```ts
tsc app.ts moduleA.ts --noResolve
```

使用 [noResolve](https://www.typescriptlang.org/tsconfig#noResolve) 编译 `app.ts` 会导致:

*   当 `moduleA` 在命令行上传递时，正确地找到它。
*   没有找到 `moduleB`，因为它没有被传递。

## 常见问题

为什么排除列表中的模块仍会被编译器选中？

`tsconfig.json` 将一个文件夹变成一个"项目"。在不指定任何 `"exclude"` 或 `"files"` 条目的情况下，文件夹中的所有子目录及`tsconfig.json` ，和所有子目录中的所有文件，都包含在你的编译中。如果你想排除一些文件，可以使用 `"exclude"`，如果你宁愿指定所有文件而不是让编译器查找它们，使用 `"files"`。

那是 `tsconfig.json` 自动包含。如上所述，这没有嵌入模块解析。如果编译器将某个文件识别为模块导入的目标，则无论它是否在前面的步骤中被排除在外，它都将包含在编译中。

因此，要从编译中排除一个文件，你需要排除它之外，还需要排除所有 `import` 它或 `/// <reference path="..." />` 指令引用它的文件。

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/module-resolution.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
