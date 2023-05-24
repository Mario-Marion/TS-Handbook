# Type Inference

在TypeScript中，有几个地方在没有显式类型注释的情况下，会使用类型推断来提供类型信息。例子：

```ts
let x = 3;
// let x: number
```

变量 `x`  类型被推断为 `number` 类型。这种推断发生在初始化变量和成员、设置参数默认值和确定函数返回值。

在大多数情况下，类型推断是直接的。接下来我们将探讨类型推断方式中的一些细微差别。

## 最佳公共类型

当类型推断来自多个表达式时，将使用这些表达式的类型计算出 "最佳公共类型"。例子：

```ts
let x = [0, 1, null];
// let x: (number | null)[]
```

上面的例子中推断 `x` 的类型，必须考虑每个数组元素的类型。这里提供了数组元素类型两种选择， `number` 和 `null`。最佳公共类型算法会考虑每个候选类型，并选择的类型与其它所有候选类型兼容。

因为必须从提供的候选类型中选择最佳公共类型，所以在某些情况下，类型共享一个公共结构。但是有时候，没有一个类型是所有候选类型的超类型。例子：

```ts
class Animal { }
class Rhino extends Animal {
  hasHorn!: true;
}
class Elephant extends Animal {
  hasTrunk!: true;
}
class Snake extends Animal {
  hasLegs!: false;
}
// ---cut---
let zoo = [new Rhino(), new Elephant(), new Snake()];
// let zoo: (Rhino | Elephant | Snake)[]
```

当没有找到最佳公共类型时，结果推断为联合数组类型 `(Rhino | Elephant | Snake)[]`。

理想情况下，我们可能希望将 `zoo` 推断为一个 `Animal[]` 类型，但是因为数组中没有严格类型为 `Animal` 的对象，所以推断成了一个联合数组类型。

我们可以不推断数组元素的类型，直接提供一个显式类型，例子：

```ts
class Animal { }
class Rhino extends Animal {
  hasHorn!: true;
}
class Elephant extends Animal {
  hasTrunk!: true;
}
class Snake extends Animal {
  hasLegs!: false;
}
// ---cut---
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
// let zoo: Animal[]
```

## 上下文类型化

在 TypeScript 的某些情况下，类型推断也会以 "另一个方向" 工作。就是所谓的 "上下文类型"。当表达式通过其所处的位置而产生隐性类型，就是上下文类型化。例如：

```ts
window.onmousedown = function (mouseEvent) {
    console.log(mouseEvent.button);
    console.log(mouseEvent.kangaroo);
     Error: // Property 'kangaroo' does not exist on type 'MouseEvent'.
};
```

例子中，TypeScript 类型检查器使用 `Window.onmousedown` 方法类型去推断在赋值右侧的函数表达式类型。它可以推断出参数 `mouseEvent` 的 [类型](https://developer.mozilla.org/zh-CN/docs/Web/API/MouseEvent)，并且包含 `button` 属性，但是没有 `kangaroo` 属性。

这是因为 window 已经在它的类型中声明了 `onmousedown`：

```ts
// Declares there is a global variable called 'window'
declare var window: Window & typeof globalThis;

// Which is declared as (simplified):
interface Window extends GlobalEventHandlers {
    // ...
}

// Which defines a lot of known handler events
interface GlobalEventHandlers {
    onmousedown: ((this: GlobalEventHandlers, ev: MouseEvent) => any) | null;
    // ...
}
```

TypeScript 足够聪明，可以在其它上下文中推断类型：

```ts
window.onscroll = function (uiEvent) {
    console.log(uiEvent.button);
     Error: // Property 'button' does not exist on type 'Event'.Property 'button' does not exist on type 'Event'.
};
```

例子中，TypeScript 知道 `uiEvent` 是 [UIEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/UIEvent)，而不是像之前例子的 [MouseEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/MouseEvent)。`UIEvent` 对象没有 `button` 属性，所以 TypeScript 将抛出异常。

如果这个函数没有在上下文类型位置，那么函数参数将具有隐性的 `any` 类型，并且不会发出错误（除非你启用 [noImolicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) 选项）：

```ts
const handler = function (uiEvent) {
    console.log(uiEvent.button); // <- OK
};
```

还可以显式地为函数参数提供类型信息，覆盖任何上下文类型:

```ts
window.onscroll = function (uiEvent: any) {
    console.log(uiEvent.button); // <- Now, no error is given
};
```

例子中，将打印 `undefined`，因为 `uiEvent` 没有 `button` 属性。

上下文类型化应用在许多场景，如：函数调用的参数、赋值右侧、类型断言、对象和数组字面量成员、返回语句。上下文类型也充当最佳公共类型中的候选类型。例子：

```ts
class Animal { }
class Rhino extends Animal {
  hasHorn!: true;
}
class Elephant extends Animal {
  hasTrunk!: true;
}
class Snake extends Animal {
  hasLegs!: false;
}
// ---cut---
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
}
```

例子中，最佳公共类型有四个候选类型：`Animal`、`Rhino`、`Elephant`、`Snake`。其中 `Animal` 可以通过最佳公共类型算法来选择。

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/type-inference.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
