# Namespaces

这篇文章概述了在 TypeScript 中使用名称空间（以前称为“内部模块”）组织代码的各种方法。此外，在声明内部模块时使用 `module` 关键字的任何地方，都可以使用 `namespace` 关键字来代替。

## 单个文件中的验证器

首先，我们编写了一组简单的字符串验证器，用于检查用户在网页表单上的输入或检查外部提供的数据文件的格式。

```ts
interface StringValidator {
  isAcceptable(s: string): boolean;
}
let lettersRegexp = /^[A-Za-z]+$/;
let numberRegexp = /^[0-9]+$/;
class LettersOnlyValidator implements StringValidator {
  isAcceptable(s: string) {
    return lettersRegexp.test(s);
  }
}
class ZipCodeValidator implements StringValidator {
  isAcceptable(s: string) {
    return s.length === 5 && numberRegexp.test(s);
  }
}
// 要验证的字符串
let strings = ["Hello", "98052", "101"];
// 新建对象，包含两个验证器
let validators: { [s: string]: StringValidator } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();
// 循环调用验证器
for (let s of strings) {
  for (let name in validators) {
    let isMatch = validators[name].isAcceptable(s);
    console.log(`'${s}' ${isMatch ? "matches" : "does not match"} '${name}'.`);
  }
}
```

## 命名空间

现在，我们添加更多的验证器时，希望有某种组织模式，这样我们就可以跟踪我们的类型，而不用担心与其他对象的名称冲突。与其将许多不同的名称放入全局命名空间，不如将对象打包到一个命名空间中。

在本例中，我们将把所有与验证器相关的实体移到一个名为 `Validation` 的命名空间中。我们希望这里的接口和类在命名空间之外可见，可使用 `export` 作为它们的开头。相反，变量 `lettersRegexp` 和 `numberRegexp` 是实现细节，命名空间之外的代码不可见，所以不会被导出。在 `index` 文件中，我们需要使用特定的类型名称去引用命名空间，例如 `Validation.StringValidator`。

```ts
// validators.ts

namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
  const lettersRegexp = /^[A-Za-z]+$/;
  const numberRegexp = /^[0-9]+$/;
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s);
    }
  }
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}
```
```ts
// index.ts

// 要验证的字符串
let strings = ["Hello", "98052", "101"];
// 新建对象，包含两个验证器
let validators: { [s: string]: Validation.StringValidator } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();
// 循环调用验证器
for (let s of strings) {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? "matches" : "does not match"
      } ${name}`
    );
  }
}
```

## 跨文件拆分

随着应用程序的增长，我们希望将代码拆分为多个文件，以使其更易于维护。

### 多文件命名空间

在这里，我们将把 `Validation` 命名空间拆分为多个文件。尽管这些文件是分开的，但它们每个都可以合并到同一个命名空间，并且可以像在一个地方定义的一样使用它们。因为文件之间存在依赖关系，所以我们将添加引用标记来告诉编译器文件之间的关系。现在版本的 TypeScript 已经不需要添加引用标记了。

**Validation.ts**

```ts
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
}
```

**LettersOnlyValidator.ts**

```ts
/// <reference path="Validation.ts" />
namespace Validation {
  const lettersRegexp = /^[A-Za-z]+$/;
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
      return lettersRegexp.test(s);
    }
  }
}
```

**ZipCodeValidator.ts**

```ts
/// <reference path="Validation.ts" />
namespace Validation {
  const numberRegexp = /^[0-9]+$/;
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}
```

**Test.ts**

```ts
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

let strings = ["Hello", "98052", "101"];
// 新建对象，包含两个验证器
let validators: { [s: string]: Validation.StringValidator } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();
// 循环调用验证器
for (let s of strings) {
  for (let name in validators) {
    console.log(
      `"${s}" - ${
        validators[name].isAcceptable(s) ? "matches" : "does not match"
      } ${name}`
    );
  }
}
```

一旦涉及到多个文件，我们需要确保加载所有已编译的代码。有两种方法可以做到。

首先，我们可以使用 [outFile](https://www.typescriptlang.org/tsconfig#outFile) 选项将所有的输入文件编译成一个 JavaScript 输出文件:

```shell
tsc --outFile sample.js Test.ts
```

编译器将根据文件中存在的参考标记自动排序输出文件。也可以单独指定每个文件:

```shell
tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

或者，我们可以把每个文件编译（默认）输入为单独的 JavaScript 文件。如果生成了多个 JS 文件，我们需要在网页上使用 \<script> 标签以适当的顺序加载每个发出的文件，例如：

```ts
<script src="Validation.js" type="text/javascript" />
<script src="LettersOnlyValidator.js" type="text/javascript" />
<script src="ZipCodeValidator.js" type="text/javascript" />
<script src="Test.js" type="text/javascript" />
```

## 别名

另一种简化命名空间工作的方法是使用 `import q = x.y.z` 语法，为常用对象创建较短的名称。不要与用于加载模块的 `import x = require("name")` 语法混淆，该语法只是为指定的符号创建一个别名。你可以将别名用于任何类型的标识符，包括从模块导入创建的对象。

```ts
namespace Shapes {
  export namespace Polygons {
    export class Triangle {}
    export class Square {}
  }
}
import polygons = Shapes.Polygons;
let sq = new polygons.Square(); // Same as 'new Shapes.Polygons.Square()'
```

注意，我们没有使用 `require` 关键字；直接从要导入的符号的限定名中赋值。这类似于使用 `var`，但是 `import` 是与原始符号不同的引用，因此对别名的更改不会反映到原始符号。

## 与其他 JavaScript 库一起工作

为了描述不是用 TypeScript 编写的库的形状，我们需要声明库公开的 API。因为大多数 JavaScript 库只公开几个顶级对象，所以命名空间是表示它们的好方法。

我们把没有定义实现的声明称为“环境”。通常这些是用 `.d.ts` 定义的。如果你熟悉 C/C++，你可以将这些文件视为 `.h` 文件。让我们看以下例子。

流行库 D3，在一个名为 D3 的全局对象中定义了它的功能。因为这个库是通过 \<script> 标签加载的（而不是模块加载器），所以它的声明使用命名空间来定义它的形状。为了让 TypeScript 编译器看到这个形状，我们使用了一个环境命名空间声明。例如，我们可以这样写:

**D3.d.ts (simplified excerpt)**

```ts
declare namespace D3 {
  export interface Selectors {
    select: {
      (selector: string): Selection;
      (element: EventTarget): Selection;
    };
  }
  export interface Event {
    x: number;
    y: number;
  }
  export interface Base extends Selectors {
    event: Event;
  }
}
declare var d3: D3.Base;
```

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/namespaces.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

