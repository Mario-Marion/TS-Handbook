# Mixins

除了传统的 OO 层次结构，另一种可重用组件构建类的流行方法是，通过组合更简单的部分类来构建它们。你可能熟悉 Scala 等语言的 mixins 或 traits 的概念，而且这种模式在 JavaScript 社区中也很流行。

## mixin 是如何工作的?

该模式依赖使用带有类继承的泛型，去扩展基类。TypeScript 最好的 mixin 支持是通过类表达式模式实现的。该模式如何在 JavaScript 中工作，可参考 [这里](https://justinfagnani.com/2015/12/21/real-mixins-with-javascript-classes/)。

首先，我们需要一个类，将在上面应用 mixins：

```ts
class Sprite {
    name = "";
    x = 0;
    y = 0;

    constructor(name: string) {
        this.name = name;
    }
}
```

然后你需要一个类型和一个工厂函数，返回一个扩展基类的类表达式

```ts
// 首先，我们需要一个类型，用于扩展其它类
// 其主要职责是声明传入的类型是一个类
 
type Constructor = new (...args: any[]) => {};
 
// 这个 mixin 添加了一个 scale 属性，带有 getter 和 setter 
// 用一个密封的私有属性来改变它
 
function Scale<TBase extends Constructor>(Base: TBase) {
  return class Scaling extends Base {
    // Mixins 不能声明 private/protected 属性
    // 然而, 你可以使用 ES2020 的私有字段（#）
    _scale = 1;
 
    setScale(scale: number) {
      this._scale = scale;
    }
 
    get scale(): number {
      return this._scale;
    }
  };
}
```

这些都设置好后，你可以创建一个类，它表示应用了 mixins 的基类：

```ts
// 根据 Sprite 构造一个新类
// 应用混入 Scale
const EightBitSprite = Scale(Sprite);

const flappySprite = new EightBitSprite("Bird");
flappySprite.setScale(0.8);
console.log(flappySprite.scale);
```

## 约束 mixins

在上面的形式中，mixin 没有类的底层知识，这使得很难创建你想要的设计。

所以，我们修改原始构造函数类型，去接受泛型参数。

```ts
// This was our previous constructor:
type Constructor = new (...args: any[]) => {};
// Now we use a generic version which can apply a constraint on
// the class which this mixin is applied to
type GConstructor<T = {}> = new (...args: any[]) => T;
```

只允许创建受基类约束的类：

```ts
type Positionable = GConstructor<{ setPos: (x: number, y: number) => void }>;

type Spritable = GConstructor<Sprite>;

type Loggable = GConstructor<{ print: () => void }>;
```

然后你可以创建 mixin，它只在一个特定的基类上才能工作：

```ts
type GConstructor<T = {}> = new (...args: any[]) => T;

type Positionable = GConstructor<{ setPos: (x: number, y: number) => void }>;

function Jumpable<TBase extends Positionable>(Base: TBase) {
  return class Jumpable extends Base {
    jump() {
      // 这个 mixin 将只有传递一个基类才会工作
      // 类拥有 setPos 定义， 因为受到 Positionable 的约束
      this.setPos(0, 20);
    }
  };
}
```

## 替代模式

本文档以前的版本推荐了另一种编写 mixin 的方法，其中你分别创建了运行时和类型层次结构，最后合并它们：

```ts
// 每个 mixin 是一个传统的 ES class
class Jumpable {
  jump() {}
}

class Duckable {
  duck() {}
}

// 包括基类
class Sprite {
  x = 0;
  y = 0;
}

// 创建一个合并接口，和你的基类名称相同
interface Sprite extends Jumpable, Duckable {}

// 通过 JS 运行时，应用 mixin 到基类
applyMixins(Sprite, [Jumpable, Duckable]);

let player = new Sprite();
player.jump();
console.log(player.x, player.y);

// 这可以存在你的代码库任何地方
function applyMixins(derivedCtor: any, constructors: any[]) {
  constructors.forEach((baseCtor) => {
    Object.getOwnPropertyNames(baseCtor.prototype).forEach((name) => {
      Object.defineProperty(
        derivedCtor.prototype,
        name,
        Object.getOwnPropertyDescriptor(baseCtor.prototype, name) ||
          Object.create(null)
      );
    });
  });
}
```

这种模式更少地依赖于编译器，而更多地依赖于代码库，以确保运行时和类型系统正确地保持同步。

## 约束

代码流分析在 TypeScript 编译器内部支持 mixin 模式。在一些情况下，你可以触及本地支持的边界：

### Decorators and Mixins [`#4881`](https://github.com/microsoft/TypeScript/issues/4881)

你不能通过代码流分析使用装饰器来提供 mixin：

```ts
// 一个复制混入模式的装饰器:
const Pausable = (target: typeof Player) => {
  return class Pausable extends target {
    shouldFreeze = false;
  };
};

@Pausable
class Player {
  x = 0;
  y = 0;
}

// Player 类没有合并装饰器类型:
const player = new Player();
player.shouldFreeze;
Error：// Property 'shouldFreeze' does not exist on type 'Player'.

// 可用 类型断言 或 接口合并 解决报错
type FreezablePlayer = Player & { shouldFreeze: boolean };

const playerTwo = (new Player() as unknown) as FreezablePlayer;
playerTwo.shouldFreeze;
```

### Static Property Mixins [`#17829`](https://github.com/microsoft/TypeScript/issues/17829)

类表达式模式创建单例（只能扩展一个类），因此不能在类型系统中映射它们，去支持不同的变量类型。

你可以通过使用函数，返回基于泛型的不同类来解决这个问题:

```ts
function base<T>() {
  class Base {
    static prop: T;
  }
  return Base;
}

function derived<T>() {
  class Derived extends base<T>() {
    static anotherProp: T;
  }
  return Derived;
}

class Spec extends derived<string>() {}

Spec.prop; // string
Spec.anotherProp; // string
```

感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/mixins.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
