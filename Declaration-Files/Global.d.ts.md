# Global Libraries

全局库是可以在全局作用域访问的库。（不用使用 `import` 导入任何东西）。大多数库只是暴露一个或多个全局变量。例如，如果你使用过 [jQuery](https://jquery.com/)，可通过 `$` 变量直接使用。

```ts
$(() => {
    console.log("hello!");
});
```

你通常会在全局库的文档中看到如何在 HTML 脚本标签中使用库：

```js
<script src="http://a.great.cdn.for/someLib.js"></script>
```

现如今，大多数流行的全局可访问库，实际上都编写为 UMD 库（见下文）。UMD 库文档很难和全局库文档区分开来。在编写全局库声明文件之前，请确保该库不是 UMD 库。

## 从代码识别全局库

全局库代码通常是非常简单的。一个全局的 "Hello,world" 库可能看起来像这样：

```js
function createGreeting(s) {
    return "Hello, " + s;
}
```

或者像这样：

```js
window.createGreeting = function (s) {
    return "Hello, " + s;
};
```

当查看全局库代码时，通常会看到：

- 顶级 `var` 声明 或 `function` 声明
- 给 `window` 赋值一个或多个属性
- 假设像 `document` 或 `window` DOM 原始对象存在

你不可能看到：

- 检查像 `require` 或 `define` 等，模块加载器的使用情况
- `var fs = require("fs)` CommonJS/Node.js 风格的导入
- 调用 `define(...)`
- 文档描述如何去使用 `require` 或 `import` 导入库

## 全局库示例

因为全局库转换为 UMD 库很简单，所以很少有流行库还会编写全局风格，除非是很小而且需要 DOM （或没有依赖项）的库仍然使用全局风格。

## 全局库模板

以下是一个全局库 DTS 示例

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 如果该库是可调用的 (例如：你可以调用 myLib(3)),
 *~ 这里包含了那些调用签名
 *~ 否则, 删除该部分
 */
declare function myLib(a: string): string;
declare function myLib(a: number): number;

/*~ 如果希望该库的名称为有效的类型名称，
 *~ 你可以在这里这么做
 *~
 *~ 例如，这里允许我们写 'var x: myLib';
 *~ 确保这是有意义的! 如果没有
 *~ 只需删除该声明并且在下面的命名空间中添加类型
 */
interface myLib {
  name: string;
  length: number;
  extras?: string[];
}

/*~ If your library has properties exposed on a global variable,
 *~ 把它们放在这里.
 *~ 你还应该在这里放置类型 (接口和类型别名)
 */
declare namespace myLib {
  //~ 我们可以写 'myLib.timeout = 50;'
  let timeout: number;
  
  //~ 我么可以访问 'myLib.version'，但不能改变它
  const version: string;
  
  //~ 我们可以通过 'let c = new myLib.Cat(42)' 创建一些类
  //~ 或引用，例如：'function f(c: myLib.Cat) { ... }
  class Cat {
    constructor(n: number);
    
    //~ 我们可以从 'Cat' 实例中阅读 'c.age'
    readonly age: number;
    
    //~ We can invoke 'c.purr()' from a 'Cat' instance
    purr(): void;
  }
  //~ 我们可以声明一个变量为
  //~ 'var s: myLib.CatSettings = { weight: 5, name: "Maru" };'
  interface CatSettings {
    weight: number;
    name: string;
    tailLength?: number;
  }
  
  //~ 我么可以写 'const v: myLib.VetID = 42;'
  //~ 或 'const v: myLib.VetID = "bob";'
  type VetID = string | number;
  
  //~ 我们可以调用 'myLib.checkCat(c)' 
  //~ 或 'myLib.checkCat(c, v);'
  function checkCat(c: Cat, s?: VetID);
}
```


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-d-ts.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

