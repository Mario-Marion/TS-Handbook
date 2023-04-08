# DOM Manipulation
### 对 ***HTMLElement*** 类型的探索

在标准化20年后，JavaScript 已经走过了很长的一段路。虽然 2020 年，JavaScript 可以在使用在服务端、数据科学甚至物联网设备，但最主要用在 web浏览器。

网站是由 HTML 和/或 XML 文档组成的。这些文档是静态的，不会改变。***Document Object Model(DOM)*** 是浏览器实现的编程接口，目的是使静态网站有功能性。DOM API 能改变文档结构、样式、内容。API 功能非常强大，无数前端框架(jQurty,React,Angular 等)都是围绕着它开发的，目的是制作动态网站，甚至更容易开发。

TypeScript 是 JavaScript 的类型化超集，并为 DOM API 定义了类型。这些 DOM 类型在任何默认的 TypeScript 项目都能轻易获得。在 lib.dom.d.ts 定义了20,000 行，其中 `HTMLElement` 类型是比较突出的，这个类型是 TypeSctipt 操作 DOM 的主干类型。

源码地址 [lib.dom.d.ts](https://github.com/microsoft/TypeScript/blob/main/lib/lib.dom.d.ts)

### 基础案例
一个简化的 *index.html* 文件
```html
<!DOCTYPE html>
<html lang="en">
  <head><title>TypeScript Dom Manipulation</title></head>
  <body>
    <div id="app"></div>
    <!-- 假设 index.js 是 index.ts 编译输出的-->
    <script src="index.js"></script>
  </body>
</html>
```
用 TypeScript 添加 `<p>Hello, World</p>` 元素到 `#app` 元素。
```ts
// 1. 使用 id 属性选中某 div 元素
const app = document.getElementById("app");
// 2. 用编程方式创建新的 <p></p> 元素
const p = document.createElement("p");
// 3. 为新的 p 元素添加文本内容
p.textContent = "Hello, World!";
// 4. 把新的 p 元素附加到一开始获取的 div 元素
app?.appendChild(p);
```
编译ts和运行 *index.html* 页面，结果 HTML 为：
```html
<div id="app">
  <p>Hello, World!</p>
</div>
```
### `Document` 接口
上例中，TypeScript 第一行代码使用全局变量 `document`。检查该变量可以发现它是由 `lib.dom.d.ts` 文件中的 `Document` 接口定义的。而且包含对两个方法的调用，`getElementById` 和 `createElement`。

#### `Document.getElementById`
该方法定义如下:
```ts
getElementById(elementId: string): HTMLElement | null;
```
传递一个元素 id 字符串，并且返回 `HTMLElement` 或 `null` 其中一个。这个方法引用了最重要的类型之一 `HTMLElement`，是所有其他元素接口的基本接口。例如，`getElementById` 定义返回的类型是 `HTMLElement`，可是基础案例中，变量 `p` 的实际类型为 `HTMLParagraphElement`（说明 `HTMLParagraphElement` 是 `HTMLElemnt` 的子类型 或 实现了 `HTMLElement`，具体可参考我另一篇文章中的 [类之间的关系](https://juejin.cn/post/7202898170613907517#heading-40)）。同时注意这方法可以返回 `null`，这是因为该方法在运行前不能确定它是否能够真正找到指定的元素。因为有可能为 `null` ，所以基础案例中最后一行代码使用了新运算符 `可选链` 去调用 `appendChild` 方法。
#### `Document.createElement`
这个方法的定义如下(省略了已弃用的定义，可到源码查看完整定义):
```ts
createElement<K extends keyof HTMLElementTagNameMap>(tagName: K, options?: ElementCreationOptions): HTMLElementTagNameMap[K];
createElement(tagName: string, options?: ElementCreationOptions): HTMLElement;
```
这是一个重载函数，第二个重载是最简单的，它的工作原理与 `getElementById` 方法非常相似。传递任意字符串并返回标准 `HTMLElement` 类型。这个定义使开发人员能够创建唯一的 HTML 元素标记。

例如 `document.createElement('xyz')` 返回 `<xyz></xyz>` 元素，显然不是 HTML 规范指定的元素。

`如果你有兴趣，可以使用 document.getElementsByTagName 获取自定义元素`

`createElement` 的第一个定义，使用了泛型模式。分解成几个部分更好理解，先从泛型表达式开始：`<K extends keyof HTMLElementTagNameMap>`。表达式定义了泛型参数 `K` ，它被约束为接口 `HTMLElementTagElement` 的键。`HTMLElementTagElement` 映射接口包含每个指定的 HTML 标签名及其对应的类型接口。例如，下面是前5个映射值:
```ts
interface HTMLElementTagNameMap {
  "a": HTMLAnchorElement;
  "abbr": HTMLElement;
  "address": HTMLElement;
  "applet": HTMLAppletElement;
  "area": HTMLAreaElement;
      ...
}
```
有些元素没有独特的属性，所以它们只返回 `HTMLElement`，拥有独特属性和方法的类型返回它们特定的接口（将继承 `HTMLElement` 或 实现 `HTMLElement`）

现在再看其它部分：`(tagName: K, options?: ElementCreationOptions): HTMLElementTagNameMap[K]`。第一个参数 `tagName` 被定义为泛型参数 `K` 。TypeScript 解释器可以从该参数的实参推断出泛型参数 `K` 类型。这意味着开发人员在使用这方法时不必指定泛型参数；并且可以在定义的其余部分中使用，如：返回值 `HTMLElementTagNameMap[K]` 使用 `tagName` 参数返回相应的类型。在基础案例中变量 `p` 调用该方法获得 `HTMLParagraphElement` 类型。如果代码是 `document.createElement('a')`，那么将获得 `HTMLAnchorElement` 类型。

### `Node` 接口
`document.getElementById` 方法返回 `HTMLElement`。`HTMLElement` 接口继承了 `Element` 接口，`Element` 接口继承了 `Node` 接口。这使得所有 HTML 元素可以使用 `Element` 与 `Node` 的方法。在基础案例中，我们使用一个定义在 `Node` 接口上的 `appendChild` 属性，把新的 `p` 元素附加到到网页。 

#### `Node.appendChild`
现在再看基础案例中最后一行 `app?.appendChild(p)`。在上面 `document.getElementById` 部分已经讲解了 `可选链` 运算符使用在 `app` 元素，是因为 `app` 运行时可能为 null。而 `appendChild` 方法的定义为：
```ts
appendChild<T extends Node>(newChild: T): T;
```
这方法工作原理和 `createElement` 方法类似，泛型参数 `T` 来自对参数 `newChild` 实参的推断，并且 `T` 受到 `Node` 接口约束。
### `NodeList` 接口 与 `NodeListOf` 接口
```ts
interface NodeList {
    readonly length: number;
    item(index: number): Node | null;
    forEach(callbackfn: (value: Node, key: number, parent: NodeList) => void, thisArg?: any): void;
    [index: number]: Node;
}
interface NodeListOf<TNode extends Node> extends NodeList {
    item(index: number): TNode;
    forEach(callbackfn: (value: TNode, key: number, parent: NodeListOf<TNode>) => void, thisArg?: any): void;
    [index: number]: TNode;
}
```
`NodeList` 接口只实现了以下属性和方法：`length`，`item(index)`，`forEach((value, key, parent) => void)`，和数值索引。而 `NodeListOf` 接口只是扩展了 `NodeList` 接口（可参考我另一篇文章：[接口扩展](https://juejin.cn/post/7202576202288414778#heading-7)），并且接收一个泛型 `TNode`，该泛型受到 `Node` 接口约束，并且把 `NodeList` 列表的元素 `Node` 类型，改为泛型 `TNode` 类型，类似于 `Array<T>` ，开发者传入指定泛型，就返回指定泛型的集合。
### `children` 和 `childNodes` 的区别
在 DOM API 中，有子元素的概念。例如在以下 HTML 中，`p` 标签是 `div` 元素的子元素。
```ts
<div>
  <p>Hello, World</p>
  <p>TypeScript!</p>
</div>;
const div = document.getElementsByTagName("div")[0];
div.children;
// HTMLCollection(2) [p, p]
div.childNodes;
// NodeList(2) [p, p]
```
捕获 `div` 元素后，`children` 属性将返回包含两个 `HTMLParagraphElement` (p 元素类型) 的 `HTMLCollection` 列表。而 `childNodes` 属性将返回 `NodeList` 列表，也是包含两个 `HTMLParagraphElement`，但是 `NodeList` 可以包含额外的 HTML 节点，`HTMLCollection` 列表不能。

例如，删除一个 `p` 标签，但是保留它的文本。

```ts
<div>
  <p>Hello, World</p>
  TypeScript!
</div>;
const div = document.getElementsByTagName("div")[0];
div.children;
// HTMLCollection(1) [p]
div.childNodes;
// NodeList(2) [p, text]
```
`children` 现在只包含 `<p>Hello, World</p>` 元素，而 `childNodes` 比 `children` 多了一个 `text` 节点。`text` 是文字节点，包含文本 "TypeScript!"。`children` 列表没有包含这个文本节点，因为它是不 `HTMLElement` 类型。
### `querySelector` 和 `querySelectorAll` 方法
这两个方法是获取**符合独特约束条件**的 dom 元素列表的工具。它们在 *lib.dom.d.ts* 中被定义为:
```ts
/**
 * Returns the first element that is a descendant of node that matches selectors.
 */
querySelector<K extends keyof HTMLElementTagNameMap>(selectors: K): HTMLElementTagNameMap[K] | null;
querySelector<K extends keyof SVGElementTagNameMap>(selectors: K): SVGElementTagNameMap[K] | null;
querySelector<E extends Element = Element>(selectors: string): E | null;
/**
 * Returns all element descendants of node that match selectors.
 */
querySelectorAll<K extends keyof HTMLElementTagNameMap>(selectors: K): NodeListOf<HTMLElementTagNameMap[K]>;
querySelectorAll<K extends keyof SVGElementTagNameMap>(selectors: K): NodeListOf<SVGElementTagNameMap[K]>;
querySelectorAll<E extends Element = Element>(selectors: string): NodeListOf<E>;
```
`querySelectorAll` 的定义类似 `getElementsById`，只是它返回一个新类型: `NodeListOf`。这个返回类型本质上是标准 JavaScript 列表元素的自定义实现。可以说，用 `Array<E>` 替换 `NodeListOf<E>` 会产生非常相似的用户体验。`NodeListOf` 只实现了以下属性和方法：`length`，`item(index)`，`forEach((value, key, parent) => void)`，和数值索引。另外，这个方法返回元素列表，不是节点列表。
```ts
<ul>
  <li>First :)</li>
  <li>Second!</li>
  <li>Third times a charm.</li>
</ul>;
const first = document.querySelector("li"); // returns the first li element
const all = document.querySelectorAll("li"); // returns the list of all li elements
```
泛型表达式 `querySelectorAll<E extends Element = Element>`，表示如果指定了泛型 `E` 的类型，并且是 `Element` 的子类型，那么 `E` 就是你指定的类型。如果没传指定泛型，那么泛型 `E` 默认就是 `Element` 类型。（注意！默认值也要受到 `Element` 接口的约束。）

例子：
```ts
type QuerySelectorAll<E extends number = 123> = (selectors: string) => Array<E>;
// 指定泛型
const Q1: QuerySelectorAll<456> = (str) => {
  return [456]
}
// 无指定泛型
const Q2: QuerySelectorAll= (str) => {
  return [123]
}
```
例子中，`Q1` 指定了泛型 `E` 为 `456`，`456` 为 `number` 的子类型，符合约束，返回值为数组，元素为 `456`。`Q2` 没指定泛型，所以 `E` 为 `123`，因为默认值是 `123`，返回值为数组，元素为 `123`。默认值也要受到 `number` 的约束，例子中默认值为 `123`，是 `number` 的子类型，所以正确。例子中如果默认值为 `string` 或其它类型是不行的。



感谢观看，欢迎互相讨论与指导，以下是参考资料链接🔗

<https://www.typescriptlang.org/docs/handbook/dom-manipulation.html>
