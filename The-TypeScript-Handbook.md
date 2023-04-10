# The TypeScript Handbook
## 关于本手册
&emsp;&emsp;在引入编程社区20年后，JavaScript 现在是有史以来最广泛跨平台语言之一。开始只是一个给网页添加琐碎交互性的小脚本，JavaScript 已经成长为各种规模的前、后端应用的首选语言。虽然用 JavaScript 编写的程序，大小，范围和复杂度呈指数性增长。JavaScript 没有表达不同代码单位之间关系的能力。结合 JacaScript 非常奇特的运行时语义，语言和程序复杂性之间的不匹配造成了 JavaScript 开发是项难以大规模管理的任务。

&emsp;&emsp;程序员编写程序的最常见错误可以称为类型错误：在与期待的值不同的地方使用了某种类型的值，这可能是由于简单的拼写错误，没有理解库的 API，对运行时行为的错误假设，或其它错误造成的。TypeScript 的目标是成为 JavaScript 程序的静态类型检查器，或者说，是一个在你运行代码之前运行(静态)的工具，并确保程序类型的正确(类型检查)。

&emsp;&emsp;如果你在没有学过 JavaScript 的情况下来到 TypeScript，希望把 TypeScript 成为你第一门语言，我们推荐你先阅读 [Microsoft Learn JavaScript tutorial](https://docs.microsoft.com/javascript/) 或 [JavaScript at the Mozilla Web Docs](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide) 任一文档。如果你有其它语言的经验，阅读这些文档你应该能够相当快速的掌握 JavaSCript 语法。

## 本手册结构怎么样
手册分成两个部分
- **The Handbook**

  TypeScript 手册打算成为一个综合文档，解释程序员日常使用的 TypeScript 。

  你可以从上到下去阅读手册

  你应该期望每一章或每一页给出的概念能给你深刻的理解。TypeScript 手册不是完整的语言规范，但打算成为语言的所有特性和行为的综合指南

  完成的了本手册的读者应该能：

  - 理解 TypeScript 的常用语法和模式
  - 解释重要编译器选项的影响
  - 在大多数情况下正确预测类型系统行为

  手册的主要内容不会探讨涵盖每个边缘情况或特性的细节。你可以在文章中的参考链接找到特定概念的更多细节
- **Reference Files**

  Reference 部分，对 TypeScript 特定部分如何工作提供了的更丰富的解释。你可以从上到下阅读，但是每个部分都是对单独概念更深层次的说明，这意味着没有按顺序阅读也没关系。

## Non-Goals
&emsp;&emsp;本手册也打算成为一份简洁的文档，能够在几小时内轻松阅读。为了保持简短，将不被包括某些论题。

&emsp;&emsp;具体来说，该手册并未完整介绍函数、类和闭包等核心 JavaScript 基础知识。在适当的地方，我们将包含特定知识点的链接，你可以通过它们学习这些概念。

&emsp;&emsp;本手册同样没有打算取代语言规范。在某些情况下，边缘情况或行为的正式描述将略过，支持更高级，更容易理解的解释。反而，一些单独的参考页面更精确、更正式地描述了TypeScript 行为的许多方面。这些参考页不适合不熟悉 TypeScript 的读者，他们可能会使用高级术语或引用你还没有读过的论题。

最后，本手册不会涉及 TypeScript 如何与其他工具交互，除非是必要的。像如何使用 webpack、rollup、parcel、react、babel、closure、lerna、rush、bazel、preact、vue、angular、slte、jquery、yarn 或 npm 配置 TypeScript 之类的论题不在讨论范围之内——你可以在网上的其他地方找到这些资源。

## Get Started
在从 [The Basics](https://www.typescriptlang.org/docs/handbook/2/basic-types.html) 开始之前，我们推荐你阅读以下介绍页面之一。这些介绍旨在突出 TypeScript 和你喜欢的编程语言之间的异同点，并清除你对这些语言的常见误解。

- [TypeScript for New Programmers](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html)
- [TypeScript for JavaScript Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
- [TypeScript for OOP Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-oop.html)
- [TypeScript for Functional Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html)

直接开始 [The Basics](https://github.com/Mario-Marion/TS-Handbook/blob/main/Handbook/The-Basics__%E4%B8%80.md)