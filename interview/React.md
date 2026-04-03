# React 设计思想
## 声明式与命令式
## 组件化思维

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
为什么会有 `$$typeof` 这个属性呢？
+ 这是一个专门用来防止 *XSS（跨站脚本攻击）* 的机制
+ 如果你的服务器漏洞导致返回了恶意的 JSON 模拟 React 元素，因为 JSON 不支持 `Symbol` 类型，React 会在渲染时检测到 `$$typeof` 缺失或错误，从而拒绝渲染该对象

宏观渲染机制：
1. 用户编写 JSX 描述 UI
2. Babel 或 swc 转化为 `React.createElement()`
3. 执行后，转化为 React 元素 --- 调度器
4. 执行协调器流程，diff 算法比对新旧树的差异，交给渲染器 --- 协调器
5. ReactDOM 负责这些差异最小化的更新到真实 DOM 上 --- 渲染器

## JSX 和模板引擎的区别
 

