# Module-Plugins.d.ts

当你想要使用 JavaScript 代码扩展另一个库时。

```js
import { greeter } from "super-greeter";

// Greeter API
greeter(2);
greeter("Hello world");

// 在运行时用新函数扩展对象
import "hyper-super-greeter";
greeter.hyperGreet();
```

"super-greeter" 的定义：

```ts
/*~ 拥有多个重载函数 */
export interface GreeterFunction {
    (name: string): void
    (time: number): void
}

/*~ 导出函数 */
export const greeter: GreeterFunction;
```

我们可以像下面这样扩展现有模块：

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ 这是一个模块插件模板文件，应该重命名为 index.d.ts
*~ 并将其放入与模块同名的文件夹中
*~ 例如, 如果你为 "super-greeter" 编写了一个文件
*~ 这个文件应该在 'super-greeter/index.d.ts'
*/

/*~ 这一行，导入要扩展的模块 */
import { greeter } from "super-greeter";

/*~ 在这里，声明与上面导入的模块相同的模块
*~ 然后扩展模块中的 GreeterFunction 函数
*/

export module "super-greeter" {
    export interface GreeterFunction {
        /** Greets even better! */
        hyperGreet(): void;
    }
}
```

这里使用了 [声明合并](https://juejin.cn/post/7222532260975689785#heading-7) 中的模块扩展。

## ES6对模块插件的影响

一些插件在现有模块上添加或修改顶级导出。虽然这在 CommonJS 和其他加载器中是合法的，但 ES6 模块被认为是不可变的，因此这种模式是不可能的。因为 TypeScript 是与加载器无关的，所以在编译时不会强制执行此策略，但打算过渡到 ES6 模块加载器的开发人员应该意识到这一点。

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-plugin-d-ts.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
