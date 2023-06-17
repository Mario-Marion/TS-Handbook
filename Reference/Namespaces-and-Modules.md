# 命名空间和模块

这篇文章概述了在 TypeScript 中使用模块和命名空间组织代码的各种方法。我们还将讨论一些关于如何使用命名空间和模块的高级话题，并解决在 TypeScript 中使用它们时的一些常见缺陷。

有关 ES 模块的更多信息，请参阅 [Modules](https://juejin.cn/post/7212840443494219835)。有关 TypeScript 命名空间的更多信息，请参阅 [Namespaces](https://juejin.cn/post/7245164914000298042)。

注意：在非常老的 TypeScript 版本中，命名空间被称为“内部模块”，早于 JavaScript 模块系统的时候。

## 使用模块

模块可以同时包含代码和声明。

模块还依赖于模块加载器（如 CommonJs/Require.js）或支持 ES 模块的运行时。模块提供了更好的代码重用、更强的隔离和更好的捆绑工具支持。

同样值得注意的是，对于 Node.js 应用程序，模块是默认的，我们**建议在现代代码中使用模块而不是命名空间**。

从 ECMAScript 2015 开始，模块是原生语言的一部分，应该被所有兼容的引擎实现所支持。因此，对于新项目，模块将是推荐的代码组织机制。

## 使用命名空间

命名空间是 Typescript 特有的一种组织代码的方式。

命名空间就是在全局命名空间中简单的命名一个 JavaScript 对象。这使得命名空间成为一个非常简单的结构。与模块不同，它们可以跨越多个文件，并且可以使用 [outFile](https://www.typescriptlang.org/tsconfig#outFile) 将它们进行连接。命名空间是在 Web 应用程序中构建代码的好方法，所有依赖关系都包含在你的 HTML 页面中的 \<script> 标签中。

但就像所有全局命名空间污染一样，很难识别组件依赖关系，尤其是在大型应用程序中。

## 命名空间和模块的缺陷

在本节中，我们将描述使用 命名空间 和 模块 时的各种常见陷阱，以及如何避免它们。

### 使用 `/// <refrence>` 引用一个模块

一个常见的错误是试图使用 `/// <reference ... />` 语法来引用模块文件，而不是使用 `import` 语句。要理解这种区别，我们首先需要了解编译器是如何根据导入路径（例如，在`import x from "...";` , `import x = require("...");`， 等等，中的 `...`）定位模块的类型信息的。

编译器将尝试找到 `.ts`， `.tsx`，然后是具有适当路径的 `.d.ts`。如果找不到特定的文件，那么编译器将查找 [环境模块声明](https://juejin.cn/post/7245164914000298042#heading-6)，环境模块声明需要在 `.d.ts` 文件中声明。

**myModules.d.ts**

```ts
// 在 .d.ts 文件或 .ts 文件，该文件不是一个模块：
declare module "SomeModule" {
  export function fn(): string;
}
```

**myOtherModule.ts**

```ts
/// <reference path="myModules.d.ts" />
import * as m from "SomeModule";
```

这里的引用标记允许我们定位包含环境模块声明的声明文件。

## 没必要的命名空间

如果你正在将一个程序从命名空间转换为模块，很容易得到如下所示的文件：

**shaps.ts**

```ts
export namespace Shapes {
  export class Triangle {
    /* ... */
  }
  export class Square {
    /* ... */
  }
}
```

这里的顶级命名空间 `Shapes` 包含了 `Triangle` 和`Square`。这对你的模块的使用者来说是困惑和讨厌的：

**shapeConsumer.ts**

```ts
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

TypeScript 模块的一个关键特性是，两个不同的模块永远不会向同一个作用域中添加名称。由模块的使用者决定给它分配名称，所以没有必要提前将导出的符号封装在命名空间中。

再次强调为什么不应该尝试在模块中使用命名空间：因为命名空间的思想是提供构造形状的逻辑分组，并防止名称冲突。而模块文件本身已经是一个逻辑分组，并且其顶级名称由导入它的代码定义，所以不需要使用额外的模块层来导出对象。

下面是一个修改后的例子：

**shapes.ts**

```ts
export class Triangle {
  /* ... */
}
export class Square {
  /* ... */
}

```

**shapeConsumer.ts**

```ts
import * as shapes from "./shapes";
let t = new shapes.Triangle();
```

## 模块的权衡

正如 JS 文件和模块之间存在一一对应的关系一样，TypeScript 在模块源文件和编译后的 JS 文件之间也是一一对应的关系。这样做的一个影响是无法根据你的目标模块系统，将多个模块源文件合并为一个文件。例如，你不能在 [module](https://www.typescriptlang.org/tsconfig#module)： `commonjs` 或 `umd` 为配置目标时，使用 [outFile](https://www.typescriptlang.org/tsconfig#outFile) 选项，但对于 [TypeScript 1.8](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-1-8.html#concatenate-amd-and-system-modules-with---outfile) 及更高版本，可以在 `amd` 或 `system` 为目标时使用 [outFile](https://www.typescriptlang.org/tsconfig#outFile) 选项。

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/namespaces-and-modules.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>



