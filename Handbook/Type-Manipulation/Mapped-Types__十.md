# Mapped Types
有时一种类型需要基于另一种类型，但你不想重复写的时候，就可使用映射类型

映射类型建立于 [索引签名](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures) 语法之上：
```ts
type Dog = { name: string }
type OnlyBoolsAndHorses = {
    [key: string]: boolean | Dog;
};
const conforms: OnlyBoolsAndHorses = {
    del: true,
    d: { name: 'Snoopy' },
};
```
映射类型为泛型，可以使用 `PropertyKeys`（通常通过 [keyof 操作符](https://juejin.cn/post/7207342547361447995) 创建）的联合来遍历键，以创建类型：
```ts
type OptionsFlags<Type> = {
    [Property in keyof Type]: boolean;
};
```
在例子中，`OptionsFlags` 将获取泛型参数类型 `Type` 的所有属性，并将属性的值改为 `boolean` 类型。
```ts
type FeatureFlags = {
    darkMode: () => void;
    newUserProfile: () => void;
};
type FeatureOptions = OptionsFlags<FeatureFlags>;
// FeatureOptions 类型为
// type FeatureOptions = { 
//     darkMode: boolean;
//     newUserProfile: boolean;
// }
```
> keyof Type 会产生一个 Type 类型的键的联合类型，in 类似于 JS 中的 for...in，会遍历这个联合类型，Property 就是联合类型的成员
## 映射修饰符
`-`，`+` 这两个附加符号能在映射期间应用 `readonly` 和 `?` 修饰符，分别影响属性可变性和可选性。

通过添加前缀 `-` 或 `+`，移除或添加修饰符。如果你没有使用前缀，默认为 `+`。
```ts
type LockedAccount = {
    readonly id: string;
    readonly name: string;
};
// 移除泛型参数类型 type 属性中的 'readonly' 修饰符
type CreateMutable<Type> = {
    -readonly [Property in keyof Type]: Type[Property];
};
type UnlockedAccount = CreateMutable<LockedAccount>;
// UnlockedAccount 类型为
// type UnlockedAccount = {
//     id: string;
//     name: string;
// }
```
```ts
type MaybeUser = {
    id: string;
    name?: string;
    age?: number;
};
// 移除泛型参数类型 type 属性中的 '?' 修饰符
type Concrete<Type> = {
    [Property in keyof Type]-?: Type[Property];
};
type User = Concrete<MaybeUser>;
// User 类型为
// type User = {
//     id: string;
//     name: string;
//     age: number;
// }
```
```ts
type MaybeUser = {
    id: string;
    name?: string;
    age?: number;
};
// 添加泛型参数类型 type 属性中的 readonly 修饰符
type Concrete2<Type> = {
    readony [Property in keyof Type]: Type[Property];
    // 或
    // +readony [Property in keyof Type]: Type[Property];
};
type User2 = Concrete2<MaybeUser>;
// User 类型为
// type User2 = {
//     readonly id: string;
//     readonly name?: string;
//     readonly age?: number;
// }
```
## 通过 `as` 重映射键
在 TypeScript 4.1 及以后版本中，你可以在映射类型中使用 `as` 从句，重新映射映射类型中的键名:
```ts
type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```
你可以利用像 [模板字面量类型](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) 这样的特性来从先前的属性名中创建新的属性名：
```ts
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};
interface Person {
    name: string;
    age: number;
    location: string;
}
type LazyPerson = Getters<Person>;
//  LazyPerson 类型：
//  type LazyPerson = {
//    getName: () => string;
//    getAge: () => number;
//    getLocation: () => string;
//  }
```
你可以通过条件类型生成 `never` 来过滤掉属性：
```ts
// Remove the 'kind' property
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};

interface Circle {
    kind: "circle";
    radius: number;
}

type KindlessCircle = RemoveKindField<Circle>;
//   type KindlessCircle = { radius: number; }
```
你可以映射任意的联合类型，不只是 `string | number | symbol`：
```ts
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E["kind"]]: (event: E) => void;
}
type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };
type Config = EventConfig<SquareEvent | CircleEvent>
//  Config 类型：
//  type Config = {
//    square: (event: SquareEvent) => void;
//    circle: (event: CircleEvent) => void;
//  }
```
## 进一步探索
映射类型能与手册中类型操作部分，里面的其他特性进行结合使用。例如，下面是一个使用 [条件类型](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html) 的映射类型，它根据对象是否有值为 `true` 的 `pii` 属性，返回 `true` 或 `false`：
```ts
type ExtractPII<Type> = {
    [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};
type DBFields = {
    id: { format: "incrementing" };
    name: { type: string; pii: true };
};
type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;
// ObjectsNeedingGDPRDeletion 类型：
// type ObjectsNeedingGDPRDeletion = {
//   id: false;
//   name: true;
// }
```

感谢观看，如有错误，望指正


> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/2/mapped-types.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>

