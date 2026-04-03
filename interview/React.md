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
首先，先明确一个概念，虚拟 DOM 是一个概念，而不是一项具体的技术。可以认为是普通 JS 对象对真实 DOM 的轻量级描述。
在 React 中，它就是 Fiber 节点树，react 16 之前是 `React.createElemet()` 函数调用后返回的结果

### 为什么使用虚拟 DOM？
两点：
+ 使用虚拟 DOM 保证了性能的下限，不会让开发者写出极其低效的 DOM 操作
+ 跨平台能力

*操作虚拟 DOM 不一定比操作真实 DOM 快：*
如果只更新一个按钮的文本，直接调用 API 更新一定比操作虚拟 DOM 快
因为虚拟 DOM 需要经过，创建对象 -> Diff 比较 -> 计算差异 -> 操作真实 DOM

但是，如果是*复杂的 UI 更新，频繁操作 DOM（浏览器会不停重排）*，那么虚拟 DOM 的价值就体现出来了——虚拟 DOM 会在内存中*收集所有的变更*，通过 Diff 算法计算出最小差异点，最后*一次性*合并到真实 DOM 上

本质上，是在尽可能的减少浏览器的重排（reflow）机制

### 为什么虚拟 DOM 可以跨平台？
这里涉及到 React 设计哲学，有个公式 `UI = f(state)`
这里的 `state` 可认为是虚拟 DOM——描述 UI 的数据，配合上 `f - function` ——不同平台操作 DOM 的 API
例如：
- 浏览器、`Node.js` 宿主环境使用 `ReactDOM` 包
- `iOS/Android` 宿主环境使用 `ReactNative` 包
- `Canvas、SVG` 宿主环境使用 `ReactArt` 包

## Diff 算法
如果两颗树完全对比，算法复杂度是 `O(n3)` ，React 通过三个假设将其优化到了 `O(n)`
- 只对比*同一层级*的节点，不跨层级对比
- 如果*组件类型*变了（比如 `div` 变 `p`），直接销毁重建，不再向下对比
- 通过向开发者提供的 `key`，React 能快速识别出哪些元素只是换了位置，而不需要销毁

# 组件 & Hooks
类组件用的不多

## Hooks
### `useState`

### `useEffect`

### `useRef`

### `useMemo & useCallback`


 
