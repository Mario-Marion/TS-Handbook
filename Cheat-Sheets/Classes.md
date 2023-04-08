---
highlight: vs2015
---
> 建议把 Handbook 部分（[官网链接](https://www.typescriptlang.org/docs/handbook/intro.html) 或 [掘金链接](https://juejin.cn/post/7207743145998925861)）全看完，再看本章节

# class

## 要点
TypeScript 对 ES2015 版本的 JavaScript 类，有一些特定类型的扩展，以及一个或两个运行时添加。

## 创建一个类实例
```js
class ABC { /**...*/ }
const abc = new ABC();
```
new ABC 的参数来自构造函数
## private 和 # 的区别

前缀 private 只是 TS 语法，在运行时不起作用，类外部能够访问（虽然类型检查器会报错）

```ts
class Bag { private item: any }
```

修饰符 # 是 JS 语法，是运行时私有的，并且在 JavaScript 引擎内部强制执行它只能在类内部访问

```ts
class Bag { #item: any }
```

修饰符 #，MDN 参考链接：<https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/Private_class_fields#%E8%AF%AD%E6%B3%95>

## 类中的 'this'
类中方法中的 this 依赖于方法是如何调用的。这可能和预期的不一致。可以使用绑定函数、箭头函数、this 参数，确保 this 类型。

this 参数是 TypeScript 语法，只是进行类型检查，编译后会被擦除，不影响运行时。
```ts
// TypeScript 输入带有 'this' 参数
function fn(this: SomeType, x: number) { /* ... */ }

// 编译后
function fn(x) { /* ... */ }
```
## 类型和值（Type and Value）

令人惊讶的是，类能够作为类型也能作为值。
```ts
const a: Bag = new Bag()
```
但最好不要这么做：
```ts
class C implements Bag { }
```

## 常用语法（common Syntax）

```ts
interface Updatable { }
type Serializzable = {}
class Account { }
// 继承 Account 类，实现 Updatable 接口 和 Serializzable 类型
class User extends Account implements Updatable, Serializzable {
  id: string; // 字段声明
  displayName?: boolean; //  可选属性
  name!: string; // 告诉类型检查器一定有 name 字段
  #attributes: Map<any, any>; // 私有属性，js语法
  roles = ["user"]; // 默认值
  readonly createdAt = new Date(); //只读属性和默认值
  private readonly createdAt = new Date(); // 只读私有属性和默认值
  constructor(id: string) {
    super()
    this.id = id
    this.#attributes = new Map()
  }

  setName(name: string) { this.name = name } // 原型方法
  verifyName = (name: string) => { this.name = name } // 实例方法，箭头函数

  // 函数重载
  sync(): Promise<{}>
  sync(cb: (result: string)=>void): void
  // 函数重载，最后要接着方法实现
  sync(cb?: (result: string)=>void): void | Promise<{}>{}
  // 属性描述符：Getters 和 Setters
  get accountID(){return 123}
  set accountID(value: number){ /** ... */ }
  
  private handleRequest(){} // 私有方法，ts 语法
  // 保护方法，只能在类和子类中使用，不可以在类外部访问
  protected handleReques(){}
  // 静态属性和方法
  static #userCount = 0 
  static registerUser(user: number){this.#userCount = user}
  // 静态块，this 引用静态类，可以获取静态私有属性，类初始化的时候执行
  static { this.#userCount = -1 }
}
```

## 泛型（Generics）
声明一个根据传入参数推断类型的字段。
```ts
class Box<Type>{
  constructor(public content:Type){}
}
const stringBox = new Box("a package")
// stringBox 类型：stringBox: Box<string>
```
## 参数属性
TypeScript 特定于类的一个扩展，自动将一个实例字段设置为输入参数。

```ts
class User {
  constructor(public name:string){}
}
```
`public` 可换成 `private` ，`public readonly`，`protected`， 等等......,但不能是 `static` 类型的属性和 js 语法私有属性 # 修饰符

这种写法等同于：
```ts
class User {
  name: string // 必须声明后才能在构造函数赋值
  constructor(name: string) {
    this.name = name
  }
}
```
## 抽象类（Abstract Classes）

在 TypeScript 中，类，方法和字段都可以是抽象的。抽象方法和字段都只能出现在抽象类中。

抽象类不能实例化，只能作为派生类的基类，派生类必须实现抽象类的所有抽象方法和字段。

```ts
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}
const b = new Base(); // 错误，无法创建抽象类的实例。

class Derived extends Base {
  // 必须实现该抽象方法
  getName() {
    return "world";
  }
}
const d = new Derived();
d.printName();
```

## 修饰器和属性（Decorators and Attributes）

在 "tsconfig" 中打开 "experimentalDecorators"

可以在类，方法，方法参数，属性和访问器上用修饰器

```ts
import { Syncable, triggerSync, preferCache, required } from "mylib"
@Syncable
class User {
  @triggerSync() // 函数修饰器
  save() { }
  @preferCache(false) // 访问器修饰器
  get displayName() { }
  update(@required info: Partial<User>) { } // 参数修饰器
}
```


感谢观看，如有错误，望指正

> 官网文档地址： <https://www.typescriptlang.org/static/TypeScript%20Classes-83cc6f8e42ba2002d5e2c04221fa78f9.png>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>


