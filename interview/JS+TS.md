# 闭包

闭包是什么？
在一个函数中访问了另一个函数作用域中的变量 —— 记住创建时作用域

实际场景中，怎么使用？
创建私有属性
```js
const test = () => {
	let cnt = 1;
	return {
		add() {
			return cnt++;
		},
		get() {
			return cnt;
		}
	}
}

const b = test();
b.add();
b.get();
```

闭包可能会导致什么问题？如何避免？

# 原型链

原型链是怎么形成的？
一切的起因来源于，JS 中的对象都有隐式原型，它指向创建它的构造函数的原型对象
原型对象在指向创建它的构造函数的原型对象
一直指向 object 原型对象的隐式原型 null 为至
[原型链](../front-tech/JavaScript/JS.md#原型链)

属性或方法查找顺序？
+ 如果对象的隐式原型上存在 —— 使用
+ 如果不存在 —— 沿着隐式原型往上招，直到 null 为至

# 事件循环
经典永流传
事件循环主要发生在浏览器的渲染主线程上，参与流程的，主要有三大部分：
+ 执行栈 —— 执行全局同步代码
+ 任务队列
	+ 宏任务队列： setTimeout、setInterval、io 操作、script 标签
	+ 微任务队列： promise、MutationObserver、queueMicrotask、process.nextTick
+ 其他线程 —— 计时线程、网络线程等

执行顺序：
执行栈执行完同步代码后，清空微任务队列，去红任务队列中取一个任务到执行栈中执行。任务队列中的任务来自于其他线程的任务，也可能来自当前任务的分任务

比如，执行栈执行到定时器代码。执行栈发现这是一个计时任务，交给计时线程（开始计时）。同时，执行栈继续执行后续同步代码。这是并行执行的，计时不会影响执行栈执行

# 异步编程
经典永流传+1

## Promise
三个状态？状态转移？
Pending、fulfilled、rejected

Pending -> fulfilled ： promise 任务完成
Pending -> rejected ： promise 任务错误

静态方法：
[Promise 静态方法](../front-tech/JavaScript/ES6+.md#Promise%20静态方法)

## Async & await
主要解决了调式调用的问题

Async 要求必须返回一个 promise 对象
Await 等待一个 promise 对象

# 作用域
作用域的划分？
+ 全局作用域
+ 函数作用域
+ 块级作用域

JS 中怎么根据作用域查找变量？
若当前作用域不存在待查找变量，会往上层作用域中，若还未找到，在往上级作用域中查找，直到找到 null 为止

# ESM & CommonJS
区别？

| CommonJS        | ESM          | 区别                |
| --------------- | ------------ | ----------------- |
| 动态              | 静态           | 导入方式              |
| Require         | Import       | 导入形式              |
| Module. Exports | Undefined    | This 指向           |
| 浅拷贝             | Live binding | 导出值机制             |
| 否               | 是            | 是否支持 TreeSharking |

为什么 ESM 一定要是静态导入？如果变成动态会有什么问题？

# 浅拷贝 & 深拷贝
区别： 更新后是否会影响到原数据
+ 不影响 —— 深拷贝
+ 影响 —— 浅拷贝

怎么实现？
浅拷贝
+ 引用类型： 扩展运算符、slice、concat、object.assgin

深拷贝：
+ 基本类型： 直接修改值
+ 引用类型： 数组、对象 —— JSON 序列化、loash、deepclone（手写）

