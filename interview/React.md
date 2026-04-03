# 设计思想
## 声明式与命令式
## 组件复用设计
## 怎么设计数据流？

# JSX & 渲染机制
## JSX 本质
>  问： JSX 本质是什么？
>  答： JSX 的本质是 `React.createElement()` 的语法糖，允许开发者使用类 HTML 的语法，描述 UI。会被 Babel 转化为 JS 函数调用。函数执行完后，会返回一个 JS 对象，也就是 React 元素。React 拿到后会构建虚拟 DOM，从而实现高效更新和跨平台渲染

首先明白，JSX 不是字符串，也不是 HTML，是 JavaScript 的扩展语法
其次，JSX 是 `React.createElement()` 的语法糖

```jsx
const element = <div className="title">hello world</div>
```

经过 babel 或 swc 转化工具，转化成下面的样子
```js
// 编译后的样子
const element = React.createElement(
  'div', // type 
  { className: 'title' }, // props
  'hello world' // children
);
```

`React.createElement` 接收三个参数：
1. *type (类型)*：可以是标签名字符串（如 `'div'`），也可以是 React 组件（函数或类）
2. *props (属性)*：一个包含所有属性的对象（如 `{ className: 'title' }`）
3. *children (子元素)*：可以是一个或多个参数，甚至是另一个 `createElement` 的调用

---
当执行完 `React.createElement()` 会返回一个 JS 对象，也就是*React 对象*
```js
{
	type: "div",
	props: {
		className: "title",
		children: "hello world" // 子元素也放在 props 中
	},
	key: null,
	ref: null,
	$$typeof: Symbol.for('react.element') // 这是一个安全标识符
}
```

---
*为什么会有 `$$typeof` 这个属性呢？*
+ 这是一个专门用来防止 *XSS（跨站脚本攻击）* 的机制
+ 如果你的服务器漏洞导致返回了恶意的 JSON 模拟 React 元素，因为 JSON 不支持 `Symbol` 类型，React 会在渲染时检测到 `$$typeof` 缺失或错误，从而拒绝渲染该对象

*宏观渲染机制*
1. 用户编写 JSX 描述 UI
2. Babel 或 swc 转化为 `React.createElement()`
3. 执行后，转化为 React 元素 --- 调度器
4. 执行协调器流程，diff 算法比对新旧树的差异，交给渲染器 --- 协调器
5. ReactDOM 负责这些差异最小化的更新到真实 DOM 上 --- 渲染器

## JSX 和模板引擎的区别
>  问： JSX 和模板引擎有什么区别？
>  答： 
>  - JSX 本质上是 JS 语法的扩展，被编译函数调用，用 JS 对象来描述 UI；而模板引擎本质上是字符串模板，用数据填充后生成 HTML
>  - JSX 具备完成的 JS 能力，而模板引擎会受限于 DSL

*运行时机不同*：
+ JSX
	+ 编译 -- JS 函数
	+ 执行 -- 虚拟 DOM
	+ 更新 -- diff 算法
 特点： 不是字符串拼接，是 JS 对象

 + 模板引擎
	+ 编译 -- 函数
	+ 执行 -- 返回 HTML 字符串
	+ 更新 -- 重新渲染整段
 特点： 本质还是字符串

同时，因为 JSX 类似于 JS，没有丢失语言便捷，可用 if/for/map 等方法。而，模板引擎受限于 DSL，DSL 中是怎么描述的，模板中就只能怎么写（灵活度下降）

*设计哲学不同*：
JSX：`UI = f(state)`
```jsx
function App({ list }) {
  return list.map(item => <div>{item.name}</div>);
}
```
特点：
- UI 是函数
- 数据驱动
- 强组合能力

模板引擎： HTML + 数据填充
```html
<div>{{name}}</div>
```
特点：
- 更像“渲染模板”

## 虚拟 DOM 是什么？
首先，先明确一个概念，虚拟 DOM 是一个概念，而不是一项具体的技术。可以狭义的认为是一段特殊的*数据结构*
在 React 中，`React.createElemet()` 函数调用后的返回的 JS 对象就是虚拟 DOM

### 为什么虚拟 DOM 快？
按之前的理解，虚拟 DOM 可被认为是一段 JS 对象，这段 JS 对象描述了页面 UI。因此，对 DOM 的操作转化成为了对 JS 对象的操作（并且这个是在内存中进行的）。

结合浏览器渲染原理，只有在更新 DOM 时，才会触发浏览器的重绘机制。

因此，虚拟 DOM 快。

### 为什么虚拟 DOM 可以跨平台？
这里涉及到 React 设计哲学，有个公式 `UI = f(state)`
这里的 `state` 可认为是虚拟 DOM——描述 UI 的数据，配合上 `f - function` ——不同平台操作 DOM 的 API
例如：
- 浏览器、`Node.js` 宿主环境使用 `ReactDOM` 包
- `Native` 宿主环境使用 `ReactNative` 包
- `Canvas、SVG` 宿主环境使用 `ReactArt` 包
- `ReactTest` 包用于渲染出 JS 对象，可以很方便地测试“不隶属于任何宿主环境的通用功能”

## Diff 算法

# 组件 & Hooks
类组件用的不多

## Hooks
### `useState`

### `useEffect`

### `useRef`

### `useMemo & useCallback`


 
