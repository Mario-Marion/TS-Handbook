# Gobal-modifying Modules

"全局修改模块" 改变已经导入全局作用域中的值。

例如：一个在导入时向 `String.prototype` 添加新成员的库。（由于在运行时有冲突的可能性，这种模式有些危险，但我们仍然可以为它编写声明文件。）

## 识别全局修改模块

全局修改模块通常很容易从其文档中识别出来。一般来说，它们类似于全局插件，但需要一个 `require` 调用来激活它们的效果。

你可能会看到这样的文档：

```js
// 调用 'require'，不使用它的返回值
var unused = require("magic-string-time");
/* 或 */
require("magic-string-time");

var x = "hello, world";
// 该库已经在内置类型上创建新方法
console.log(x.startsWithHello());

var y = [1, 2, 3];
// 该库已经在内置类型上创建新方法
console.log(y.reverseAndSort());
```

这有一个示例模板：

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是一个全局修改模块模板文件，应该重命名为 index.d.ts 
 *~ 并将其放入与模块同名的文件夹中。例如, 如果你为
 *~ "super-greeter" 编写了一个文件
 *~ 这个文件应该在 'super-greeter/index.d.ts'
 */
 
/*~ 注意：如果你的全局修改模块是可调用的，或可构造的
 *~ 需要将该模式与 类模块或函数模块 的模式相结合
 */
declare global {
  /*~ 这里，声明全局命名空间的内容，或扩展
   *~ 已存在全局命名空间的声明
   */
  interface String {
    fancyFormat(opts: StringFormatOptions): string;
  }
}

/*~ 如果你的模块导出类型或值，像往常一样编写它们 */
export interface StringFormatOptions {
  fancinessLevel: number;
}

/*~ 例如，在模块上声明一个方法（除了全局副作用外） */
export function doSomething(): void;

/*~ 如果你的模块没有导出任何内容，则需要这一行。否则，删除它 */
export {};
```


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-modifying-module-d-ts.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

