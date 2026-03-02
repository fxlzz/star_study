
# 一、入门篇

### 快速开始：

```bash
npx create-react-app my-app
cd my-app
npm start
```

当然用 _vite_ 也可以创建 _react_

```bash
yarn vite
```

## 初识

React 是一个很灵活的 JavaScript 库，

分成两个库：

- React -> 核心库（任何环境都可以运行，无论是浏览器还是 node）
- React-dom -> 提供一个 API 在浏览器环境下运行

【React 元素】

React 底层使用`React.createElement("容器", {属性}, "内容");`来创建 React 元素，是 React 核心库提供的功能。例如： `const h1 = React.createElement("h1", {}, "hello world");`这个是创建虚拟 dom 元素，是 React 元素。

创建完 React 元素后，会交给`ReactDOM.render()`创建成 dom 元素，

```jsx
import React from "react";
import { createRoot } from "react-dom/client";

createRoot(document.getElementById("root")).render(
  <h1>
    Hello World <span>span元素</span>
  </h1>
);
```

像我们自己定义的组件，一般称为： React Component Element 比如：`<Comp />`

react 中内置的组件，一般为：React Html Element 比如：`<h1></h1>`

## JSX语法

其实也不是什么新的东西，就是长的很像 _HTML_ ，但是不是的！！

是_JS_语言的语法扩展，_React_ 是_JavaScript_库，虽然是官网上这么说的

与 _Vue_ 的区别：

- _Vue_ 使用来描述页面
- _React_ 使用JSX来描述页面

【注意】：一些语法规则

使用 _JSX_ 来描述页面时，有如下的一些语法规则：

- 根元素只能有一个，单标签需要闭合
- _JSX_ 中使用 _JavaScript_ 表达式。大括号 _{}_ 可以将_JavaScript_中的逻辑和变量带入到标签
- {{ 和 }} 不是特殊语法：他只是包在JSX大括号内的_JavaScript_对象
- _style_ 对应样式对象，记住 _style_ 对象里面的属性要写成 驼峰命名法，_class_ 要写作 _className_
- _JSX_ 注释需要写在花括号
- _JSX_ 允许在模板中插入数组，数组会自动展开所有成员

## React组件

在我的理解力，应该是又两种写法，一种是 _类组件_ ，一种是 _函数组件_ ，现在用的比较多的是 _函数组件_

**类组件**

```jsx
class Test extends React.Component{
  render(
    return {

    }
  )
}
```

**函数组件 【现在一般用这个】**

```jsx
function Test{ // 这里必须大写
  return (

  );
}
```

【注意】

1、函数名必须大写

2、_return_ 要用 _()_ 包起来，不然下行代码会被忽视

3、一个组件中可以声明多个组件，但是不能嵌套声明（而且需要注意的是，在顶层声明其他的组件）

### 导入和导出

跟_JavaScript_中的导入和导出差不多，可以用 _具名_ 和 _默认_ 导入和导出，要注意的是，**一个组件中，只能有一个默认导出，多个具名导出**

|   |   |   |
|---|---|---|
|语法|导出语句|导入语句|
|默认|`export default function Button() {}`|`import Button from './Button.js';`|
|具名|`export function Button() {}`|`import { Button } from './Button.js';`|

### 熟悉的 Props

跟 _Vue_ 差不多，也是来传递参数的，就是写法有点不一样而已

_React_ **组件函数**接收一个参数，一个 _props_ 对象，_props_ **是组件的唯一参数**

通常不需要整个 `props` 对象，可以将它**解构**为单独的 _props_

在声明 props 时， **不要忘记** `(` **和** `)` **之间的一对花括号** `{` **和** `}` ，从对象中 _解构_

```jsx
function Avatar({ person, size }) {
  // ...
}
```

#### 设置默认值

```jsx
function Avatar({ person, size = 100 }) {
  // ...
}
```

默认值仅在缺少 `size` prop 或 `size={undefined}` 时生效。

但是如果你传递了 `size={null}` 或 `size={0}`，默认值将 **不** 被使用。

#### 类型检测

#### children -> 插槽

_props_ 中的 _children_ 的功能就类似于 _Vue_ 中的 _插槽_ 的功能，可以接收**父组件使用任意 JSX **

例如：

```jsx
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children} // 类似于 Vue <slot></slot> 的写法
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
        />
    </Card>
  );
}
```

### 条件渲染

在 _React_ 中，可以通过使用 _JavaScript_ 的 _if_ 语句、_&&_ 和 _? :_ 运算符来选择性的渲染 _JSX_

**返回值为** **_null_** ，那么就不会渲染满足条件的_组件_。

在 JSX 里，React 会将 `false` 视为一个“空值”，就像 `null` 或者 `undefined`，React 不会在这里进行任何渲染

**切勿将数字放在** `&&` **左侧**

_JavaScript_ 逻辑运算符判断：如果 `null && 123` 前面判断为 _false_ 返回前面的值 _null_ ，为真值时，返回后面的值 _123_

比如：`0 && 123` _JavaScript_ 中在进行 _&&_ 运算时，会进行自动的布尔判定，0 判定为 false，所以这个表达式的返回值为 0

例如：

```jsx
一、if - else语句
function Item({name, isPacked}) {
  if (isPacked) {
    return <li>{name + '√'}</li>
  } 
  return <li>{name}</li>
}

二、? : 三元运算符
function Item({name, isPacked}) {
  return <li>{isPacked ? name + '√' : name}</li>
}

三、&& 
function Item({name, isPacked}) {
  return <li>{name}{isPacked && '√'}</li>
}

四、额外增加一个存储空间
function Item({name, isPacked}) {
  let content = name;
  if (isPacked) {
    content = name + '√'
    或者
    content = (
      <del>
        {name + '√'}
      </del>
    ) 
    // 这是一段 JSX 代码，在 JSX 代码中使用JS代码需要用{}
    // 不要感觉到很奇怪！
  }
  return <li>{content}</li>; 
}
```

### 渲染列表

前提：_JSX_ 可以直接渲染 **_数组_**

这个跟 _Vue_ 不一样的是，没有 _v-for_ 的指令，其实 _React_ 更加接近元素 _JavaScript_ ，在渲染列表中，会大量使用 _map_ _filter_ 这写函数

比如：

```jsx
function Test() {
  const list = [
    {id: 1, name: '张三', age: 17},
    {id: 2, name: '李四', age: 18},
    {id: 3, name: '王五', age: 19}
  ];
  const lists = list.map(item => <li key={item.id}>姓名：{item.name}，年龄：{item.age}</li>)
  return (
    <ul>{lits}</ul>
  );
}
```

## 添加交互

### 响应事件

如何添加事件呢？在 _React_ 中，使用的方式更加贴近原生 _JavaScript_ 的操作方式。

事件处理函数，在 DOM（wip fiber tree） 生成完成后执行

**事件处理函数**：

- 通常定义在组件**内部**
- 名称以 _handle_ 开头，后跟事件名称

例如：

```jsx
function Test() {
  function handleClick() {
    console.log('触发了点击事件')
  }
  return (
    <button onClick={handleClick}>点击</button>
  );
}

或者
  <button onclick={() => {
  console.log('点击触发了')
}}>点击</button>
```

**特别注意**：

传递事件处理函数，一定是函数，在JSX中 `{}` 里面的都被看做成 _JS_ 代码。要传递函数，而不是调用函数

区别：

**第一个**会等点击事件之后，触发 _handleClick_ 回调函数

**第二个**是一串 _JS_ 代码，会直接调用_handleClick_ 函数

|   |   |
|---|---|
|传递一个函数（正确）|调用一个函数（错误）|
|`<button onClick={handleClick}>`<br><br>`<button onClick={() => handleClick()}>`|`<button onClick={handleClick()}>`|

### 事件冒泡

跟原生 _JavaScript_ 一样，_react_ 中的事件处理函数也会**冒泡**，阻止事件冒泡

只能使用 **事件对象**`e.stopPropagation()` 不能使用 `return false`的方式

### 阻止默认行为

`e.preventDefault()`

## state

相当于 _Vue_ 中的 _data_ ，用于保存数据，事件处理函数可以处理 _state_ 中的数据，不破坏从 _prop_ 中传入的数据，保持组件的 **纯粹性**

### 第一个 Hook -> useState

在 React 中，`useState` 以及任何其他以“`use`”开头的函数都被称为 **Hook**。

**Hooks ——以** `use` **开头的函数——只能在组件或**[自定义 Hook](https://react.docschina.org/learn/reusing-logic-with-custom-hooks) **的最顶层调用。** 不能在条件语句、循环语句或其他嵌套函数内调用 Hook

定义useState：

```jsx
// 定义了多个 state
const [index, setIndex] = useState(0);
const [showMore, setShowMore] = useState(false);
```

- `useState` Hook 返回一对值：当前 state 和更新它的函数。
- 你可以拥有多个 state 变量。在内部，React 按顺序匹配它们。

### state 是隔离且私有的

换句话说，**如果你渲染同一个组件两次，每个副本都会有完全隔离的 state**！改变其中一个不会影响另一个。

与 props 不同，**state 完全私有于声明它的组件**。父组件无法更改它。这使你可以向任何组件添加或删除 state，而不会影响其他组件。

@__@!!!

### 按下快门

_React_ 渲染组件：1、组件的 **初次渲染** 2、**state** 的修改（包括自身和祖先组件，设置 _state_ 不会更改现有渲染中的变量，会请求一次新的渲染）

在 _React_ 渲染过程中，_setValue_ 就像是一个请求者，只要看见这个东东，_React_ 就会马上就会 **“按下快门”** ，也就是当前状态下的 _state_ 值都会使用 **同一个值**

例如：

```jsx
import {useState} from 'react';
export default function Test (){
  const [num, setNum] = useState(0);
  function handleClick() {
    setNum(num + 1); 
    setNum(num + 1);
    setNum(num + 1);
  }
  return (
    <> 
      <p>{num}</p>
      <button onClick={handleClick}>+3</button>
    </>
  );
}
```

**注意**：

这里，本意是每次按下按钮 _num_ 的值就会 _+3_ ，但是实际上，每次点击按钮只会 _+1_ ，这是因为 _React_ 看到第一个 _setNum(num + 1)_ 马上就 “按下快门”，因此，**上面的代码**就变成了**下面的代码**

```jsx
function handleClick() {
  setNum(0 + 1);
  setNum(0 + 1);
  setNum(0 + 1);
}
```

### 如何解决呢？

前提知识：**React 会等到事件处理函数中的** 所有 **代码都运行完毕再处理你的 state 更新（异步的，批处理）。** 这就是为什么重新渲染只会发生在所有这些 `setNumber()` 调用 **之后** 的原因。

替换成 **更新函数**，一次多个设置 _state_ 值，上述代码，可以修改成下面的代码

```jsx
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

**解释：**

1. React 会将此**函数**加入队列，以便在事件处理函数中的所有其他代码运行后进行处理。
2. 在下一次渲染期间，React 会遍历队列并给你更新之后的最终 state。

**【注意】：**

- `setNumer(num + 1)` ： 这句话意思是，将 **下一次**的 _state_ 值传入函数， _React_ 看到之后就用当前的值**替换** _num_
- `setNumber(n => n + 1)` ：这句话意思是，根据 _state_ 队列中的前一个 _state_ 值来计算这次的值，也就是告诉 _React_ 要 **怎么做**

例如：

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

_React_ 在执行事件处理函数时处理的过程：

1. `setNumber(number + 5)`：`number` 为 `0`，所以 `setNumber(0 + 5)`。React 将 _“替换为_ `5`_”_ 添加到其队列中。
2. `setNumber(n => n + 1)`：`n => n + 1` 是一个更新函数。React 将该函数添加到其队列中。
3. `setNumber(42)`：React 将 _“替换为_ `42`_”_ 添加到其队列中。

所以，最后的结果时 _42_

事件处理函数执行完成后，React 将触发**重新渲染**。在重新渲染期间，React 将处理队列。更新函数会在渲染期间执行，因此 **更新函数必须是** [纯函数](https://react.docschina.org/learn/keeping-components-pure) 并且只 **返回** 结果。

### 更新 state 中的对象

可以去看官网：[https://react.docschina.org/learn/updating-objects-in-state](https://react.docschina.org/learn/updating-objects-in-state)

补充一个JS基本常识：

1、number、string、boolean这些基本数据类型都是 **不可变的（immutable）**，修改值，让其指向新开辟的空间，原来的空间回收，**注意：不是在原来的空间上直接修改的，而是重新开辟了一个新的空间**

2、引用数据类型，是可变的，在指向的是**引用地址**，引用地址指向的才是真正的数据

所以说，要想 _React_ 知道，_state_ 中的引用数据发生了改变的话，就需要 **覆盖** **_state_** **数据的地址**。（也就是，创建一个新的数组 或 对象来替换原有的值，让 _state_ 指向新的地址）

总结：**就是浅拷贝是不行的，需要用深拷贝，要指向新的内存地址**，怎么替换呢？在 _React_ 中使用 _useState_ 中的第二个属性，也就是那个 _setxxx_

还可以**创建一个新的**对象或数组，_copy_初始的对象或数组修改新的值，最后在给_state_

#### 方法：

可以使用扩展运算符来浅拷贝对象

```jsx
setPerson({ // 这里是重新创建了一个新的对象，这个新对象中的内容，是从原来的对象上复制过来的
  ...person, // 复制上一个 person 中的所有字段
  firstName: e.target.value // 但是覆盖 firstName 字段 
});

或者：
const newPerson = {...person, fristName: e.target.value}
setPerson(newPerson);
```

对于多层嵌套的对象，可以使用 **Immer** 库，方便“修改”，这里的修改，指的是表面上”直接修改“，但是内在还是遵守 **“替换”** 的

使用 Immer：

使用这个库，就可以直接修改数据，跟原来操作是一样的

1. 运行 `npm install use-immer` 添加 Immer 依赖
2. 用 `import { useImmer } from 'use-immer'` 替换掉 `import { useState } from 'react'`

```jsx
// state 对象 
const [person, updatePerson] = useImmer({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});

function handleNameChange(e) {
  // 使用 Immer 库，只需要正常修改即可
  updatePerson(draft => {
    draft.artwork.title = e.target.value;
  });

  // 如果是 state，需要全部替换
  setPerson({
    ...person,
    artwork: {
      ...person.artwork,
      title: e.target.value
    }
  })
}
```

## 表单(受控组件)

收到 _state_ 中数据的控制，换句话说，就是 _state_ 中的数据更改之后，表单中的数据也会发生变化。**所有的表单元素，得到表单值全部都是value** 使用`e.target.value`

#### 示例

**普通input输入框**

```jsx
import { useState } from "react";
export default function Test() {
  const [value, setValue] = useState("");
  function handleChange(e) {
    setValue(e.target.value)
  }
  return (
    <input 
      type="text" 
      value={value} 
      placeholder="please input your name" 
      onChange={handleChange} 
    />
  );
}
```

**文本域**

与上同理，也是一个 _value_ 值来控制，输入输出

**复选框**

```jsx
// Test
import {useState} from 'react';
const initArr = [
  {id: 1, content: '吃饭', name: "eat", checked: false},
  {id: 2, content: '睡觉', name: "sleep", checked: false},
  {id: 3, content: '打豆豆', name: "play", checked: false},
]
export default function Test (){
  const [checkBox, setCheckBox] = useState(initArr);
  
  function handleChange(checkId) {
    // 修改点击项的 checked 值，相当于 修改数组中对象的值
    setCheckBox(checkBox.map(cb => {
      if (cb.id === checkId) {
        return {
          ...cb,
          checked: !cb.checked
        }
      } else {
        return cb;
      }
    }));
  }
  return (
    <> 
      {
        checkBox.map(cb => (
          <div key={cb.id}>
            <input 
              type="checkbox" 
              name={cb.name} 
              checked={cb.checked} 
              onChange={() => handleChange(cb.id)}
            />
            <span>{cb.content}</span>
          </div>
        ))
      }
    </>
  );
}
```

**下拉列表**

```jsx
// Test
import {useState} from 'react';
export default function Test (){
  const [value, setValue] = useState("css");
  function handleChange(e) {
    setValue(e.target.value);
  }
  return (
    <> 
      <select value={value} onChange={handleChange}>
        <option value="html">HTML</option>
        <option value="css">CSS</option>
        <option value="js">JS</option>
        <option value="vue">vue</option>
        <option value="react">react</option>
      </select>
    </>
  );
}
```

## 状态提升

## useEffect

这是第二个认识的 _hook_，不错，又进一步！

这里先了解一个概念：

【编程范式】：

1、**命令式编程**：告诉计算机怎么做（How），我们需要给计算机指明每一个步骤

- 面向过程
- 面向对象

2、**声明式编程**：告诉计算机我要什么（What）

- 函数式编程
- _DSL_（领域专用语言，_HTML、CSS、SQL_）

_Hooks_ 的出现，还有更加重要的一个信号，那就是整个 _React_ 思想上面的转变，从“面向对象”的思想开始转为“函数式编程”的思想。这是编程范式上面的转变。

### useEffect 是用来干什么的呢？

_useState_ 是用来**_声明状态_**的，这个 _useEffect_ 是用来处理 **_副作用_** 的 。

常见的副作用有：

- 发送网络请求
- 添加一些监听的注册和取消注册
- 手动修改 _DOM_

### 使用 useEffect

useEffect 执行时机：在 DOM 生成之后执行(commit 之后)，异步执行，不阻塞主进程

useEffect 会返回一个**清理函数**，在组件被摧毁时使用（在下一次函数组件渲染之前执行）

1、执行清理工作

```jsx
import { useState, useEffect } from 'react';

function App() {

  let [count, setCount] = useState(0);

  useEffect(()=>{
    // 书写你要执行的副作用，会在组件渲染完成后执行
    const stopTimer = setInterval(()=>{
      console.log("Hello");
    },1000)   

    // console.log("副作用函数执行了");
    // 在 useEffect 中，可以返回一个函数，该函数我们称之为清理函数（一般就是做一些清理操作）
    // 该函数会在下一次渲染之后，但是在执行副作用操作之前执行
    return ()=>{
      // console.log("清理函数执行了");
      clearInterval(stopTimer);
    }
  });


  function clickhandle() {
    setCount(++count);
  }

  return (
    <div>
      <div>你点击了{count}次</div>
      <button onClick={clickhandle}>+1</button>
    </div>
  );
}

export default App;
```

2、 副作用的依赖

目前，我们的副作用函数，每次重新渲染后，都会重新执行，有些时候我们是需要设置依赖项，传递第二个参数，第二个参数为一个**依赖数组**

```jsx
import { useState, useEffect } from 'react';

function App() {

  let [count1, setCount1] = useState(0);
  let [count2, setCount2] = useState(0);
  let [count3, setCount3] = useState(0);

  useEffect(()=>{
    console.log("执行副作用函数");
  },[count1]);

  return (
    <div>
      <div>count1:{count1}</div>
      <div>count2:{count2}</div>
      <div>count3:{count3}</div>
      <button onClick={()=>setCount1(++count1)}>+1</button>
      <button onClick={()=>setCount2(++count2)}>+1</button>
      <button onClick={()=>setCount3(++count3)}>+1</button>
    </div>
  );
}

export default App;
```

3、 如果想要副作用**只执行一次**，传递第二个参数为一个空数组

```jsx
useEffect(()=>{
  console.log("执行副作用函数");
},[]);
```

### useEffect 怎么知道依赖项发生了变化？
React 通过浅比较（Shallow Comparison） 检测依赖项变化，但比较的是每次渲染时的值，具体机制如下：

1. React 为每次渲染“保存”一份依赖数组
每次组件渲染时，React 会记录当前渲染中 useEffect 的依赖数组（例如 [a, b, c]）。
这些依赖值是本次渲染闭包中的值（不是引用，是具体值）。
2. 执行前进行严格相等比较
在本次渲染的提交阶段（Commit Phase），React 会：
+ 取出上一次渲染保存的依赖数组（旧值）。
+ 与本次渲染的依赖数组（新值）逐项对比。
+ 使用 Object.is() 规则判断是否变化（类似 ===，但处理 NaN 和 -0 更严格）。
只要有一项不相等，就触发 Effect 重新执行。

## react-router

**补充知识**：

前端路由与后端路由的区别：

### 快速搭建服务器

又学到一个好东西，_json-server_ 可以快速搭建服务器，用来测试就很不错

### 如何使用react-router？

先安装 _npm i react-router-dom_ 依赖，在 _main.js_ 中引入，跟 _Vue_ 不一样的是， _Vue_ 是将所有的东西进行配置，_React_ 是当成组件使用

1、_main.js_

```jsx
import { BrowserRouter } from "react-router-dom";
import App from "./App.jsx";

const app = createRoot(document.getElementById("root"));
app.render(
  <BrowserRouter> // 这个就是 history 模式
    <App />
  </BrowserRouter>
);
```

2、在导航栏组件中，使用 _react-router_

```jsx
import { Routes, Route, NavLink, Navigate } from "react-router-dom";

... 内容
  <NavLink to="/home" className="navigation">主页</NavLink>
{/* react 切换可视区 */}
<div className="routes">
  <Routes>
    {/* 填写路由组件的路径和对应的组件 */}
    <Route path="/home" element={<Home />} />
    <Route path="/about" element={<About />} />
    <Route path="/add" element={<Add />} />
    {/* 重定向 */}
    <Route path="/" element={<Navigate replace to="/home" />} />
  </Routes>
</div>
```

## _React-router v6_ 路由总结

### 组件

- **BrowserRouter**：整个前端路由以 history 模式开始，包裹根组件
- **HashRouter**：整个前端路由以 hash 模式开始，包裹根组件
- **Routes**：类似于 v5 版本的 Switch，主要是提供一个上下文环境
- **Route**：在 Route 组件中书写你对应的路由，以及路由所对应的组件

- path：匹配的路由
- element：匹配上该路由时，要渲染的组件

- **Navigate**：导航组件，类似于 useNavigate 的返回值函数，只不过这是一个组件
- **NavLink**：类似于 Link，最终和 Link 一样，会被渲染为 a 标签，注意它和 Link 的区别，实际上就是当前链接，会有一个名为 active 的激活样式，所以一般用于做顶部或者左侧导航栏的跳转

### _Hooks_

- **useLocation**：获取到 location 对象，获取到 location 对象后，我们可以获取 state 属性，这往往是其他路由跳转过来的时候，会在 state 里面传递额外的数据
- **useNavigate**：调用之后会返回一个函数，通过该函数**可以做跳转**。
- **useParams**：获取**动态参数** `**path="/detail/:id"**`
- **useRoutes**：类似于 _Vue_ 的写法

### 补充内容

### _useRoutes_

使用示例如下：

```jsx
function Router(props) {
  return useRoutes([
    {
      path: "/home",
      element: <Home />,
    },
    {
      path: "/about",
      element: <About />,
    },
    {
      path: "/add",
      element: <AddOrEdit />,
    },
    {
      path: "/detail/:id",
      element: <Detail />,
    },
    {
      path: "/edit/:id",
      element: <AddOrEdit />,
    },
    {
      path: "/",
      element: <Navigate replace to="/home" />
    }
  ]);
}

export default Router;

在需要展示数据的地方：
  <Router />
```

### 嵌套路由

直接在 useRoutes 进行 **chilren** 属性的配置即可，类似于 vue-router，children 对应的是一个数组，数组里面是一个一个路由对象

```jsx
{
  path: "/about",
    element: <About />,
    children : [
    {
      path : "email",
      element : <Email/>
    },
    {
      path : "tel",
      element : <Tel/>
    },
    {
      path : "",
      element: <Navigate replace to="email" />
    }
  ]
},

在可视区使用：
  <OutLet></OutLet> children中的组件就会出现在这里面
```

之后，使用 Outlet 组件，该组件表示匹配上的子路由组件渲染的位置。

## react-redux

安装：

```bash
npm install @reduxjs/toolkit react-redux
```

类似于 _Vue_ 的状态管理工具，作用就是提供一种方法让任意两个组件之间通信。

1. 定义个 slice 对象：使用 `createSlice`
2. 定义store数据：`initialState`
3. 定义方法：`reducers` 在这个对象中配置方法，每个方法形参都包含(state, action)，通过state获得store数据，action获得附加的值
4. 使用store对象：两个hook `useSelector, useDispatch`

`useSelector` : 获得所有 store 最新的state值

`useDispatch` : 修改store对象中的方法

例如：

### 创建分片

```jsx
import { createSlice } from "@reduxjs/toolkit";

export const countSlide = createSlice({
  // 数据部分
  name: "counter", // 给每一个store一个名字，告诉store要管理的是哪个的状态
  initialState: {
    value: 0, // 初始化状态值
  },

  // 方法 函数 (state, action)
  reducers: {
    add: (state) => {
      state.value += 1;
    },
    jian: (state) => {
      state.value -= 1;
    },

    // incrementByAmount: (state, action) => {
    //   state.value += action.payload;
    // },
     
     // action({
     //    type: "counter/jian" //表示从那个函数获得
     //    payload: undefined 表示从外传进来的形参(附加的值)
     // })
  },
});
export const { add, jian } = countSlide.actions; // 向外提供可修改状态的函数
export default countSlide.reducer; // 默认导出一个 reducer，方便管理
```

### 创建 store

通常，我们还会创建一个名为 store.js 的文件来管理项目中用到的所有 reducer 对象

```jsx
import { configureStore } from "@reduxjs/toolkit";
import countSlide from "./countSlide";
export default configureStore({
  reducer: {
    counter: countSlide,
    // ...
  },
});
```

### 导入 store

当然在 index.js 文件中需要引入 _react-redux_ 这个配置，具体是引入 _Provider_ 和 我们自己创建的 _store.js_ 文件

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import { Provider } from "react-redux"; // 在react中使用react-redux
import store from "./store";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

### 使用状态

在组件中使用 store 对象

```jsx
// Test
import { useSelector, useDispatch } from "react-redux";
import { add, jian } from "./countSlide";
export default function Test() {
  // useSelector 获得 store 中状态值的最新值
  const count = useSelector((state) => state.counter.value);
  // useDispatch 返回 store 的 dispatch 方法让你可以去修改某个 store 中提供的 actions
  const dispatch = useDispatch();

  return (
    <>
      <button className="add" onClick={() => dispatch(add())}>
        +
      </button>
      <span className="title">{count}</span>
      <button className="jian" onClick={() => dispatch(jian())}>
        -
      </button>
    </>
  );
}
```

# 二、进阶篇

## 属性默认值

## HOC

高阶组件 _Higher-Order Components，在 React 中是一种逻辑复用的技巧_

一般以 with 命名

**【高阶组件是一个函数】**

这个点非常有意思，很多人一看到这个名字，自然的会认为高阶组件是一个组件，但是往往有些名字具有欺骗性，就像 _JavaScript_ 会被误认为和 _Java_ 相关一样。

**高阶组件就是接收一个组件，返回一个新组件。用于对组件的横向抽离，抽离公共代码。**

例如：

【为每一个组件都添加日志信息】

```jsx
// WithLog.jsx
import React, { useEffect } from "react";
/**
 * 高阶组件 -> 用于log日志记录
 * @param {*} Comp 组件
 * @returns 返回一个新组件
 */
export default function WithLog(Comp) {
  return function WithLogHOC(props) {
    useEffect(() => {
      console.log(`${Comp.name}已经创建，时间为：${new Date().toLocaleString()}`);
      return () => {
        console.log(`${Comp.name}被销毁，时间为${new Date().toLocaleString()}`);
      };
    }, []);

    return <Comp {...props} />;
  };
}

// App,jsx -> 使用高阶组件
import React, { useState } from "react";
import ChildComp1 from "./ChildComp1";
import ChildComp2 from "./ChildComp2";
import WithLog from "./HOC/WithLog";

export default function App() {
  const NewChildComp1 = WithLog(ChildComp1);
  const NewChildComp2 = WithLog(ChildComp2);
  const [toggle, setToggle] = useState(true);

  return (
    <>
      <button onClick={() => setToggle(!toggle)}>切换状态</button>
      {toggle ? <NewChildComp1 name={"张三"} /> : <NewChildComp2 age={18} />}
    </>
  );
}
```

但其实，高阶组件的出现，是为了解决类组件横向抽离而出现的。现在，以函数组件为主的 React，其实用自定义 hook 就可以实现逻辑复用了~

## Ref

就是用于获取 dom 元素的，如果不是万不得以，就不要用。最好是，**数据**来操作 UI 视图。

写下这个知识点的时候，用的是 react19，很多 API 都废弃了

### _createRef_

很多都是用于**类组件**中，不过现在都是用的函数组件（类组件我也不是很了解）

比如：

- 使用 `_createRef_` -> **获取某个 dom 元素【目前还可以用，但是已经是过时 API 了】**

```jsx
import React, { Component } from "react";

export default class App extends Component {
  inputRef = React.createRef();
  
  handleClick = () => {
    this.inputRef.current.focus();
  };
  render() {
    return (
      <div>
        <input type="text" ref={this.inputRef} />
        <button onClick={this.handleClick}>聚焦</button>
      </div>
    );
  }
}
```

除了在 _JSX_ 中关联 _Ref_，我们还可以直接关联一个类组件，这样就可以直接**调用该组件内部的方法**

```jsx
// 父组件
import React, { Component } from "react";
import Comp from "./Comp";

export default class App extends Component {
  inputRef = React.createRef();
  compRef = React.createRef();
  handleClick = () => {
    this.inputRef.current.focus();
    this.compRef.current.test();
  };
  render() {
    return (
      <div>
        <Comp ref={this.compRef} />
        <input type="text" ref={this.inputRef} />
        <button onClick={this.handleClick}>聚焦</button>
      </div>
    );
  }
}

// 子组件
import React, { Component } from "react";

export default class Comp extends Component {
  test = () => {
    console.log("调用Comp组件中test方法");
  };
  render() {
    return <h1>这是Comp组件</h1>;
  }
}
```

### _forwardRef_

有几个功能：

#### 父组件操作子组件的 dom 元素

跟类组件中的 refs、createRef 差不多，类组件中是将 ref 挂载到了子组件上，然后操作子组件的 dom 元素

```jsx
// 父组件
import React, { useRef } from "react";
import Comp from "./Comp";
export default function App() {
  const compRef = useRef(); // 用comRef来控制要操作的dom元素
  function handleClick() {
    compRef.current.focus();
  }

  return (
    <>
      <button onClick={handleClick}>聚焦</button>
      <Comp ref={compRef} /> // 这里也是挂载在子组件上，注意，这里传递只能用 ref
    </>
  );
}

// 子组件
import React from "react"; 
function Comp(props, ref) { // 经过 forwardRef 处理后，React会转发两个参数给Comp组件
  return (
      <input type="text" ref={ref} />
  );
}
// 如果不用这个静态方法，函数组件只会接收一个参数 props
export default React.forwardRef(Comp);
```

#### 多个组件间的 ref 传递

给子组件绑定一个 `ref`在用`React.forwardRef`将组件包裹起来，在传递到下一个组件，就可以了

#### 与`useImperativeHandle`hook 联用

React.forwardRef(组件名)可以让父组件操作子组件，这个 hook ，可以让子组件觉得向父组件暴露什么东西

```
useImperativeHandle(ref,createHandle,dependencies?)
```

```jsx
// 父组件
import React, { useRef } from "react";
import Input from "./Input";
export default function App(props) {
  const ref = useRef();
  function handleClick() {
    ref.current.focus();
    ref.current.all.style.opacity = 0.5; // 可以操作整个dom节点
  }
  return (
    <>
      <Input ref={ref} />
      <button onClick={handleClick}>聚焦</button>
    </>
  );
}

// 子组件
import React, { useRef, useImperativeHandle } from "react";

export default React.forwardRef(function Input(props, ref) {
  const inputRef = useRef();
  const allRef = useRef();
  useImperativeHandle( // 用这个hook能够让子组件觉得，向父组件暴露什么东西
    ref,
    () => {
      return {
        focus() {
          inputRef.current.focus(); // 暴露一个方法
        }, 
        all: allRef.current, // 暴露dom元素
      };
    },
    []
  );

  return (
    <>
      暴露少一点的dom元素：
      <input type="text" ref={inputRef} />
      <br />
      全部暴露dom元素：
      <input type="text" ref={allRef} />
    </>
  );
});
```

`forwardRef 和 useImperactiveHandle`的使用场景：

[https://www.yuque.com/fanxiaolan-975oy/zvuxpw/yf2shb447040nt6m](https://www.yuque.com/fanxiaolan-975oy/zvuxpw/yf2shb447040nt6m)

### useRef

>  useRef 与 React.createRef 的区别:
>  - useRef：一般用于**函数组件**，挂载在 fiber 上，**只会更新一次**
>  - React.createRef：一般用于**类组件**，每次更新都会重新创建

```
const ref = useRef(initialValue);
```

**【注意】改变 ref 不会触发重新渲染**, 这意味着 ref 是存储一些**不影响组件视图输出**信息的完美选择。

例如：

存储在 ref 中存储**定时器的 id**，随后可以轻松的清除定时器

```jsx
import React, { useRef, useState } from "react";

export default function App(props) {
  const timerRef = useRef(null);
  const [count, setCount] = useState(0);
  const start = () => {
    clearInterval(timerRef.current);
    timerRef.current = setInterval(() => {
      setCount((count) => count + 1);
    }, 100);
  };

  const stop = () => {
    clearInterval(timerRef.current);
  };

  return (
    <>
      <p>{count}</p>
      <button onClick={start}>开启定时器</button>
      <button onClick={stop}>停止定时器</button>
    </>
  );
}
```

使用 ref 可以确保：

- 可以在重新渲染之间 **存储信息**（普通对象存储的值每次渲染都会重置）。
- 改变它 **不会触发重新渲染**（状态变量会触发重新渲染）。
- 对于组件的每个副本而言，**这些信息都是本地的**（外部变量则是共享的）。

注意：使用`useRef`hook 一般用于组件内部的 React 内置组件（input、button ...），如果绑定在自定义组件上，需要给自定义组件加上`React.forwardRef`将 ref 传递。

## Context

可以创建一个**上下文环境**，用于解决组件之间共享数据的问题（当然 redux 也可以解决这个问题，它的底层就是用 context）

话说回来，这个上下文环境，可以提供一些数据，可以往环境中加入数据。

上下文使得组件能够无需通过显式传递参数的方式 [将信息逐层传递](https://react.docschina.org/learn/passing-data-deeply-with-context)。

### React.createContext

```jsx
const SomeContext = React.createContext(defaultValue);

// defaultValue: 默认值，静态的，永远不会改变
```

返回值：

- `SomeContext.Provider`可以在上下文中**添加**数据
- `SomeContext.Consumer`可以在上下文中**获取**数据【现在更加推荐用`useContext`】

```jsx
// App.jsx
import React, {useState} from "react";
function App() {
  const CountContext = React.createContext(1); // 创建上下文
  const [count, setCount] = useState(1);
  return (
    <CountContext.Provider value={{count, setCount}}> // 添加数据
      <Page />;
    </CountContext.Provider>
  );
}

// Page.jsx
import React from "react";
function Page() {
  return (
    <CountContext.Consumer> // 获取数据 -> 用回调获取
      {
        (context) => <button onClick={() => context.setCount(context.count + 1)}>+1<button/>
      }
    </CountContext.Consumer>
  );
}
```

### useContext

更加**推荐**的获取上下文数据的方式

```jsx
const value = useContext(SomeContext);
```

一般情况下，会创建一个`contexts.js`的文件夹，存放所有的上下文

例如：

```jsx
// contexts/index.js
import {createContext} from "react";

export const CountContext = createContext(1);
export const ThemeContext = createContext("light");
//...
```

提供数据还是用`Provider`

获取数据

```jsx
import React, { useContext } from "react";
import { CountContext } from "./contexts";

export default function Child() {
  const {count, setCount} = useContext(CountContext);
  return (
    <button onClick={() => setCount(count + 1)}>点我加一</button>
  );
}
```

## Protal

可以将一段 JSX 代码指定在某个地方渲染，是 **react-dom** 库提供的。

可以用于一些需要渲染在**出乎意料**地方的 JSX，还可以在不是 react 构建的页面使用 protal 渲染 react 代码**（将 React 组件渲染到非 React DOM 节点 ）**

```jsx
import {createProtal} from "react-dom";

createProtal(JSX, domNode, key?)
```

调用 createPortal 创建 portal，并传入 JSX 与实际渲染的目标 DOM 节点：

```jsx
import React, { useState } from "react";
import Modal from "./Modal";
export default function App() {
  const [isShow, setIsShow] = useState(false);
  return (
    <>
      <div>
        <h1>这是App组件</h1>
        <button onClick={() => setIsShow(!isShow)}>打开模态框</button>
      </div>
      <div id="modal">{isShow ? <Modal /> : null}</div>
    </>
  );
}

import React from "react";
import { createPortal } from "react-dom";
export default function Modal() {
  return createPortal(
    <div
      style={{
        width: "300px",
        height: "100px",
        border: "2px solid",
        textAlign: "center",
        lineHeight: "100px",
        position: "absolute",
        top: "calc(50% - 50px)",
        left: "calc(50% - 150px)",
      }}
    >
      这是模态框
    </div>,
    document.getElementById("modal")
  );
}
```

【注意】：使用`createProtal`冒泡的顺序是根据 react 树冒泡的，不是根据最后生成的 dom 节点。

## 错误边界

目前版本中，还没有提供给函数组件用的错误边界组件（类组件中可以使用官方定义好的错误边界组件），没错，错误边界是一个组件。

**类组件**

```jsx
import React, { Component } from "react";

export default class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新状态，以便下一次渲染将显示降级 UI
    return { hasError: true };
  }
  componentDidCatch(error, info) {
    console.log(info); // 执行栈的错误信息 -> 一般可以上传到服务器当错误日志
  }
  render() {
    if (this.state.hasError) {
      // 自定义降级 UI，当要显示的内容出错时
      return <h1 style={{ color: "red" }}>{this.props.fallback}</h1>;
    }

    return this.props.children;
  }
}

// 使用
// ... 
<ErrorBoundary fallback="child3组件错误了~">
    <Child3 />
</ErrorBoundary>
```

【注意】

- 错误边界组件包裹下面的所有子组件，如果报错了都能被捕获到（比如，Child3 内部用到了 Child4，Child4 报错了，那么也会被错误边界组件捕获到）
- 一些异步渲染出来的 UI，无法捕获
- 事件处理
- 异步代码（例如 `setTimeout` 或 `requestAnimationFrame` 回调函数）
- 服务端渲染

**函数组件**

函数组件中可以使用一个第三方库 [react-error-boundary](https://github.com/bvaughn/react-error-boundary)

```jsx
import React from "react";
import Child3 from "./Child3";
import { ErrorBoundary } from "react-error-boundary";
export default function Child1() {
  return (
    <div style={{ border: "2px solid", width: "400px", height: "200px" }}>
      Child1组件
      <div>
        <ErrorBoundary fallback={<h1 style={{ color: "red" }}>出错了</h1>}>
          <Child3 />
        </ErrorBoundary>
      </div>
    </div>
  );
}
```

## 组件渲染性能优化

### shouldComponentUpdate

用于类组件中，这是一个生命周期函数，可以用 PureComponent 代替（不过现版本，）

PureComponent 内部是通过 shouldComponentUpdate 函数实现的

使用：

```jsx
shouldComponentUpdate(nextProps, nextState, nextContext)
```

当收到新的 props 或 state 时，React 会在渲染之前调用 `shouldComponentUpdate`，默认值为 `true`。初始渲染或使用 `forceUpdate` 时将不会调用此方法。

```jsx
import React, { Component } from "react";

// 实现对象浅比较
function isEqual(obj1, obj2) {
  for (const key in obj1) {
    if (!Object.is(obj1[key], obj2[key])) {
      return false;
    }
  }
  return true;
}

export default class Child extends Component {
  state = {
    counter: 1,
  };

  shouldComponentUpdate(nextProps, nextState, nextContext) {
    // 跟之前的 props 和 state 进行浅比较，如果相等那么返回 false，说明不用重新渲染
    if (isEqual(this.props, nextProps) && isEqual(this.state, nextState)) {
      return false;
    }
    return true;
  }

  render() {
    console.log("Child组件渲染了~");
    return (
      <div>
        <button onClick={() => this.setState({ counter: this.state.counter + 1 })}>+1</button>
        <p>数字：{this.state.counter}</p>
      </div>
    );
  }
}
```

### PureComponent

使用 `PureComponent` 只要 props 和 state 没有改变，React 就不需要重新渲染组件。

```jsx
import React, { PureComponent } from "react";

export default class Child extends PureComponent {
  state = {
    counter: 1,
  };

  render() {
    console.log("Child组件渲染了~");
    return (
      <div>
        <button onClick={() => this.setState({ counter: this.state.counter + 1 })}>+1</button>
        <p>数字：{this.state.counter}</p>
      </div>
    );
  }
}
```

前面两个一般用于类组件，那么函数组件呢？

### React.memo

高阶组件，**props 和 context** 改变了那么组件会重新渲染

其实底层实现，就是用套了一层 PureComponent，我们知道，PureCompoent 是看 props 和 state 是否发生变化。只不过，函数组件的 state 因为 hook 的原因，优化了。所以只需要看 props 是否发生改变，就能知道要不要渲染组件。

使用：

```jsx
import {memo} from "react";
memo(Component, arePropsEqual?)

// component：待优化的组件
// arePropsEqual：自定义props相等的函数，注意这个地方也应该使用浅比较
function arePropsEqual(preProps, nextProps) {
  return true;  // 不渲染。因为props是相同的
         false; // 渲染
}
```

例如：props 修改

```jsx
// App.jsx
import React, { useState } from "react";
import Child from "./Child";
export default function App() {
  const [counter1, setCounter1] = useState(1); // App自用的数
  const [counter2, setCounter2] = useState(1); // App传递给子组件的数

  console.log("App组件被渲染了~");
  return (
    <>
      <h1>App组件</h1>
      {/* 修改App的数值，不应该重新渲染Child组件，不使用memo就会重新渲染Child组件 */}
      <button onClick={() => setCounter1(counter1 + 1)}>+1</button>
      <p>数字：{counter1}</p>
      <Child counter={counter2} setCounter={setCounter2} />
    </>
  );
}

// Child.jsx
import React from "react";
function Child(props) {
  console.log("Child组件被渲染了");
  return (
    <>
      <h2>Child组件</h2>
      <p>App组件传递的值：{props.counter}</p>
      <button onClick={() => props.setCounter(props.counter + 1)}>修改counter值</button>
    </>
  );
}

export default React.memo(Child);
```

例如：context 修改

```jsx
import { createContext, memo, useContext, useState } from "react";
import "./index.css";

const ThemeContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState("dark");

  function handleClick() {
    setTheme(theme === "dark" ? "light" : "dark");
  }

  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={handleClick}>Switch theme</button>
      <Greeting name="Taylor" />
    </ThemeContext.Provider>
  );
}

const Greeting = memo(({ name }) => {
  console.log("Greeting组件被渲染了", new Date().toLocaleTimeString());
  const theme = useContext(ThemeContext);
  return <h3 className={theme}>Hello, {name}!</h3>;
});
```

### useCallback

允许多次渲染中，缓存**函数本身**的一个 hook

为什么要缓存呢？因为，函数组件中，每次修改了 state 会导致重新渲染组件，此时，组件中的函数都会被重新创建，**可能导致依赖该函数的子组件也重新渲染**。所以这个地方可以进行优化。

用法：

```jsx
const cachedFn = useCallback(fn, dependencies)
```

```jsx
// App.jsx
import React, { useState, useCallback } from "react";
import Child1 from "./Child1";

function App() {
  console.log("App组件渲染了");
  const [counter, setCounter] = useState(1);
  const [counter1, setCounter1] = useState(1); // 传递给child1组件的数据

  // 没使用 useCallback 优化前，
  // 子组件修改父组件的状态，导致重新渲染，test函数都会被重新创建，
  // 从而导致 props 发生改变，导致子组件重新渲染
  // useCallback 将test函数缓存起来，只有依赖项改变了，缓存才会重新加载
  const test = useCallback(() => {
    console.log("test函数");
  }, []);

  return (
    <>
      <p>App组件，数字：{counter}</p>
      <button onClick={() => setCounter(counter + 1)}>+1</button>
      <Child1 counter={counter1} setCounter={setCounter1} test={test} />
    </>
  );
}
export default App;

// Child1.jsx
import { memo } from "react";
function Child1(props) {
  console.log("Child1组件渲染了");
  return (
    <>
      <p>Child1组件，数字：{props.counter}</p>
      <button onClick={() => props.setCounter(props.counter + 1)}>+1</button>
    </>
  );
}
// memo：只要 props 和 context 没有改变，那么Child1不会被重新渲染
export default memo(Child1);
```

### useMemo

缓存**数值**，类似于 Vue 中的计算属性，调用函数并缓存结果

用法：

```jsx
const cachedValue = useMemo(calculateValue, dependencies)
```

例如：

```jsx
import React, { useState, useMemo } from "react";

function App() {
  console.log("App组件渲染了");
  const [counter, setCounter] = useState(1);
  const [val, setVal] = useState("");

  const countNum = useMemo(
    function getCount() {
      console.log("函数被调用了");
      return counter + 100;
    },
    [counter]
  );

  return (
    <>
      <h1>App组件：{countNum}</h1>
      <button onClick={() => setCounter(counter + 1)}>+1</button>
      <input type="text" value={val} onChange={(e) => setVal(e.target.value)} />
    </>
  );
}
export default App;
```

## 关于什么时候进行性能优化

`React.memo()`：缓存组件

`useCallback()`：缓存函数

`useMemo()`：缓存数值

共同的使用场景：说白了，就是不想让**缓存值**的影响到其他的组件（依赖这些值的组件）渲染

还有一个其实也可以算到这个里面，

`useRef()`：只会更新一次，挂载载 fiber 上，不会触发重新渲染

# 三、深入理解篇

# React 与 Vue 描述 UI 的区别

一句话说，就是 React 使用 JSX 来描述 UI，Vue 使用模板来描述 UI

**JSX 起源：**

JSX 是由 React 团队提出来的，

React 团队认为，UI 本质上和逻辑是有耦合部分的：

- 在 UI 上面绑定事件
- 数据变化后通过 JS 去改变 UI 的样式或者结构

作为一个前端工程师，JS 是用得最多，所以 React 团队思考屏蔽 HTML，**整个都用 JS 来描述 UI**，因为这样做的话，可以让 **UI 和逻辑**配合得更加紧密，所以最终设计出来了类 XML 形式的 JS 语法糖

由于 JSX 是 JS 的语法糖（本质上就是 JS），因此可以**非常灵活**的和 JS 语法组合使用，例如：

- 可以在 if 或者 for 当中使用 jsx
- 可以将 jsx 赋值给变量
- 可以将 jsx 当作参数来传递，当然也可以在一个函数中返回一段 jsx

```jsx
function App({isLoading}) {
  if (isLoading) {
    return <h1>loading...<h1/>
  }
  return <h1>Hello World<h1/> 
}
```

**模板起源：**

Vue 的模板，应该是来源于后端语言。在前后端还未分离的时代，前端主要写 HTML、CSS 和一些简单的 JS 效果。前端在写完的 HTML 文件中挖坑（根据后端的模板语言的语法挖坑，比如 JSP）交给后端，后端连接数据库将数据回填到坑中，在返回给前端。

总结一下，就是 JSX 和模板都能描述 UI，但出发点不同

- Vue 的模块是从 **HTML** 来出发的，既然 HTML 能**描述 UI**，那么通过扩展让 HTML 也能描述逻辑的交互“从 UI 出发，扩展 UI，在 UI 中能够描述逻辑”
- React 的 JSX 是从 **JS** 出发的，JS 能**描述逻辑**，那么就扩展 JS 让它也能描述 UI“从逻辑出发，扩展逻辑，描述 UI”

这两者虽然都可以描述 UI，但是思路或者说方向是完全不同的，从而造成了整体框架架构上面也是不一样的。

# 虚拟 DOM

先明确一点，虚拟 DOM 是一个概念，并不是一个具体的技术实现

虚拟 DOM 和 JS 对象之间的关系：**前者是一种思想，后者是这种思想的具体实现。**

使用虚拟 DOM 的好处，

- 更新速度比操作原始 DOM 快
- 多平台适用

**那么为什么使用虚拟 DOM 更新速度更快呢？**

React 团队发现，前端开发者一般使用 `innerHTML`来渲染 UI，而不是直接使用 DOM API 来操作 dom 元素。

```
document.body.innerHTML = `
  <p>Hello World</p>
`;
```

第一次渲染：

|   |   |   |
|---|---|---|
||使用 innerHTML|使用虚拟 dom|
|JS 层面|JS 解析字符串|创建 JS 对象（实现虚拟 dom）|
|dom 层面|操作 dom 元素|操作 dom 元素|

可以看到第一次渲染，虚拟 dom 元素一定没有直接操作 dom 元素快，因为虚拟 dom 还需要在操作 dom 前套一层创建 JS 对象。但是跟 innerHTML 比较的话，其实也没差多少，都需要 JS 操作。

更新：

|   |   |   |
|---|---|---|
||使用 innerHTML|使用虚拟 dom|
|JS 层面|JS 解析字符串|创建 JS 对象|
|dom 层面|销毁原来所有的 DOM 节点|修改必要的 DOM 节点|
|dom 层面|创建和操作 dom 元素||

更新视图的时候，可以看到使用虚拟 DOM 会比单纯操作 dom 元素快的多，不需要全部销毁也不需要全部创建。

**多平台的渲染抽象能力**

UI = f（state）这个公式进一步进行拆分可以拆分成两步：

- 根据自变量的变化计算出 UI
- 根据 UI 变化执行具体的宿主环境的 API

虚拟 DOM 只是多真实 UI 的一个描述，回头根据不同的宿主环境，可以执行不同的渲染代码：

- 浏览器、Node.js 宿主环境使用 ReactDOM 包
- Native 宿主环境使用 ReactNative 包
- Canvas、SVG 或者 VML（IE8）宿主环境使用 ReactArt 包
- ReactTest 包用于渲染出 JS 对象，可以很方便地测试“不隶属于任何宿主环境的通用功能”

题目：什么是虚拟 DOM？其有点有哪些？

参考答案：

虚拟 DOM 最初是由 React 团队所提出的概念，这是一种编程的思想，指的是针对真实 UI DOM 的一种描述能力。

在 React 中，使用了 JS 对象来描述真实的 DOM 结构。虚拟DOM和 JS 对象之间的关系：前者是一种思想，后者是这种思想的具体实现。

使用虚拟 DOM 有如下的优点：

- 相较于 DOM 的体积和速度优势
- 多平台渲染的抽象能力

相较于 DOM 的体积和速度优势

- JS 层面的计算的速度，要比 DOM 层面的计算快得多

- DOM 对象最终要被浏览器显示出来之前，浏览器会有很多工作要做（浏览器渲染原理）
- DOM 上面的属性也是非常多的

- 虚拟 DOM 发挥优势的时机主要体现在更新的时候，相比较 innerHTML 要将已有的 DOM 节点全部销毁，虚拟 DOM 能够做到针对 DOM 节点做最小程度的修改

多平台渲染的抽象能力

- 浏览器、Node.js 宿主环境使用 ReactDOM 包
- Native 宿主环境使用 ReactNative 包
- Canvas、SVG 或者 VML（IE8）宿主环境使用 ReactArt 包
- ReactTest 包用于渲染出 JS 对象，可以很方便地测试“不隶属于任何宿主环境的通用功能”

在 React 中，通过 JSX 来描述 UI，JSX 仅仅是一个语法糖，会被 Babel 编译为 createElement 方法的调用。该方法调用之后会返回一个 JS 对象，该对象就是虚拟 DOM 对象，官方更倾向于称之为一个 React 元素。

# React 架构

果然架构这个东西不是人能搞出来了，确实要深入分析

React v16 之前都是用的 stack 架构，16 以后用的是 fiber 架构

那么两者有什么区别呢？后者比前者多了什么功能呢？

## 一切的开始

一切的一切都要从浏览器说起，我们知道页面是由浏览器画出来的，速度是一秒画 60 次。

也就是 1000ms / 60 = 16.6ms 浏览器必须在这 16ms 中执行完流水线。

事实上，浏览器在绘制页面前，还需要做其他很多事情：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748939272955-aaf71cac-6ae4-4205-9540-a0a12220a46b.png)

上图中的任务被称之为“**渲染流水线**”，每次执行流水线的时候，大致是需要如上的一些步骤，但是并不是说每一次所有的任务都需要全部执行：

- 当通过 JS 或者 CSS 修改 DOM 元素的几何属性（比如长度、宽度）时，会触发完整的渲染流水线，这种情况称之为重排（回流）
- 当修改的属性不涉及几何属性（比如字体、颜色）时，会省略掉流水线中的 Layout、Layer 过程，这种情况称之为重绘
- 当修改“不涉及重排、重绘的属性（比如 transform 属性）”时，会省略流水线中 Layout、Layer、Print 过程，仅执行合成线程的绘制工作，这种情况称之为合成

按照性能高低进行排序的话：合成 > 重绘 > 重排

## react的思考

看起来，好像与 react 没有关系，但有一个很大的关系就是，JS 的执行跟渲染页面是一个线程。如果 JS 执行时间过长，那么浏览器看起来就卡住了（其实是因为浏览器不能在 16.6ms 中执行完 JS 代码，就会出现掉帧）

### stack 架构

在 react v16 之前，react 描述虚拟 dom 是用 JS 对象的形式。是一种**树形**结构，在进行对比算法时，遍历树结构，采用递归并且该**递归不能停下来**。从而容易导致浏览器执行不完 JS 代码，导致卡帧。

比如：16 之前描述 dom 节点

假设有如下的 DOM 层次结构：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748939711185-23b17f08-308b-40ab-b3f2-989f6d0b814d.png)

那么转换成虚拟 DOM 对象结构大致如下：

stack 描述虚拟 dom

```js
{  
type : "div",  
props : {  
id : "test",  
children : [  
{  
type : "h1",  
props : {  
children : "This is a title"  
}  
}  
{  
type : "p",  
props : {  
children : "This is a paragraph"  
}  
},{  
type : "ul",  
props : {  
children : [{  
type : "li",  
props : {  
children : "apple"  
}  
},{  
type : "li",  
props : {  
children : "banana"  
}  
},{  
type : "li",  
props : {  
children : "pear"  
}  
}]  
}  
}  
]  
}  
}

```
### fiber 架构

那么为了解决这个 JS 执行过长并且一旦执行不能停止的问题，16 之后，

提出的 fiber 架构可以实现**时间切片**功能。在发现一帧时间已经不够，不能够再继续执行 JS，需要渲染下一帧的时候，这个时候就会打断 JS 的执行，优先渲染下一帧。渲染完成后再接着回来完成上一次没有执行完的 JS 计算。

fiber 其实是用**链表**来描述 dom 节点，可以认为是 16 之后虚拟 dom 的具体实现。

Fiber 本质上也是一个对象，但是和之前 React 元素不同的地方在于对象之间使用链表的结构串联起来，child 指向子元素，sibling 指向兄弟元素，return 指向父元素。

同样是上面的 dom 节点，fiber 描述起来，如下图：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748940039247-56ae1f1e-4827-4c7a-a6bf-a48c92c406ec.png)

使用链表这种结构，有一个最大的好处就是在进行整颗树的对比（reconcile）计算时，这个过程是可以被打断。

## I/O 优化方面

除此之外，在优化 I/O 方面。

从 React v16 开始引入了 Scheduler（调度器），用来调度任务的优先级。

UI = f（state）：

- 根据自变量的变化计算出 UI
- 根据 UI 变化执行具体的宿主环境的 API

React v16之前：

- Reconciler（协调器）：vdom 的实现，根据自变量的变化计算出 UI 的变化
- Renderer（渲染器）：负责将 UI 的变化渲染到宿主环境

从 React v16 开始，多了一个组件：

- **Scheduler（调度器）**：调度任务的优先级，高优先级的任务会优先进入到 Reconciler
- Reconciler（协调器）：vdom 的实现，根据自变量的变化计算出 UI 的变化
- Renderer（渲染器）：负责将 UI 的变化渲染到宿主环境

新架构中，Reconciler 的更新流程也从之前的递归变成了“可中断的循环过程”。

```jsx
function workLoopConcurrent{
  // 如果还有任务，并且时间切片还有剩余的时间
  while(workInProgress !== null && !shouldYield()){
    performUnitOfWork(workInProgress);
  }
}

function shouldYield(){
  // 当前时间是否大于过期时间
  // 其中 deadline = getCurrentTime() + yieldInterval
  // yieldInterval 为调度器预设的时间间隔，默认为 5ms
  return getCurrentTime() >= deadline;
}
```

每次循环都会调用 shouldYield 判断当前的时间切片是否有足够的剩余时间，如果没有足够的剩余时间，就暂停 reconciler 的执行，将主线程还给渲染流水线，进行下一帧的渲染操作，渲染工作完成后，再等待下一个宏任务进行后续代码的执行。

题目：是否了解过 React 的架构？新的 Fiber 架构相较于之前的 Stack 架构有什么优势？

参考答案：

React v15及其之前的架构：

- Reconciler（协调器）：VDOM 的实现，负责根据自变量变化计算出 UI 变化
- Renderer（渲染器）：负责将 UI 变化渲染到宿主环境中

这种架构称之为 Stack 架构，在 Reconciler 中，mount 的组件会调用 mountComponent，update 的组件会调用 updateComponent，这两个方法都会递归更新子组件，更新流程一旦开始，中途无法中断。

但是随着应用规模的逐渐增大，之前的架构模式无法再满足“快速响应”这一需求，主要受限于如下两个方面：

- CPU 瓶颈：由于 VDOM 在进行差异比较时，采用的是递归的方式，JS 计算会消耗大量的时间，从而导致动画、还有一些需要实时更新的内容产生视觉上的卡顿。
- I/O 瓶颈：由于各种基于“自变量”变化而产生的更新任务没有优先级的概念，因此在某些更新任务（例如文本框的输入）有稍微的延迟，对于用户来讲也是非常敏感的，会让用户产生卡顿的感觉。

新的架构称之为 Fiber 架构：

- Scheduler（调度器）：调度任务的优先级，高优先级任务会优先进入到 Reconciler
- Reconciler（协调器）：VDOM 的实现，负责根据自变量变化计算出 UI 变化
- Renderer（渲染器）：负责将 UI 变化渲染到宿主环境中

首先引入了 Fiber 的概念，通过一个对象来描述一个 DOM 节点，但是和之前方案不同的地方在于，每个 Fiber 对象之间通过链表的方式来进行串联。通过 child 来指向子元素，通过 sibling 指向兄弟元素，通过 return 来指向父元素。

在新架构中，Reconciler 中的更新流程从递归变为了“可中断的循环过程”。每次循环都会调用 shouldYield 判断当前的 TimeSlice 是否有剩余时间，没有剩余时间则暂停更新流程，将主线程还给渲染流水线，等待下一个宏任务再继续执行。这样就解决了 CPU 的瓶颈问题。

另外在新架构中还引入了 Scheduler 调度器，用来调度任务的优先级，从而解决了 I/O 的瓶颈问题。

# React 渲染过程

那么通过上面的内容，我们大致能够知道 Reactv16 之后，分为三个组件，

分别是调度器（Scheduler）、协调器（Reconciler）和渲染器（Renderer）

其实现代前端框架，有一个定性的公式

UI = f（state）

- 根据自变量计算 UI
- 根据 UI 变化执行对应宿主环境调的 API

对应的公式：

```jsx
const state = reconciler(update); // 通过 reconciler 计算出最新的状态
const UI = commit(state); // 根据上一步计算出来的 state 渲染出 UI
```

对应到 React 里面就两大阶段：

- render 阶段：调度合成虚拟 DOM，计算出最终要渲染出来的虚拟 DOM
- commit 阶段：根据上一步计算出来的虚拟 DOM，渲染具体的 UI

每个阶段对应不同的组件：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748953104160-3461b781-6600-42ac-bd05-404fe9b858d6.png)

- 调度器（Scheduer）：调度任务，为任务排序优先级，让优先级高的任务先进入到 Reconciler
- 协调器（Reconciler）：生成 Fiber 对象，收集副作用，找出哪些节点发生了变化，打上不同的 flags（节点更新 update，删除 delete 等），著名的 diff 算法也是在这个组件中执行的。
- 渲染器（Renderer）：根据虚拟 DOM **同步**的渲染节点到视图上。

一个例子：

```jsx
export default () => {
  const [count, updateCount] = useState(0);
  return (
    <ul>
    	<button onClick={() => updateCount(count + 1)}>乘以{count}</button>
      <li>{1 * count}</li>
      <li>{2 * count}</li>
      <li>{3 * count}</li>
    </ul>
  );
}
```

当用户点击按钮时，首先是由 Scheduler 进行任务的协调，render 阶段（虚线框内）的工作流程是可以随时被以下**原因中断**：

- 有其他更高优先级的任务需要执行
- 当前的 time slice 没有剩余的时间
- 发生了其他错误

【注意】：上面 **render 阶段**的工作是在**内存**里面进行的，不会更新宿主环境 UI，因此这个阶段即使工作流程反复被中断，用户也不会看到“更新不完整的UI”。

当 Scheduler 调度完成后，将任务交给 Reconciler，Reconciler 就需要计算出新的 UI，最后就由 Renderer 同步进行渲染更新操作。

如下图所示：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748953740420-636f003d-aee6-4c67-9c3e-7ae4e3cbe528.png)

## 调度器

在 React v16 版本之前，采用的是 Stack 架构，所有任务只能同步进行，无法被打断，这就导致浏览器可能会出现丢帧的现象，表现出卡顿。React 为了解决这个问题，从 v16 版本开始从架构上面进行了两大更新：

- 引入 Fiber
- 新增了 Scheduler

Scheduler 在浏览器的原生 API 中实际上是有类似的实现的，这个 API 就是 requestIdleCallback

MDN：[https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback](https://gitee.com/link?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWindow%2FrequestIdleCallback)

虽然每一帧绘制的时间约为 16.66ms，但是如果屏幕没有刷新，那么浏览器会安排长度为 50ms 左右的空闲时间。

为什么是50ms？

根据研究报告表明，用户操作之后，100ms以内的响应给用户的感觉都是瞬间发生，也就是说不会感受到延迟感，因此将空闲时间设置为 50，浏览器依然还剩下 50ms 可以处理用户的操作响应，不会让用户感到延迟。

虽然浏览器有类似的 API，但是 React 团队并没有使用该 API，因为该 API 存在兼容性问题。因此 React 团队自己实现了一套这样的机制，这个就是调度器 Scheduler。

后期 React 团队打算单独发行这个 Scheduler，这意味着调度器不仅仅只能在 React 中使用，凡是有涉及到任务调度需求的项目都可以使用 Scheduler。

## 协调器

协调器是 render 阶段的第二阶段工作，**类组件或者函数组件**本身就是在这个阶段被调用的。

根据 Scheduler 调度结果的不同，协调器起点可能是不同的

- performSyncWorkOnRoot（同步更新流程）
- performConcurrentWorkOnRoot（并发更新流程）

```jsx
// performSyncWorkOnRoot 会执行该方法
function workLoopSync(){
  while(workInProgress !== null){
    performUnitOfWork(workInProgress)
  }
}
```

```jsx
// performConcurrentWorkOnRoot 会执行该方法
function workLoopConcurrent(){
  while(workInProgress !== null && !shouldYield()){
    performUnitOfWork(workInProgress)
  }
}
```

新的架构使用 Fiber（对象）来描述 DOM 结构，最终需要形成一颗 Fiber tree，这不过这棵树是通过链表的形式串联在一起的。

workInProgress 代表的是当前的 FiberNode。

performUnitOfWork 方法会创建下一个 FiberNode，并且还会将已创建的 FiberNode 连接起来（child、return、sibling），从而形成一个链表结构的 Fiber tree。

如果 workInProgress 为 null，说明已经没有下一个 FiberNode，也就是说明整颗 Fiber tree 树已经构建完毕。

上面两个方法唯一的区别就是是否调用了 shouldYield方法，该方法表明了是否可以中断。

performUnitOfWork在创建下一个 FiberNode 的时候，整体上的工作流程可以分为两大块：

- 递阶段
- 归阶段

**递阶段**

递阶段会从 HostRootFiber 开始向下以**深度**优先的原则进行遍历，遍历到的每一个 FiberNode 执行 beginWork 方法。该方法会根据传入的 FiberNode 创建下一级的 FiberNode，此时可能存在两种情况：

- 下一级只有一个元素，beginWork 方法会创建对应的 FiberNode，并于 workInProgress 连接

```
<ul>
  <li></li>
</ul>
```

这里就会创建 li 对应的 FiberNode，做出如下的连接：

```
LiFiber.return = UlFiber;
```

- 下一级有多个元素，这是 beginWork 方法会依次创建所有的子 FiberNode 并且通过 sibling 连接到一起，每个子 FiberNode 也会和 workInProgress 连接

```
<ul>
  <li></li>
  <li></li>
  <li></li>
</ul>
```

此时会创建 3 个 li 对应的 FiberNode，连接情况如下：

```
// 所有的子 Fiber 依次连接
Li0Fiber.sibling = Li1Fiber;
Li1Fiber.sibling = Li2Fiber;

// 子 Fiber 还需要和父 Fiber 连接
Li0Fiber.return = UlFiber;
Li1Fiber.return = UlFiber;
Li2Fiber.return = UlFiber;
```

由于采用的是深度优先的原则，因此无法再往下走的时候，会进入到归阶段。

**归阶段**

归阶段会调用 completeWork 方法来处理 FiberNode，做一些副作用的收集。

当某个 FiberNode 执行完了 completeWork 方法后，如果存在兄弟元素，就会进入到兄弟元素的递阶段，如果不存在兄弟元素，就会进入父 FiberNode 的归阶段。

```
function performUnitOfWork(fiberNode){
  // 省略 beginWork 
  if(fiberNode.child){
    performUnitOfWork(fiberNode.child);
  }
  // 省略 CompleteWork 
  if(fiberNode.sibling){
    performUnitOfWork(fiberNode.sibling);
  }
}
```

最后我们来看一张图：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1749125520039-c69d99ae-f1d6-49d5-815b-5ed79def1678.png)

## 渲染器

Renderer 工作的阶段被称之为 commit 阶段。该阶段会将各种副作用 commit 到宿主环境的 UI 中。

相较于之前的 render 阶段可以被打断，commit 阶段一旦开始就会**同步**执行直到完成渲染工作。

整个渲染器渲染过程中可以分为三个子阶段：

- BeforeMutation 阶段
- Mutation 阶段
- Layout 阶段

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1749125520155-25040c98-3a36-4438-8bfa-aa0df90a8fa0.png)

题目：是否了解过 React 的整体渲染流程？里面主要有哪些阶段？

React 整体的渲染流程可以分为两大阶段，分别是 render 阶段和 commit 阶段。

render 阶段里面会经由调度器和协调器处理，此过程是在内存中运行，是异步可中断的。

commit 阶段会由渲染器进行处理，根据副作用进行 UI 的更新，此过程是同步不可中断的，否则会造成 UI 和数据显示不一致。

**调度器**

调度器的主要工作就是调度任务，让所有的任务有优先级的概念，这样的话紧急的任务可以优先执行。Scheduler 实际上在浏览器的 API 中是有原生实现的，这个 API 叫做 requestIdleCallback，但是由于兼容性问题，React 放弃了使用这个 API，而是自己实现了一套这样的机制，并且后期会把 Scheduler 这个包单独的进行发布，变成一个独立的包。这就意味 Scheduler 不仅仅是只能在 React 中使用，后面如果有其他的项目涉及到了任务调度的需求，都可以使用这个 Scheduler。

**协调器**

协调器是 Render 的第二阶段工作。该阶段会采用深度优先的原则遍历并且创建一个一个的 FiberNode，并将其串联在一起，在遍历时分为了“递”与“归”两个阶段，其中在“递”阶段会执行 beginWork 方法，该方法会根据传入的 FiberNode 创建下一级 FiberNode。而“归”阶段则会执行 CompleteWork 方法，做一些副作用的收集

**渲染器**

渲染器的工作主要就是将各种副作用（flags 表示）commit 到宿主环境的 UI 中。整个阶段可以分为三个子阶段，分别是 BeforeMutation 阶段、Mutation 阶段和 Layout 阶段。

# Fiber双缓冲

面试题：谈一谈你对 React 中 Fiber 的理解以及什么是 Fiber 双缓冲？

## 对 Fiber 的理解

实际上，我们可以从三个维度来理解 Fiber：

- 是一种架构，称之为 Fiber 架构
- 是一种数据类型
- 动态的工作单元

**是一种架构，称之为 Fiber 架构**

在 React v16之前，使用的是 Stack Reconciler，因此那个时候的 React 架构被称之为 Stack 架构。从 React v16 开始，重构了整个架构，引入了 Fiber，因此新的架构也被称之为 Fiber 架构，Stack Reconciler 也变成了 Fiber Reconciler。各个FiberNode之间通过链表的形式串联起来：

```
function FiberNode(tag, pendingProps, key, mode) {
  // ...

  // 周围的 Fiber Node 通过链表的形式进行关联
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  // ...
}
```

**是一种数据类型**

Fiber 本质上也是一个对象，是在之前 React 元素基础上的一种升级版本。每个 FiberNode 对象里面会包含 React 元素的类型、周围链接的FiberNode以及 DOM 相关信息：

```
function FiberNode(tag, pendingProps, key, mode) {
  // 类型
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null; // 映射真实 DOM

  // ...
}
```

**动态的工作单元**

在每个 FiberNode 中，保存了本次更新中该 React 元素变化的数据，还有就是要执行的工作（增、删、更新）以及副作用的信息：

```
function FiberNode(tag, pendingProps, key, mode) {
  // ...

  // 副作用相关
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;
    // 与调度优先级有关  
  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  // ...
}
```

为什么指向父 FiberNode 的字段叫做 return 而非 parent？

因为作为一个动态的工作单元，return 指代的是 FiberNode 执行完 completeWork 后返回的下一个 FiberNode，这里会有一个返回的动作，因此通过 return 来指代父 FiberNode

## Fiber 双缓冲

Fiber 架构中的双缓冲工作原理类似于显卡的工作原理。

显卡分为前缓冲区和后缓冲区。首先，前缓冲区会显示图像，之后，合成的新的图像会被写入到后缓冲区，一旦后缓冲区写入图像完毕，就会前后缓冲区进行一个互换，这种将数据保存在缓冲区再进行互换的技术，就被称之为双缓冲技术。

Fiber 架构同样用到了这个技术，在 Fiber 架构中，同时存在两颗 Fiber Tree，一颗是真实 UI 对应的 Fiber Tree，可以类比为显卡的前缓冲区，另外一颗是在内存中构建的 FiberTree，可以类比为显卡的后缓冲区。

在 React 源码中，很多方法都需要接收两颗 FiberTree：

```
function cloneChildFibers(current, workInProgress){
  // ...
}
```

**current 指的就是前缓冲区的 FiberNode，workInProgress 指的就是后缓冲区的 FiberNode**

两个 FiberNode 会通过 alternate 属性相互指向：

```
current.alternate = workInProgress;
workInProgress.alternate = current;
```

接下来我们从首次渲染（mount）和更新（update）这两个阶段来看一下 FiberTree 的形成以及双缓存机制：

**mount 阶段**

首先最顶层有一个 FiberNode，称之为 **FiberRootNode**，该 FiberNode 会有一些自己的任务：

- Current Fiber Tree 与 Wip Fiber Tree 之间的切换
- 应用中的过期时间
- 应用的任务调度信息

现在假设有这么一个结构：

```
<body>
  <div id="root"></div>
</body>
```

```jsx
function App(){
  const [num, add] = useState(0);
  return (
      <p onClick={() => add(num + 1)}>{num}</p>
  );
}
const rootElement = document.getElementById("root");
ReactDOM.createRoot(rootElement).render(<App />);
```

当执行 ReactDOM.createRoot 的时候，会创建如下的结构：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1749127296538-e7add6dd-5617-4f6e-b575-e963398efcdc.png)

此时会有一个 HostRootFiber（其实就是 root 根节点），FiberRootNode 通过 current 来指向 HostRootFiber。

接下来进入到 mount 流程，该流程会基于每个 React 元素以深度优先的原则依次生成 wip FiberNode，并且每一个 wipFiberNode 会连接起来，如下图所示：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1749127296591-3aab4466-2d9b-4865-ae36-4c5172c65e1b.png)

生成的 wip FiberTree 里面的每一个 FiberNode 会和 current FiberTree 里面的 FiberNode进行关联，关联的方式就是通过 alternate。但是目前 currentFiberTree里面只有一个 HostRootFiber，因此就只有这个 HostRootFiber 进行了 alternate 的关联。

当 **wip FiberTree生成**完毕后，也就意味着 **render 阶段**完毕了，此时 FiberRootNode就会被传递给 Renderer（渲染器），接下来就是进行渲染工作。渲染工作完毕后，**浏览器中就显示了对应的 UI**，此时 FiberRootNode.current 就会指向这颗 wip Fiber Tree，曾经的 wip Fiber Tree 它就会变成 current FiberTree，完成了双缓存的工作：（FiberRootNode 会重新指向，指向之前的后缓冲区生成的 Tree）

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1749127296569-700444d2-21fe-48dd-a5aa-29d669b2e832.png)

**update 阶段**

点击 p 元素，会触发更新，这一操作就会开启 update 流程，此时就会生成一颗新的 wip Fiber Tree，流程和之前是一样的

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1749127296669-0af3ed9e-3bd0-4c2c-958e-286c9197d725.png)

新的 wip Fiber Tree 里面的每一个 FiberNode 和 current Fiber Tree 的每一个 FiberNode 通过 alternate 属性进行关联。

当 wip Fiber Tree 生成完毕后，就会经历和之前一样的流程，FiberRootNode 会被传递给 Renderer 进行渲染，此时宿主环境所渲染出来的真实 UI 对应的就是左边 wip Fiber Tree 所对应的 DOM 结构，FiberRootNode.current 就会指向左边这棵树，右边的树就再次成为了新的 wip Fiber Tree

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1749127296822-8f575fa9-8af8-498f-909c-ff3a71404138.png)

这个就是 Fiber双缓存的工作原理。

另外值得一提的是，开发者是可以在一个页面创建多个应用的，比如：

```
ReactDOM.createRoot(rootElement1).render(<App1 />);
ReactDOM.createRoot(rootElement2).render(<App2 />);
ReactDOM.createRoot(rootElement3).render(<App3 />);
```

在上面的代码中，我们创建了 3 个应用，此时就会存在 3 个 FiberRootNode，以及对应最多 6 棵 Fiber Tree 树。

## 真题解析

题目：谈一谈你对 React 中 Fiber 的理解以及什么是 Fiber 双缓冲？

参考答案：

Fiber 可以从三个方面去理解：

- **FiberNode 作为一种架构**：在 React v15 以及之前的版本中，Reconceiler 采用的是递归的方式，因此被称之为 Stack Reconciler，到了 React v16 版本之后，引入了 Fiber，Reconceiler 也从 Stack Reconciler 变为了 Fiber Reconceiler，各个 FiberNode 之间通过链表的形式串联了起来。
- **FiberNode 作为一种数据类型**：Fiber 本质上也是一个对象，是之前虚拟 DOM 对象（React 元素，createElement 的返回值）的一种升级版本，每个 Fiber 对象里面会包含 React 元素的类型，周围链接的 FiberNode，DOM 相关信息。
- **FiberNode 作为动态的工作单元**：在每个 FiberNode 中，保存了“本次更新中该 React 元素变化的数据、要执行的工作（增、删、改、更新Ref、副作用等）”等信息。

所谓 Fiber 双缓冲树，指的是在内存中构建两颗树，并直接在内存中进行替换的技术。在 React 中使用 Wip Fiber Tree 和 Current Fiber Tree 这两颗树来实现更新的逻辑。Wip Fiber Tree 在内存中完成更新，而 Current Fiber Tree 是最终要渲染的树，两颗树通过 alternate 指针相互指向，这样在下一次渲染的时候，直接复用 Wip Fiber Tree 作为下一次的渲染树，而上一次的渲染树又作为新的 Wip Fiber Tree，这样可以加快 DOM 节点的替换与更新。

# MessageChannel

## 回顾事件循环

之前在学习事件循环的时候，大家看得更多的是下面这张图：

![](https://cdn.nlark.com/yuque/0/2025/gif/51701855/1752929554811-68ae76c4-a7dd-44cb-afb1-c1ba34105ccb.gif)

首先会执行同步代码，同步代码执行的时候，如果遇到异步代码，就会放到 Webapis 里面进行执行，当 webapis 执行完毕之后，会将结果放入到 task queue（任务队列），同步代码执行完毕后，就会从任务队列中会获取一个一个的任务进行执行。

如果将事件循环和浏览器渲染结合到一起，大致就是下面这张图：

![](https://xiejie-typora.oss-cn-chengdu.aliyuncs.com/2022-12-29-022329.gif "null")

总结一下：事件循环的机制就是每一次循环，会从任务队列中取一个任务来执行，如果还没有达到浏览器需要重新渲染的时间（16.66ms），那么就继续循环一次，从任务队列里面再取一个任务来执行，依此类推，直到浏览器需要重新渲染，这个时候就会执行重选渲染的任务，执行完毕后，回到之前的流程。

_requestAnimationFrame Api_ 是在每一次重新渲染之前执行，这个 _API_ 的出现，就是专门拿来做动画的。以前我们做动画，用的更多的是 setInterval 或者 setTimeout，但是这些 API 本意不是拿来做动画的。使用 _requestAnimationFrame Api_ 拿到做动画，最大的优点就是频率是和浏览器重新渲染的频率一致。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752929554969-2911e8c4-3009-46b1-b37c-6f541a04c0b0.png)

requestAnimationFrame 就不会存在这个问题，因为它是在渲染之前，保证了和浏览器渲染是同频

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752929554947-dc128dcd-00c1-4deb-a183-194bf64ed0bb.png)

## MessageChannel 以及为什么选择它

MessageChannel 接口本身是用来做消息通信的，允许我们创建一个消息通道，通过它的两个 MessagePort 来进行信息的发送和接收。

基本使用如下：

```jsx
<div>
  <input type="text" id="content" placeholder="请输入消息">
</div>
<div>
  <button id="btn1">给 port1 发消息</button>
  <button id="btn2">给 port2 发消息</button>
</div>
```

```jsx
const channel = new MessageChannel();
// 两个信息端口，这两个信息端口可以进行信息的通信
const port1 = channel.port1;
const port2 = channel.port2;
btn1.onclick = function(){
  // 给 port1 发消息
  // 那么这个信息就应该由 port2 来进行发送
  port2.postMessage(content.value);
}
// port1 需要监听发送给自己的消息
port1.onmessage = function(event){
  console.log(`port1 收到了来自 port2 的消息：${event.data}`);
}

btn2.onclick = function(){
  // 给 port2 发消息
  // 那么这个信息就应该由 port1 来进行发送
  port1.postMessage(content.value);
}
// port2 需要监听发送给自己的消息
port2.onmessage = function(event){
  console.log(`port2 收到了来自 port1 的消息：${event.data}`);
}
```

那么这个和 scheduler 有什么关系呢？

之前，我们有说过 scheduler 是用来调度任务，调度任务需要满足两个条件：

- JS 暂停，将主线程还给浏览器，让浏览器能够有序的重新渲染页面
- 暂停了的 JS（说明还没有执行完），需要再下一次接着来执行

那么这里自然而然就会想到事件循环，我们可以将没有执行完的 JS 放入到任务队列，下一次事件循环的时候再取出来执行。

那么，如何将没有执行完的任务放入到任务队列中呢？

那么这里就需要产生一个任务（宏任务），这里就可以使用 MessageChannel，因为 MessageChannel 能够产生宏任务。

**为什么不选择 setTimeout**

以前要创建一个宏任务，可以采用 setTimeout(fn, 0) 这种方式，但是 _react_ 团队没有采用这种方式。

这是因为 setTimeout 在嵌套层级超过 5 层，timeout（延时）如果小于 4ms，那么则会设置为 4ms。

这个你可以参阅 _HTML_ 规范：[https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#dom-settimeout](https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#dom-settimeout)

If nesting level is greater than 5, and timeout is less than 4, then set timeout to 4.

可以写一个例子来进行验证：

```jsx
let count = 0; // 计数器
let startTime = new Date(); // 获取当前的时间戳
console.log("start time:", 0, 0);
function fn(){
  setTimeout(function(){
    console.log("exec time:", ++count, new Date() - startTime);
    if(count === 50){
      return;
    }
    fn();
  },0)
}
fn();
```

正因为这个原因，所以 react 团队没有选择使用 setTimeout 来产生任务，因为 4ms 的时间的浪费还是不可忽视的。

**为什么没有选择 requestAnimationFrame**

这个也不合适，因为这个只能在重新渲染之前，才能够执行一次，而如果我们包装成一个任务，放入到任务队列中，那么只要没到重新渲染的时间，就可以一直从任务队列里面获取任务来执行。

而且 requestAnimationFrame 还会有一定的兼容性问题，safari 和 edge 浏览器是将 requestAnimationFrame 放到渲染之后执行的，chrome 和 firefox 是将 requestAnimationFrame 放到渲染之前执行的，所以这里存在不同的浏览器有不同的执行顺序的问题。

根据标准，应该是放在渲染之前。

**为什么没有选择包装成一个微任务？**

这是因为和微任务的执行机制有关系，微任务队列会在清空整个队列之后才会结束。那么微任务会在页面更新前一直执行，直到队列被清空，达不到将主线程还给浏览器的目的。

# Scheduler 调度器

源码位置：packages/scheduler/forks/Scheduler.js

调度器的任务：

1. 根据不同的优先级，优先调度优先级更高的

2. 可时间切片（如果当前任务要超过浏览器渲染一帧的时间，那么可归还主线程，不用等任务的执行）

内部的实现：

1. 怎么实现优先级的呢？

-> taskQueue中，根据不同有优先级赋予 不同的 timeout（值越小，优先级越高）

-> timerQueue中，根据传进来 不同的delay（值越小，优先级越高，小顶堆靠近根节点）

具体实现步骤：

任务开始执行的时间startTime = currentTime + delay 或者 currentTime 与 currentTime 比较，

如果 " > " 那么加入到 timerQueue（延时队列）

如果 " <= " 那么加入到 taskQueue（任务队列）

2. 执行任务的时候，会分三种情况 -> 都是看任务队列

2.1 taskQueue 中有任务 -> 调 requestHostCallback() 函数

2.2 taskQueue 没有任务，但 timerQueue 中有任务到了延时时间，此时会将改任务加入到 taskQueue 然后调用 requestHostCallback()

2.3 taskQueue 没有任务，并且 timerQueue 中没有任务到了延时时间，会调requestHostTimeout()

**taskQueue：根据 timeout 来区分优先级**

**timerQueue：根据 delay 来区分优先级**

**因为在小顶堆的排序是根据这个时间来的，**

**要不就是 currentTime + timeout，要不就是 currentTime + delay**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753103642554-ec2ed5c1-9c9b-4ab8-a706-eef1e68c5c0d.png)

# 小顶堆

在 Scheduler 中，有大量地方用到了最小堆的相关操作。

**堆**是在**完全二叉树**的基础上进行操作的，其根节点是**最大**或**最小值，**分别对应着大顶堆和小顶堆

在 react 中，使用小顶堆，从任务/延时队列中，取出一个延时时间最小的任务（也就是优先级高的任务）进行操纵。总之，可以取出当前优先级较高的任务。

用数组来描述二叉树：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753169571943-9bca7e97-606b-480a-80ec-50396e0c164d.png)

通过数组，我们可以非常方便的找到一个节点的所有亲属

- 父节点：Math.floor((当前节点的下标 - 1) / 2)
- 左分支节点：当前节点下标 * 2 + 1
- 右分支节点：当前节点下标 * 2 + 2

在 react 中导出了三个方法，

- peek：从队列中得到第一个任务
- pop：弹出队列的第一个任务
- push：将任务加入到队列中

还有一些在内部使用的方法，

- siftUp：向上调整
- siftDown：向下调整
- compare：比较

## peek

从最小堆中，获取一个优先级最高的出来执行（其实就是延时时间最短的）

```js
currentTask = peek(taskQueue);

export function peek<T: Node>(heap: Heap<T>): T | null {
  return heap.length === 0 ? null : heap[0];
}
```

## pop

与原生数组的方法不同，这个是获取队列的第一个元素，原生是获取最后一个元素（向下调整）

```js
export function pop<T: Node>(heap: Heap<T>): T | null {
  if (heap.length === 0) {
    return null;
  }
  // 获取第一个节点
  const first = heap[0];
  // 得到最后一个元素
  const last = heap.pop();
  // 如果堆中不止一个元素，那么将二叉树最后那个元素，
  // 移动到根元素的位置，在向下调整，保证最小堆
  if (last !== first) {
    heap[0] = last;
    siftDown(heap, last, 0);
  }
  return first;
}
```

## push

将节点（任务）推入到队列的末尾，因为要保持小顶堆的特性，需要向上调整

```js
export function push<T: Node>(heap: Heap<T>, node: T): void {
  const index = heap.length;
  heap.push(node);
  siftUp(heap, node, index);
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753157026179-840a1e7c-2bf3-4de8-8166-fa3cd0f8d3e5.png)

# Reconciler 协调器

这个是 react 渲染流程 render 阶段的第二个部分

它的目的是，创建 fiberNode 或给需要更新的 fiberNode 打上标记（更新、删除、插入）等，这一阶段不会更新节点，这个阶段是收集信息。交给 commit 阶段，让 renderer 根据 flags 更新。

内部大致分成两个阶段

- 递阶段：`beginWork`
- 归阶段：`completeWork`

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1749125520039-c69d99ae-f1d6-49d5-815b-5ed79def1678.png)

## beginWork

我们需要了解 beginWork 的流程即可

beginWork 的主要工作，根据传入的 FiberNode 创建下一级 FiberNode

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753328662934-42d78c77-ec49-47d8-ac5e-d5193e1eabd5.png)

如果是 update，接下来会判断 wipFiberNode 是否能够复用，如果不能够复用，那么 update 和 mount 的流程大体上一致：

- 根据 wip.tag 进行不同的分支处理
- 根据 reconcile 算法生成下一级的 FiberNode（diff 算法）

无法复用的 update 流程和 mount 流程大体一致，主要区别在于是否会生成带副作用标记 flags 的 FiberNode

关于 `workInProgress.tag`：

不同的 FiberNode，会有不同的 tag（有 28 种不同的 tag）

- HostComponent 代表的就是原生组件（div、span、p）
- FC 在 mount 的时候，对应的 tag 为 IndeterminateComponent，在 update 的时候就会进入 FunctionComponent
- HostText 表示的是文本元素

根据不同的 tag 处理完 FiberNode 之后，根据是mount 还是 update 会进入不同的方法：

- mount：mountChildFibers
- update：reconcileChildFibers

这两个方法实际上都是一个叫 ChildReconciler 方法的返回值：

```js
var reconcileChildFibers = ChildReconciler(true);
var mountChildFibers = ChildReconciler(false);

function ChildReconciler(shouldTrackSideEffects) {}
```

也就是说，在 ChildReconciler 方法内容，shouldTrackSideEffects 是一个布尔值

- false：不追踪副作用，不做 flags 标记，因为你是 mount 阶段
- true：要追踪副作用，做 flags 标记，因为是 update 阶段

在 ChildReconciler 方法内部，就会根据 shouldTrackSideEffects 做一些不同的处理：

```js
function placeChild(newFiber, lastPlacedIndex, newIndex){
  newFiber.index = newIndex;

  if(!shouldTrackSideEffects){
    // 说明是初始化
    // 说明不需要标记 Placement
    newFiber.flags |= Forked;
    return lastPlacedIndex
  }
  // ...
  // 说明是更新
  // 标记为 Placement
  newFiber.flags |= Placement;

}
```

可以看到，在 beginWork 方法内部，也会做一些 flags 标记（主要是在 update 阶段），这些 flags 标记主要和元素的位置有关系：

- 标记 ChildDeletion，这个是代表**删除**操作
- 标记 Placement，这是代表**插入或者移动**操作

请你阐述一下 beginWork 是什么？工作流程是什么？

beginWork 是 react 渲染流程 render 阶段中 reconciler 组件的“递阶段”，其主要目的是，传入一个 fiberNode 创建下一级 fiberNode.

工作流程：

首先， beginWork 先会判断是否存在 currentFiberTree，如果存在会进入 update 阶段，如果不存在会进入 mount 阶段。

update 阶段，会看是否能复用 workInProgress Fiber Tree 中的 props 和 content。如果能复用，那么就会进入**优化路径**的步骤（会调一个函数）。

如果不能复用，则流程与 mount 阶段差不多：

- 会根据`wip.tag`进入不同的分支处理
- 生成下一级 fiberNode

update 不复用阶段和 mount 阶段，**区别**是什么呢？

其实就是，如果是 update 阶段会生成带有 flags 的 fiber Node，而 mount 阶段不会带有 flags

## completeWork

这个阶段是 reconciler 阶段的“归阶段”

与 beginWork 类似，completeWork 也会根据 wip.tag 区分对待，流程上面主要包括两个步骤：

- 创建元素或者标记元素的更新
- flags 冒泡

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753349997774-585de373-8503-487a-9e55-ca8582d38140.png)

completeWork 是什么？工作流程是啥？

completeWork 是 reconciler 的第二个部分“归阶段”，它的主要功能是**收集副作用**。

工作流程：

由于也会根据`wip.tag`来区别对待，两个步骤：

- 创建或更新元素的标记
- flags 冒泡

根据`current !== null`来判断是`update`还是`mount`阶段

### mount 阶段

1. 调用`createInstance`创建 Dom 元素，
2. 在调用`appendAllChildren`将下一层的 dom 元素插入刚刚创建的 Dom 元素中。`appendAllChildren`先看有没有子节点，在看有没有兄弟元素。
3. 然后执行`finalizeInitialChildren` 初始化 dom 元素，

- styles，对应的方法为 setValueForStyles 方法
- innerHTML，对应 setInnerHTML 方法
- 文本类型 children，对应 setTextContent 方法
- 不会再在 DOM 中冒泡的事件，包括 cancel、close、invalid、load、scroll、toggle，对应的是 listenToNonDelegatedEvent 方法
- 其他属性，对应 setValueForProperty 方法

4. 执行 flags 冒泡

### update 阶段

这个阶段，就是完成属性更新的标记

updateHostComponent 的主要逻辑是在 diffProperties 方法里面，这个方法会包含两次遍历：

- 第一次遍历，标记**删除**了的属性
- 第二次遍历，标记**更新**了的属性

### flags 冒泡

我们知道，当整个 Reconciler 完成工作后，会得到一颗完整的 wipFiberTree，这颗 wipFiberTree 是由一颗一颗 FiberNode 组成的，这些 FiberNode 中有一些标记了 flags，有一些没有标记，现在就存在一个问题，我们如何高效的找到散落在这颗 wipFiberTree 中有 flag 标记的 FiberNode，那么此时就可以通过 flags 冒泡。

completeWork 是属于归的阶段，整体流程是自下往上，就非常适合用来收集副作用，收集的相关的代码如下：

```js
let subtreeFlags = NoFlags;

// 收集子 FiberNode 的子孙 FiberNode 中标记的 flags
subtreeFlags |= child.subtreeFlags;
// 收集子 FiberNode 中标记的 flags
subtreeFlags ｜= child.flags;
// 将收集到的所有 flags 附加到当前 FiberNode 的 subtreeFlags 上面
completeWork.subtreeFlags |= subtreeFlags;
```

这样的收集方式，有一个好处，在渲染阶段，通过任意一级的 FiberNode.subtreeFlags 都可以快速确定该 FiberNode 以及子树是否存在副作用从而判断是否需要执行和副作用相关的操作。

# diff 算法

react 中在 render 阶段会生成`workInprogress Fiber Node`，diff 算法就是来辅助创建`wipTree`

所以请注意：**diff 算法比较的是**`**currentFiberNode**`**和**`**react元素(createElement)**`

diff 算法是有性能消耗的，使用最前沿的算法，都需要`O(n^3)`的时间复杂度，这个效率说得上是非常恐怖了。

所以，react 团队为了提升效率，给 diff 算法设置了三个限制：

1. 对同级别的元素才进行 diff，如果跨级别了，react 将不考虑复用
2. 两个不同类型的元素会产生不同的树。比如元素从 div 变成了 p，那么 React 会直接销毁 div 以及子孙元素，新建 p 以及 p 对应的子孙元素
3. 可以通过设置 key 来暗示哪些元素可以保持稳定

**对于限制一：**

更新前：

```
<div>
  <div>
    <p key="one">one</p>
  </div>
</div>
```

更新后：

```
<div>
    <p key="one">one</p>
</div>
```

更新前后不是同一级别，那么 react 不会去尝试复用。

**对于限制二和三：**

更新前：

```
<div>
  <p key="one">one</p>
  <h3 key="two">two</h3>
</div>
```

更新后：

```
<div>
  <h3 key="two">two</h3>
  <p key="one">one</p>
</div>
```

如果没有 key 的话，react 会遵循规则二的情况，会直接删除 p 和 h3 元素，重新创建。

如果有 key 的话，那么此时 dom 元素是可以复用的，react 会交换顺序，不会重新创建 dom 元素。

---

对于**同级别**的元素，diff 的流程：

diff 算法根据生成的节点个数，分成了两种情况：

- 单节点 diff
- 多节点 diff

## 单节点 diff

意思就是，更新完后只会创建一个 fiberNode 节点

比较复用的过程如下：

- key 不同

- 无需判断 type，结果直接无法复用（如果有兄弟节点还需要判断兄弟节点）

- key 相同

- 如果没有设置 key，默认为 null，也是 key 相同的情况
- 判断 type 是否相同，

- type 相同，直接复用
- type 不同，无法复用，并且兄弟节点也会一并标记为删除

例如：

更新前

```
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</ul>
```

更新后

```
<ul>
  <p>1</p>
</ul>
```

没有设置 key，那么默认 key 是 null，那么，

1. key 相同 -> 继续判断 type
2. 从 li 变成了 p，type 不相同 -> 无法复用

---

上面的例子修改一下，

更新前

```
<ul>
  <li key="one">1</li>
  <li key="two">2</li>
  <li>3</li>
</ul>
```

更新后

```
<ul>
  <p key="two">1</p>
</ul>
```

首先，更新后只生成了一个节点，那么还是用 **单节点 diff 算法**

1. 比较 key：更新前第一个 key 为 one，与更新后 key 为 two 不相同，不可复用

**特别注意：**比较 **key 不相同**，只能代表**当前这个 fiberNode 不能复用**，还需要**遍历后续的兄弟节点**，比如这里第一个 li 不能复用，还需要判断第二个 li 能否复用

2. 比较第二个 key 相同，比较 type，更新前是 li，更新后是 p，**type 不相同**。不可复用，**后续兄弟节点都标记上删除**（这里与第一种情况不一样，不需要继续遍历后续兄弟节点）

## 多节点 diff

意思就是，更新后的 react 元素，有多个节点

多节点 diff 算法，需要进行两轮的遍历：

第一轮遍历：会尝试遍历所有节点，看能不能复用

第二轮遍历：处理上一轮遍历中没有处理的节点

### 第一轮遍历

- 如果新旧子节点的**key 和 type 都相同**，说明可以复用
- 如果新旧子节点的 **key 相同**，但是 **type 不相同**，这个时候就会根据 ReactElement 来生成一个全新的 fiber，旧的 fiber 被放入到 deletions 数组里面，回头统一删除。但是注意，此时遍历并不会终止
- 如果新旧子节点的 **key 不相同**，结束遍历

更新前

```
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

更新后

```
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="e">e</div>
  <div key="d">d</div>
</div>
```

首先遍历到 div.key.a，发现该 FiberNode 能够复用

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753429332709-c40e7631-a58b-43ec-840c-7aceef029a69.png)

继续往后面走，发现 div.key.b 也能够复用

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753429332702-8f8fe87e-3db5-4f10-a44a-2beac1ea9280.png)

接下来继续往后面走，div.key.e，这个时候发现 key 不一样，因此第一轮遍历就结束了

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753429332784-a8b69790-b8f2-4a1f-b6ec-0637f8afc5c6.png)

### 第二轮遍历

如果第一轮遍历被提前终止了，那么意味着有新的 React 元素或者旧的 FiberNode 没有遍历完，此时就会采用第二轮遍历

第二轮遍历会处理这么三种情况：

- **只剩下旧子节点**：将旧的子节点添加到 deletions 数组里面直接删除掉（**删除**的情况）
- **只剩下新的 JSX 元素**：根据 ReactElement 元素来创建 FiberNode 节点（**新增**的情况）
- **新旧子节点都有剩余**：会将剩余的 FiberNode 节点放入一个 **map 里面**，遍历剩余的新的 JSX 元素，然后从 map 中去寻找能够复用的 FiberNode 节点，如果能够找到，就拿来复用。（**移动**的情况）

如果不能找到，那就新增。如果剩余的 JSX 元素都遍历完了，map 结构中还有剩余的 Fiber 节点，就将这些 Fiber 节点添加到 deletions 数组里面，之后统一做删除操作。

**只剩下旧子节点**

更新前

```
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

更新后

```
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  // 复用前两个div，后面两个删除
</div>
```

遍历前面两个节点，发现能够复用，此时就会复用前面的节点，对于 React 元素来讲，遍历完前面两个就已经遍历结束了，因此剩下的FiberNode就会被放入到 deletions 数组里面，之后统一进行删除

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753429710575-2e31e430-cd4b-4095-aaf3-a9271b3d8fee.png)

**只剩下新的 JSX 元素**

更新前

```
<div>
  <div key="a">a</div>
  <div key="b">b</div>
</div>
```

更新后

```
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

根据新的 React 元素新增对应的 FiberNode 即可。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753429710726-7c965fd6-45ad-4d65-8ec4-fe80724d225c.png)

**新旧子节点都有剩余**

更新前

```
<div>
  <div key="a">a</div>
  <div key="b">b</div>
  <div key="c">c</div>
  <div key="d">d</div>
</div>
```

更新后

```
<div>
  <div key="a">a</div>
  <div key="c">b</div>
  <div key="b">b</div>
  <div key="e">e</div>
</div>
```

首先会将剩余的旧的 FiberNode 放入到一个 map 里面

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753429710681-6bb4cd35-dd9b-4b53-9f8f-c84d504fa5f9.png)

接下来会继续去遍历剩下的 JSX 对象数组，遍历的同时，从 map 里面去找有没有能够复用

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753429710619-3d52e97f-12e1-456a-9775-ef267bd5f329.png)

如果在 map 里面没有找到，那就会新增这个 FiberNode，如果整个 JSX 对象数组遍历完成后，map 里面还有剩余的 FiberNode，说明这些 FiberNode 是无法进行复用，直接放入到 deletions 数组里面，后期统一进行删除。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753429710661-ae883ea9-bf4f-4d53-923a-53100dacce27.png)

为什么 react 不采用双端 diff？

其实主要原因是，因为双端 diff 需要从后往前遍历节点。但是在 fiberNode 中，兄弟节点中没有从后往前的指向，只能从前往后遍历（只有前一个节点通过 sibling 指向它的兄弟节点，其兄弟节点不能找到它的前一个兄弟节点）

# commit 工作流程

面试题：commit 阶段的工作流程是怎样的？此阶段可以分为哪些模块？每个模块在做什么？

整个 React 的工作流程可以分为两大阶段：

- Render 阶段

- Schedule
- Reconcile

- Commit 阶段

注意，Render 阶段的行为是在内存中运行的，这意味着可能被打断，也可以被打断，而 commit 阶段则是一旦开始就会**同步**执行直到完成。

commit 阶段整体可以分为 3 个子阶段：

- BeforeMutation 阶段
- Mutation 阶段
- Layout 阶段

整体流程图如下：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753522499216-ed089c2b-5350-4ad3-ac69-870ee75f21e7.png)

每个阶段，又分为三个子阶段：

- commitXXXEffects
- commitXXXEffects_begin
- commitXXXEffects_complete

所分成的这三个子阶段，是有一些共同的事情要做的

**commitXXXEffects**

该函数是每个子阶段的入口函数，finishedWork 会作为 firstChild 参数传入进去，相关代码如下：

```js
function commitXXXEffects(root, firstChild){
  nextEffect = firstChild;
  // 省略标记全局变量
  commitXXXEffects_begin();
  // 省略重置全局变量
}
```

因此在该函数中，主要的工作就是将 firstChild 赋值给全局变量 nextEffect，然后执行 commitXXXEffects_begin

**commitXXXEffects_begin**

向下遍历 FiberNode。遍历的时候会遍历直到第一个满足如下条件之一的 FiberNode：

- 当前的 FiberNode 的子 FiberNode 不包含该子阶段对应的 flags
- 当前的 FiberNode 不存在子 FiberNode

接下来会对目标 FiberNode 执行 commitXXXEffects_complete 方法，commitXXXEffects_begin 相关代码如下：

```js
function commitXXXEffects_begin(){
  while(nextEffect !== null) {
    let fiber = nextEffect;
    let child = fiber.child;
    
    // 省略该子阶段的一些特有操作
    
    if(fiber.subtreeFlags !== NoFlags && child !== null){
      // 继续向下遍历
      nextEffect = child;
    } else {
      commitXXXEffects_complete();
    }
  }
}
```

**commitXXXEffects_complete**

该方法主要就是针对 flags 做具体的操作了，主要包含以下三个步骤：

- 对当前 FiberNode 执行 flags 对应的操作，也就是执行 commitXXXEffectsOnFiber
- 如果当前 FiberNode 存在兄弟 FiberNode，则对兄弟 FiberNode 执行 commitXXXEffects_begin
- 如果不存在兄弟 FiberNode，则对父 FiberNode 执行 commitXXXEffects_complete

相关代码如下：

```js
function commitXXXEffects_complete(root){
  while(nextEffect !== null){
    let fiber = nextEffect;
    
    try{
      commitXXXEffectsOnFiber(fiber, root);
    } catch(error){
      // 错误处理
    }
    
    let sibling = fiber.sibling;
    
    if(sibling !== null){
      // ...
      nextEffect = sibling;
      return
    }
    
    nextEffect = fiber.return;
  }
}
```

总结一下，每个子阶段都会以 DFS 的原则来进行遍历，最终会在 commitXXXEffectsOnFiber 中针对不同的 flags 做出不同的处理。

## BeforeMutation 阶段

BeforeMutation 阶段的主要工作发生在 commitBeforeMutationEffects_complete 中的 commitBeforeMutationEffectsOnFiber 方法，相关代码如下：

```js
function commitBeforeMutationEffectsOnFiber(finishedWork){
  const current = finishedWork.alternate;
  const flags = finishedWork.falgs;
  
  //...
  // Snapshot 表示 ClassComponent 存在更新，且定义了 getSnapsshotBeforeUpdate 方法
  if(flags & Snapshot !== NoFlags) {
    switch(finishedWork.tag){
      case ClassComponent: {
        if(current !== null){
          const prevProps = current.memoizedProps;
          const prevState = current.memoizedState;
          const instance = finishedWork.stateNode;
          
          // 执行 getSnapsshotBeforeUpdate
          const snapshot = instance.getSnapsshotBeforeUpdate(
              finishedWork.elementType === finishedWork.type ? 
            prevProps : resolveDefaultProps(finishedWork.type, prevProps),
            prevState
          )
        }
        break;
      }
      case HostRoot: {
        // 清空 HostRoot 挂载的内容，方便 Mutation 阶段渲染
        if(supportsMutation){
          const root = finishedWork.stateNode;
          clearCOntainer(root.containerInfo);
        }
        break;
      }
    }
  }
}
```

上面代码的整个过程中，主要是处理如下两种类型的 FiberNode：

- ClassComponent：执行 getSnapsshotBeforeUpdate 方法
- HostRoot：清空 HostRoot 挂载的内容，方便 Mutation 阶段进行渲染

## Mutation 阶段

对于 HostComponent，Mutation 阶段的主要工作就是对 DOM 元素及进行增、删、改

### 删除 DOM 元素

删除 DOM 元素相关代码如下：

```js
function commitMutationEffects_begin(root){
  while(nextEffect !== null){
    const fiber = nextEffect;
    // 删除 DOM 元素
    const deletions = fiber.deletions;
    
    if(deletions !== null){
      for(let i=0;i<deletions.length;i++){
        const childToDelete = deletions[i];
        try{
          commitDeletion(root, childToDelete, fiber);
        } catch(error){
          // 省略错误处理
        }
      }
    }
    
    const child = fiber.child;
    if((fiber.subtreeFlags & MutationMask) !== NoFlags && child !== null){
      nextEffect = child;
    } else {
      commitMutationEffects_complete(root);
    }
  }
}
```

删除 DOM 元素的操作发生在 commitMutationEffects_begin 方法中，首先会拿到 deletions 数组，之后遍历该数组进行删除操作，对应删除 DOM 元素的方法为 commitDeletion。

commitDeletion 方法内部的完整逻辑实际上是比较复杂的，原因是因为在删除一个 DOM 元素的时候，不是说删除就直接删除，还需要考虑以下的一些因素：

- 其子树中所有组件的 unmount 逻辑
- 其子树中所有 ref 属性的卸载操作
- 其子树中所有 Effect 相关 Hook 的 destory 回调的执行

假设有如下的代码：

```js
<div>
    <SomeClassComponent/>
  <div ref={divRef}>
      <SomeFunctionComponent/>
  </div>
</div>
```

当你删除最外层的 div 这个 DOM 元素时，需要考虑：

- 执行 SomeClassComponent 类组件对应的 componentWillUnmount 方法
- 执行 SomeFunctionComponent 函数组件中的 useEffect、useLayoutEffect 这些 hook 中的 destory 方法
- divRef 的卸载操作

整个删除操作是以 DFS 的顺序，遍历子树的每个 FiberNode，执行对应的操作。

### 插入、移动 DOM 元素

上面的删除操作是在 commitMutationEffects_begin 方法里面执行的，而插入和移动 DOM 元素则是在 commitMutationEffects_complete 方法里面的 commitMutationEffectsOnFiber 方法里面执行的，相关代码如下：

```js
function commitMutationEffectsOnFiber(finishedWork, root){
  const flags = finishedWork.flags;

  // ...
  
  const primaryFlags = flags & (Placement | Update | Hydrating);
  
  outer: switch(primaryFlags){
    case Placement:{
      // 执行 Placement 对应操作
      commitPlacement(finishedWork);
      // 执行完 Placement 对应操作后，移除 Placement flag
      finishedWork.falgs &= ~Placement;
      break;
    }
    case PlacementAndUpdate:{
      // 执行 Placement 对应操作
      commitPlacement(finishedWork);
      // 执行完 Placement 对应操作后，移除 Placement flag
      finishedWork.falgs &= ~Placement;
      
      // 执行 Update 对应操作
      const current = finishedWork.alternate;
      commitWork(current, finishedWork);
      break;
    }
      
    // ...
  }
  

}
```

可以看出， Placement flag 对应的操作方法为 commitPlacement，代码如下：

```js
function commitPlacement(finishedWork){
  // 获取 Host 类型的祖先 FiberNode
  const parentFiber = getHostParentFiber(finishedWork);
  
  // 省略根据 parentFiber 获取对应 DOM 元素的逻辑
  
  let parent;
  
  // 目标 DOM 元素会插入至 before 左边
  const before = getHostSibling(finishedWork);
  
  // 省略分支逻辑
  // 执行插入或移动操作
  insertOrAppendPlacementNode(finishedWork, before, parent);
}
```

整个 commitPlacement 方法的执行流程可以分为三个步骤：

- 从当前 FiberNode 向上遍历，获取第一个类型为 HostComponent、HostRoot、HostPortal 三者之一的祖先 FiberNode，其对应的 DOM 元素是执行 DOM 操作的目标元素的父级 DOM 元素
- 获取用于执行 parentNode.insertBefore(child, before) 方法的 “before 对应的 DOM 元素”
- 执行 parentNode.insertBefore 方法（存在 before）或者 parentNode.appendChild 方法（不存在 before）

对于“还没有插入的DOM元素”（对应的就是 mount 场景），insertBefore 会将目标 DOM 元素插入到 before 之前，appendChild 会将目标DOM元素作为父DOM元素的最后一个子元素插入

对于“UI中已经存在的 DOM 元素”（对应 update 场景），insertBefore 会将目标 DOM 元素移动到 before 之前，appendChild 会将目标 DOM 元素移动到同级最后。

因此这也是为什么在 React 中，插入和移动所对应的 flag 都是 Placement flag 的原因。（可能面试的时候会被问到）

### 更新 DOM 元素

更新 DOM 元素，一个最主要的工作就是更新对应的属性，执行的方法为 commitWork，相关代码如下：

```js
function commitWork(current, finishedWork){
  switch(finishedWork.tag){
    // 省略其他类型处理逻辑
    case HostComponent:{
      const instance = finishedWork.stateNode;
      if(instance != null){
        const newProps = finishedWork.memoizedProps;
        const oldProps = current !== null ? current.memoizedProps : newProps;
        const type = finishedWork.type;
        
        const updatePayload = finishedWork.updateQueue;
        finishedWork.updateQueue = null;
        if(updatePayload !== null){
          // 存在变化的属性
          commitUpdate(instance, updatePayload, type, oldProps, newProps, finishedWork);
        }
      }
      return;
    }
  }
}
```

之前有讲过，变化的属性会以 key、value 相邻的形式保存在 FiberNode.updateQueue ，最终在 FiberNode.updateQueue 里面所保存的要变化的属性就会在一个名为 updateDOMProperties 方法被遍历然后进行处理，这里的处理主要是处理如下的四种数据：

- style 属性变化
- innerHTML
- 直接文本节点变化
- 其他元素属性

相关代码如下：

```js
function updateDOMProperties(domElement, updatePayload, wasCustomComponentTag, isCustomComponentTag){
  for(let i=0;i< updatePayload.length; i+=2){
    const propKey = updatePayload[i];
    const propValue = updatePayload[i+1];
    if(propKey === STYLE){
      // 处理 style
      setValueForStyle(domElement, propValue);
    } else if(propKey === DANGEROUSLY_SET_INNER_HTML){
      // 处理 innerHTML
      setInnerHTML(domElement, propValue);
    } else if(propsKey === CHILDREN){
      // 处理直接的文本节点
      setTextContent(domElement, propValue);
    } else {
      // 处理其他元素
      setValueForProperty(domElement, propKey, propValue, isCustomComponentTag);
    }
  }
}
```

当 Mutation 阶段的主要工作完成后，在进入 Layout 阶段之前，会执行如下的代码来完成 FiberTree 的切换：

```
root.current = finishedWork;
```

## Layout 阶段

有关 DOM 元素的操作，在 Mutation 阶段已经结束了。

在 Layout 阶段，主要的工作集中在 commitLayoutEffectsOnFiber 方法中，在该方法内部，会针对不同类型的 FiberNode 执行不同的操作：

- 对于 ClassComponent：该阶段会执行 componentDidMount/Update 方法
- 对于 FunctionComponent：该阶段会执行 useLayoutEffect 的回调函数

## 真题解答

题目：commit 阶段的工作流程是怎样的？此阶段可以分为哪些模块？每个模块在做什么？

参考答案：

整个 commit 可以分为三个子阶段

- BeforeMutation 阶段
- Mutation 阶段
- Layout 阶段

每个子阶段又可以分为 commitXXXEffects、commitXXXEffects_beigin 和 commitXXXEffects_complete

其中 commitXXXEffects_beigin 主要是在做遍历节点的操作，commitXXXEffects_complete 主要是在处理副作用

BeforeMutation 阶段整个过程主要处理如下两种类型的 FiberNode：

- ClassComponent，执行 getSnapsshotBeforeUpdate 方法
- HostRoot，清空 HostRoot 挂载的内容，方便 Mutation 阶段渲染

对于 HostComponent，Mutation 阶段的工作主要是进行 DOM 元素的增、删、改。当 Mutation 阶段的主要工作完成后，在进入 Layout 阶段之前，会执行如下的代码完成 Fiber Tree 的切换。

Layout 阶段会对遍历到的每个 FiberNode 执行 commitLayoutEffectOnFiber，根据 FiberNode 的不同，执行不同的操作，例如：

- 对于 ClassComponent，该阶段执行 componentDidMount/Update 方法
- 对于 FunctionComponent，该阶段执行 useLayoutEffect callback 方法

# lane 模型

这个是 react 内部维护的一个颗粒化更高的处理优先级的模型，跟 Scheduler 差不多，但是更加精细化。

那为什么有了 Scheduler 还需要设计这个 lane 模型呢？

> 主要是 react 在后期想将 Scheduler 独立发包，与 react 耦合度没有那么高，需要考虑通用性

虽然两套优先级不互通，但是，这两套优先级模型可以进行转化的，

**在 Scheduler 中，有五类优先级：**

```js
export const NoPriority = 0;
export const ImmediatePriority = 1;
export const UserBlockingPriority = 2;
export const NormalPriority = 3;
export const LowPriority = 4;
export const IdlePriority = 5;
```

在 Scheduler 中，会根据优先级赋予不同的 timeout，从而来区分优先级

**在 React 内部中，在四类优先级：**

```js
export const DiscreteEventPriority: EventPriority = SyncLane;
export const ContinuousEventPriority: EventPriority = InputContinuousLane;
export const DefaultEventPriority: EventPriority = DefaultLane;
export const IdleEventPriority: EventPriority = IdleLane;
```

React 中，交互产生的回调函数 update 也有不同的优先级，因此，优先级与事件有关。**React 优先级也被称为****EventPriority**，各种优先级的含义如下：

- DiscreteEventPriority：对应**离散**事件优先级，例如 click、input、focus、blur、touchstart 等事件都是离散触发的
- ContinuousEventPriority：对应**连续**事件的优先级，例如 drag、mousemove、scroll、touchmove 等事件都是连续触发的
- DefaultEventPriority：对应**默认**的优先级，例如通过**计时器**周期性触发更新，这种情况下产生的 update 不属于交互产生 update，所以优先级是默认的优先级
- IdleEventPriority：对应**空闲**情况的优先级

**不同的 EventPriority 会对应不同的 lane 模型**

两套模型的相互转化：

**React 优先级转为 Scheduler 的优先级**

会经历两次转化，

- 将 lanes 转化成 EventPriority

```js
export function lanesToEventPriority(lanes: Lanes): EventPriority {
  // getHighestPriorityLane 方法用于分离出优先级最高的 lane
  const lane = getHighestPriorityLane(lanes);
  if (!isHigherEventPriority(DiscreteEventPriority, lane)) {
    return DiscreteEventPriority;
  }
  if (!isHigherEventPriority(ContinuousEventPriority, lane)) {
    return ContinuousEventPriority;
  }
  if (includesNonIdleWork(lane)) {
    return DefaultEventPriority;
  }
  return IdleEventPriority;
}
```

- 在将 EventPriority 转化成 Scheduler

```js
// ...
let schedulerPriorityLevel;
switch (lanesToEventPriority(nextLanes)) {
  case DiscreteEventPriority:
    schedulerPriorityLevel = ImmediateSchedulerPriority;
    break;
  case ContinuousEventPriority:
    schedulerPriorityLevel = UserBlockingSchedulerPriority;
    break;
  case DefaultEventPriority:
    schedulerPriorityLevel = NormalSchedulerPriority;
    break;
  case IdleEventPriority:
    schedulerPriorityLevel = IdleSchedulerPriority;
    break;
  default:
    schedulerPriorityLevel = NormalSchedulerPriority;
    break;
}
// ...
```

举个例子，

有一个点击事件，在 onclick 事件中有回调函数来触发更新（update），这个更新的优先级，一开始是`DiscreteEventPriority`，经过转化后在 Scheduler 中是`ImmediateSchedulerPriority`

**Scheduler 的优先级转为 React 的优先级**

```js
const schedulerPriority = getCurrentSchedulerPriorityLevel();

// 根据不同的 schedulerPriority 转化成不同的 EventPriority
switch (schedulerPriority) {
  case ImmediateSchedulerPriority:
    return DiscreteEventPriority;
  case UserBlockingSchedulerPriority:
    return ContinuousEventPriority;
  case NormalSchedulerPriority:
  case LowSchedulerPriority:
    return DefaultEventPriority;
  case IdleSchedulerPriority:
    return IdleEventPriority;
  default:
    return DefaultEventPriority;
}
```

这里会涉及到一个问题，在同一时间可能存在很多的更新，究竟先去更新哪一个？

- 从众多的有优先级的 update 中选出一个优先级最高的
- 表达批的概念（我的理解是，一批是哪些任务这次更新，哪些任务下次更新）

有两种模型的迭代：

- 基于 expirationTime 的算法（之前的）
- 基于 lane 的算法（最新的）

## expirationTime 算法

这个其实处理优先级的想法更 Scheduler 是差不多的，算 timeout 加上 currentTime = expriationTime 给小顶堆进行排序，取出优先级最高的（时间最短的）

这种算法，理解起来很简单。但是它将 ”优先级“ 和 ”批“ 的概念耦合在一起了，在expriationTime 中处理 ”批“是这样的：

```js
const isUpdateIncludedInBatch = priorityOfUpdate >= priorityOfBatch;
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753525317752-6a1c8b3b-236e-4f95-a32e-ef24a40ca2ca.png)

意思就是，如果当前更新的优先级大于 priorityOfBatch 那么就会放到同一批次。不能随意分配 ”批“

但是此时就会存在一个问题，如何将**某一范围的某几个优先级**划为同一批？

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1753525421688-02d0bbf4-1131-4b0b-8214-07c0c55aff40.png)

就因为这些原因，react 引进了 lane 算法

## lane 算法

处理优先级：

```js
export const SyncLane: Lane = /*                   */ 0b0000000000000000000000000000001;
export const InputContinuousLane: Lane = /*        */ 0b0000000000000000000000000000100;
export const DefaultLane: Lane = /*                */ 0b0000000000000000000000000010000;
export const IdleLane: Lane = /*                   */ 0b0100000000000000000000000000000;
export const OffscreenLane: Lane = /*              */ 0b1000000000000000000000000000000;
```

位置越低，优先级越高，

这里，SyncLane 优先级最高，OffscreenLane 优先级最低

处理”批“：

lane 中使用二进制来描述优先级，使用位运算，可以很方便的**增删改查**

```js
// 要使用的批
let batch = 0;
// laneA 和 laneB。是不相邻的优先级
const laneA = 0b0000000000000000000000001000000;
const laneB = 0b0000000000000000000000000000001;
// 将 laneA 纳入批中
batch |= laneA;
// 将 laneB 纳入批中
batch |= laneB;
```

可以随意将哪些优先级，加入到一批中

React 中的 lane 模型是什么？

lane 模型其实是 React 内部维护的更加颗粒化的处理优先级的模型，在内部使用 32 bit 的二进制来描述优先级。越低位的优先级更高。由于使用的是二进制，位运算可以很方便的 ”增删改查“，可以将不同的优先级加入到同一 ”批“来处理。

在 React 中其实是维护了一套它的优先级（被称为 EventPriority，事件优先级其实被赋予了不同的 lane），区别于 Scheduler 的优先级。但是这两套优先级可以互相转化。

**React 转 Scheduler**：

1. React 转 lane
2. lane 转 Scheduler

**Scheduler 转 React**：直接调方法，该函数可以根据不同的 Scheduler 优先级转化成 React 优先级

## 在代码层面，react 组件第一次渲染，会发生什么？

1. **JSX 转换为 React 元素**（虚拟 DOM）

2. JSX 是不能被浏览器解析的，需要通过 Babel 等工具编译成为`React.createElement`函数调用，生成 React 元素。

3. **启动渲染**：ReactDOM.render 或 createRoot.render（react18）这个调用会触发 React 底层的【初始化渲染流程】，创建**根节点并启动协调器**
4. **初始化根节点**（FieberRoot 与 rootFiber）

5. FiberRoot：关联真实容器 #root（记录真实的 Fiber 树）
6. rootFiber：根节点<App /> 对应的 fiber 节点，是 Fiber 树的逻辑根

7. **调度器**（Scheduler）

8. 首次渲染属于【同步优先级】任务，会直接将任务交给协调器

9. **协调器**（Reconciler）diff 算法，对比 fiber 树（首次渲染不需要比对），生成 Fiber 树

10. 从 rootFiber 开始递归
11. 处理组件，生成子 Fiber 节点

12. 函数组件，执行函数，获取返回的 React 元素
13. 类组件，实例化组件，调用 constructor，再调用 render 方法获取返回的 React 元素
14. 原生 DOM 标签（div），直接创建对应的 fiber 节点

15. 设置 Fiber 节点关系，child、sibling、return

16. **提交阶段**（commit）：生成真实 DOM 并挂载，协调器 Fiber 树构建后，会将结果交给渲染器（renderer，如 ReactDOM 负责浏览器环境），这个阶段是不能中断的，操作真实 DOM

17. **Before Mutation**： 执行 DOM 变更前的副作用
18. **Mutation**：创建并插入真实 DOM

19. 遍历 Fiber 树，对于【原生 DOM 类型 Fiber 节点，如果 type 不是原生标签，是组件，会直接遍历 child 指针指向的元素】，直接调用 document.createElement 创建真实 DOM
20. 设置 DOM 属性，通过 setAttribute 或直接赋值
21. 挂载自节点，按照 child 的关系，将子 DOM 元素插入到父节点
22. 最终将根节点挂载到 #root

23. **Layout** 阶段：执行 DOM 变更后的副作用

24. 类组件，调用 componentDidMount 生命周期
25. 函数组件，执行 useEffect 回调
26. 处理 ref
