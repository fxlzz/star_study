# 设计思想
## 声明式与命令式
【编程范式】：
1、**命令式编程**：告诉计算机怎么做（How），我们需要给计算机指明每一个步骤
- 面向过程
- 面向对象

2、**声明式编程**：告诉计算机我要什么（What）
- 函数式编程
- _DSL_（领域专用语言，_HTML、CSS、SQL_）

_Hooks_ 的出现，还有更加重要的一个信号，那就是整个 _React_ 思想上面的转变，从“面向对象”的思想开始转为“函数式编程”的思想。这是编程范式上面的转变。

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
首先，有一个前提，如果两颗树完全对比，时间复杂度是 `O(n3)` ，时间耗时太大，React 通过三个假设优化为 `O(n)`

三个假设：
+ 只比较*同层级*的节点
+ 优先*比较 key 值*
+ 若*组件类型*不相同，那么直接销毁，不复用

在这个前提下，react diff 算法的流程，还分成了两种情况：
+ 单节点 diff
+ 多节点 diff

展开来说，*单节点 diff*——更新后只生成一个 react 元素
具体流程：
```txt
比较key --- key相同
			  |
			比较type 
			  | -- type 相同 -- 复用
			  | -- type 不同 -- 不复用
		--- key不同 
			  | 
			当前节点不复用，比较兄弟节点
```

*多节点 diff* 流程会稍微复杂一点，会分成两轮遍历
+ 第一轮遍历——尽可能遍历所有节点，判断是否能复用
+ 第二轮遍历——处理第一轮未被遍历到的节点

第一轮遍历的流程：
```txt
比较key --- key相同
			 |
		  比较type
		  	 | -- type 相同 -- 复用
		  	 | -- type 不同 -- 节点加入deletions数组，同一删除
		--- key不同
		     | 
		  结束遍历
```

第二轮遍历，分成三种情况：
+ 只剩下需*删除*的节点——原个数 > 更新后个数（加入 deletions 数组）
+ 只剩下需*新增*的节点——原个数 < 更新后个数
+ 新增和删除节点都存在（*移动*）——原个数 = 更新后个数（会将第一轮终止后的节点都加入 Map，第二轮遍历后续节点时，如果在 Map 中能找到，那么复用，找不到就新增；如果更新后的节点遍历完毕，Map 中还有剩余节点，那么加入到 deletions 数组）

### 为什么需要加 key？
因为在 diff 算法中，会优先判断 key 是否相同，来确定组件的复用性。
若 key 值不相等，react 能直接判断当前节点不能复用。
若不加 key，默认为 null，会走 key 相同的流程，会多判断一步 type。这样，判断时间会增加，复用效率会降低。

### 为什么不能用 index 作为 key？
因为使用 index 作为 key ，可能会影响复用效率。

比如，渲染数组，原数组为 `[0,1,2,3]` 更新成 `[2,3]`
走多节点 diff 流程，
1.  `0 == 0` key 相同，`0 !== 2` type 不同 ---> 0 加入 deletions
2. 同理，1 也加入 deletions 数组
3. 此时 `[2,3]` 遍历完成，*删除原数组所有元素*

若不用 index 作为 key 值，一样的场景
原数组为 `[0,1,2,3]` 更新成 `[2,3]`
走多节点 diff 流程，
1.  `0 !== 2` key 不同，结束遍历。将 `[0,1,2,3]` 加入到 Map 中
2.  `[2,3]` 可在 Map 中找到，*复用*
3. 遍历完后，`0,1` 删除

# 组件 & Hooks
类组件用的不多

## Hooks
### `useState`
渲染时机：
1. 在 DOM（wip fiber tree）更新后 
2. 执行*事件处理函数*（遇到 `setxxx`函数加入更新队列）
3. 触发*重新渲染*，期间执行更新队列，更新 state
4. 生成新的虚拟 DOM
5. *Diff* 比较，计算出最小差异
6. React 将计算好的差异应用到*真实 DOM*上（此时，浏览器尚未重绘）
7. 浏览器完成布局（layout）和绘制（paint）
8. 执行 `useEffect` 等*副作用*函数

有什么用：
宏观一点，更新 state，会更新组件

### `useEffect`
渲染时机： 在真实 DOM 生成后（commit 后），异步执行

会返回一个*清理函数*
*执行时机*： 在 7、8 之间执行
7. 浏览器完成布局（layout）和绘制（paint）
8. 执行 `useEffect` 清理函数
9. 执行 `useEffect` 等*副作用*函数

关于*依赖对象*
+ *不设置*依赖项 —— 组件每次重新渲染，`useEffect` 中代码重新执行
+ *具体*依赖项 —— 只有相关依赖项改变时， `useEffect` 才重新执行代码
+ *空数组* —— 只有组件初始时，执行 `useEffect` 代码（清理函数不执行）

```jsx
import { useState, useEffect } from "react";

const App = () => {

	const [count, setCount] = useState(0);

	useEffect(() => {
		if(count > 5) {
			console.log("定时器不在开启");
			return;
		}
	
		const timer = setInterval(() => {
			console.log("函数被调用了")
		}, 1000)
	
		// 每当用户点击一次，就会调一次清理函数
		// 但是，现象时定时器似乎没有停下来
		// 是因为，清理完上一个定时器后，马上又执行了 useEffect 函数，重新开启了一个定时器
		return () => {
			console.log("清理函数被调用了")
			clearInterval(timer);
		}
	}, [count])

	return (
		<>
			<p>{count}</p>
			<button onClick={() => setCount(count + 1)}>+1</button>
		</>
	);
}
```

### `useRef`
挂载在 fiber 上，只会更新一次

注意：
改变 ref 不会触发重新渲染, 这意味着 ref 是存储一些**不影响组件视图输出**信息的完美选择

```jsx
import { useState, useEffect, useRef } from "react";

const App = () => {

  const [count, setCount] = useState(0);
  const ref = useRef(0);

  const handleRef = () => {
    ref.current += 1;
    console.log(ref.current);
  };

  useEffect(() => {
    console.log("函数被调用了");

    return () => {
      console.log("清理函数被调用了");
    };
  }, [ref, count]);

  return (
    <>
      <p>count: {count}</p>
      <button onClick={() => setCount(count + 1)}>更新count</button>
      <button onClick={handleRef}>更新ref</button>
    </>
  );
};

export default App;
```

这段代码就能很好的反应出，三件事情：
+ `useEffect` 设置具体的依赖项，确实是依赖项改变后，执行 `useEffect` 中的代码
+ `useRef` 定义的变量，更新不会影响组件渲染
+ `useState` state 改变才会触发组件重新渲染

---
现象：
点击更新 ref，useEffect 代码不会执行
点击更新 count，视图更新，useEffect 代码执行

### `useMemo & useCallback`
优化措施：

 `useMemo` —— 缓存数值
 `useCallback` —— 缓存函数
UseMemo 和 useCallback 执行时机： 依赖项更新后，才会执行对应的代码

`React.memo` —— 缓存组件
props 和 context 发生改变时，才会重新渲染 `memo` 包裹的组件

