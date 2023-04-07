---
highlight: vs2015
---
# Template Literal Types
模板字面量类型基于 [字符串字面量类型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types) 构建，并且能够通过联合类型扩展字符串。

它们具有与 JavaScript 中的 [模板字面量字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) 相同的语法，但用于表示类型。当与具体的字面量类型一起使用时，模板字面量类型会连接内容生成一个新的字符串字面量类型。
```ts
type World = "world";
type Greeting = `hello ${World}`;
// Greeting 类型：type Greeting = "hello world"
```
当在插值位置使用联合类型时，类型为每个联合类型成员所能表示的的字符串字量集合:
```ts
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// AllLocaleIDs 类型：
// type AllLocaleIDs = 
// "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```
对于模板字面量类型中的每一个插值位置，联合类型都交叉相乘：
```ts
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";

type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
// LocaleMessageIDs 类型：
type LocaleMessageIDs = "en_welcome_email_id" | "en_email_heading_id" | "en_footer_title_id" | "en_footer_sendoff_id" | "ja_welcome_email_id" | "ja_email_heading_id" | "ja_footer_title_id" | "ja_footer_sendoff_id" | "pt_welcome_email_id" | "pt_email_heading_id" | "pt_footer_title_id" | "pt_footer_sendoff_id"
```
我们通常建议人们使用提前生成的大型字符串联合类型，但这用于较小的情况。
## 在类型中的字符串联合
模板字面量的强大之处在于，根据类型内部的信息定义一个新字符串。

假设向函数 `makeWatchedObject` 传递对象，并在对象上添加了一个名为 `on()` 的新方法。在JavaScript 中，它的调用是：`makeWatchedObject(baseObject)`。我们可以把对象假设为：
```ts
const passedObject = {
    firstName: "Saoirse",
    lastName: "Ronan",
    age: 26,
};
```
在对象上添加 `on` 方法，并期望两个参数，一个 `eventName`（`string`）和一个 `callBack`（`function`）。

`eventName` 应该是 `attributeInThePassedObject + "Changed"` 形式，如：`firstNameChanged` 派生自对象中的 `firstName` 属性

当调用 `callBack` 函数时：

- 参数的类型应该与 `attributeInThePassedObject` 相关联；比如 `firstName` 为 `string`，因此 `firstNameChanged` 事件的回调在调用时期望传递一个 `string` 类型。类似地，与 `age` 相关的事件应该期望调用时传递 `number` 类型。
- `void` 返回类型（为了演示简单）

`on` 函数签名可能是这样的：
```ts
type PropEventSource<Type> = {
    on(eventName: string, callback: (newValue: any) => void): void;
};
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

const person = makeWatchedObject({
    firstName: "Saoirse",
    lastName: "Ronan",
    age: 26,
});

person.on("firstNameChanged", (newValue) => {
    console.log(`firstName was changed to ${newValue}!`);
});
```
但是，在前面的描述中，我们确定了在代码中的一些类型约束。所以，`on` 监听事件 `"firstNameChanged"`，我们要确保事件名称受到对象中属性名称的联合类型约束，并在末尾添加 "Changed"。换句话说，就是确保事件名称是 `attributeInThePassedObject + "Changed"` 形式。这样的话 `on()` 函数签名的规范就可以变得更加健壮。

模板字面量类型可以让我们将这些约束带入我们的代码中。虽然我们可以在 JavaScript 中做这样的计算，如： 
```ts
Object.keys(passedObject).map(x => `${x}Changed`)
```
，但是类型系统中的模板字面量类型提供了类似的字符串操作方法：
```ts
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```
有了这个模板字面量类型，我们在构建时传递错误的参数就会报错：
```ts
const person = makeWatchedObject({
    firstName: "Saoirse",
    lastName: "Ronan",
    age: 26
});
person.on("firstNameChanged", () => {});

// 防止简单的人为错误 (使用键而不是事件名称)
person.on("firstName", () => {});
// Error：
// 实参 "firstName" 类型不能赋值给行参 "firstNameChanged" | "lastNameChanged" | "ageChanged" 类型。

// 防止打字错误
person.on("frstNameChanged", () => {});
// Error：
// 实参 "frstNameChanged"' 类型不能赋值给行参 "firstNameChanged" | "lastNameChanged" | "ageChanged" 类型。
```
## 使用模板字面量的推断
注意，我们还必须确保回调函数的参数类型应该与 `attributeInThePassedObject` 相关联。就是，监听 `firstNameChanged` 事件，我们应该期望回调函数接收一个 `string` 类型的参数。类似的，监听 `ageChanged` 事件，回调函数应该接收一个 `number` 类型的参数。上面例子中，我们天真地在回调函数参数使用了 `any` 类型。应该使用模板字面量类型确保它们之间的关联。

实现这一点的关键：

1. 使用一个泛型函数
2. 第一个参数使用模板字面量类型，并在插值位置使用泛型变量属性的联合类型
3. 回调函数参数类型使用索引访问类型，在泛型变量中查找属性的类型

```ts
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>
    (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void ): void;
};
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
const person = makeWatchedObject({
    firstName: "Saoirse",
    lastName: "Ronan",
    age: 26
});
person.on("firstNameChanged", newName => {
    // newName 类型：(parameter) newName: string
    console.log(`new name is ${newName.toUpperCase()}`);
});
person.on("ageChanged", newAge => {
    // newAge 类型：(parameter) newAge: number
    if (newAge < 0) {
        console.warn("warning! negative age");
    }
})
```
这里我们把 `on` 变成了一个泛型方法。

当用户使用字符串 `"firstNameChanged"` 调用时，TypeScript 将尝试推断 `Key` 属性的正确类型。为此，它会将 `Key` 与 `"Changed"` 之前的内容进行匹配，并推断出 `Key` 为字符串 `"firstName"`。一旦 TypeScript 弄清楚了这一点，`on` 方法就可以获取原始对象 `firstName` 属性值的类型，在本例中为 `string`。同样，当使用 `"ageChanged"` 调用时，TypeScript 会找到属性 `age` 值的类型，即 `number`。

类型推断可以用不同的方式组合，通常是解构字符串，并以不同的方式重构它们。
## 内置字符串操作类型
为了帮助进行字符串操作，TypeScript 包含了一组可用于字符串操作的类型。为了提高性能，编译器内置了这些类型，并且不能在 TypeScript 附带的 `.d.ts` 文件中找到。
### `Uppercase<StringType>`
将字符串中的每个字符转换为大写版本。
```ts
type Greeting = "Hello, world"
type ShoutyGreeting = Uppercase<Greeting>
// ShoutyGreeting  类型：type ShoutyGreeting = "HELLO, WORLD"

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<"my_app">
// MainID 类型：type MainID = "ID-MY_APP"
```
### `Lowercase<StringType>`
将字符串中的每个字符转换为小写版本。
```ts
type Greeting = "Hello, world"
type QuietGreeting = Lowercase<Greeting>
// QuietGreeting类型：type QuietGreeting = "hello, world"

type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<"MY_APP">
// MainID 类型：type MainID = "id-my_app"
```
### `Capitalize<StringType>`
将字符串中的第一个字符转换为大写字母。
```ts
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
// Greeting 类型：type Greeting = "Hello, world"
```
### `Uncapitalize<StringType>`
将字符串中的第一个字符转换为小写字母。
```ts
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
// UncomfortableGreeting 类型：type UncomfortableGreeting = "hELLO WORLD"
```
## 内置字符串操作类型的技术细节
从 TypeScript 4.1开始，这些内置函数直接使用 JavaScript 字符串函数进行操作，并且不支持字符串区域设置。
 ```ts
function applyStringMapping(symbol: Symbol, str: string) {
    switch (intrinsicTypeKinds.get(symbol.escapedName as string)) {
        case IntrinsicTypeKind.Uppercase: return str.toUpperCase();
        case IntrinsicTypeKind.Lowercase: return str.toLowerCase();
        case IntrinsicTypeKind.Capitalize: return str.charAt(0).toUpperCase() + str.slice(1);
        case IntrinsicTypeKind.Uncapitalize: return str.charAt(0).toLowerCase() + str.slice(1);
    }
    return str;
}
```



> 官网文档地址： <https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html>
>
> 本章已上传 github： <https://github.com/Mario-Marion/TS-Handbook>
