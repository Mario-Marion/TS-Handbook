# Module-Class.d.ts

当 super-greeter 库暴露的是一个类时：

```ts
const Greeter = require("super-greeter");
const greeter = new Greeter();
greeter.greet();
```

处理通过 UMD 和模块导入：

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是一个类模块模板文件，应该重命名为 index.d.ts
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
 *~ 当在模块加载器环境之外加载，暴露了一个全局变量 'myClassLib'
 *~ 在这声明这个全局变量
 *~ 否则，删除这个声明
 */
export as namespace "super-greeter";

/*~ 此声明指定类构造函数
 *~ 是从文件导出的对象
 */
export = Greeter;

/*~ 在这个类中编写模块的方法和属性 */
declare class Greeter {
    constructor(customGreeting?: string);
    greet: void;
    myMethod(opts: MyClass.MyClassMethodOptions): number;
}

/*~ 如果你想暴露模块中的其它类型，可以把它们放在一块
 *~
 *~ 注意，如果你决定包含此命名空间，则模块可能
 *~ 被错误的导入到命名空间对象, 除非
 *~ --esModuleInterop 是开启的:
 *~ import * as x from '[~THE MODULE~]'; // 错误! 不要这么做
 */
declare namespace MyClass {
    export interface MyClassMethodOptions {
        width?: number;
        height?: number;
    }
}
```


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-class-d-ts.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

