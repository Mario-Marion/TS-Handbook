# Module-Class.d.ts

当 super-greeter 库暴露的是一个函数时：

```ts
import greeter from "super-greeter";

greeter(2);
greeter("Hello world");
```

处理通过 UMD 和模块导入：

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是一个函数模块模板文件，应该重命名为 index.d.ts
 *~ 并将其放入与模块同名的文件夹中
 *~ 例如, 如果你为 "super-greeter" 编写了一个文件
 *~ 这个文件应该在 'super-greeter/index.d.ts'
 */

// 注意，ES6模块不能直接导出类对象
// 这个文件应该使用 CommonJS 风格导入
// import x = require('[~THE MODULE~]');
//
// 或则, 如果开启了 --allowSyntheticDefaultImports 或
// --esModuleInterop , 这个文件也可以
// 作为默认导入去导入:
// import x from '[~THE MODULE~]';
//
// 参考 TypeScript 文档
// https://www.typescriptlang.org/docs/handbook/modules.html#export--and-import--require
// 了解 ES6 Moudles 对此限制的常见解决方法

/*~ 如果这个模块是一个 UMD 模块
 *~ 当在模块加载器环境之外加载，暴露了一个全局变量 'myFuncLib'
 *~ 在这声明这个全局变量
 *~ 否则，删除这个声明
 */
export as namespace "myFuncLib";

/*~ 此声明指定类构造函数
 *~ 是从文件导出的对象
 */
export = Greeter;

/*~ 这里展示了如何为函数设置多个重载 */
declare function Greeter(name: string): Greeter.NamedReturnType;
declare function Greeter(length: number): Greeter.LengthReturnType;

/*~ 如果你也想在暴露模块中的类型，你可以
 *~ 把它们放在一块. 通常你想要描述
 *~ 函数返回类型的形状；这种声明应该
 *~ 在这里声明，如下所示.
 *~
 *~ 注意，如果你决定包含这个命名空间, 模块可能会
 *~ 错误的导入命名空间对象, 除非
 *~ 开启了 --esModuleInterop:
 *~ import * as x from '[~THE MODULE~]'; // 错误! 不要这么做!
 */

declare namespace Greeter {
    export interface LengthReturnType {
        width: number;
        height: number;
    }

    export interface NamedReturnType {
        firstName: string;
        lastName: string;
    }

/*~ 如果模块也有属性, 在这里声明它们. 例如,
 *~ 这个声明表示该代码是合法的：
 *~ import f = require('super-greeter');
 *~ console.log(f.defaultName);
 */
    export const defaultName: string;
    export let defaultLength: number;
}
```


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-function-d-ts.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
