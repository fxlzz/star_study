react 的状态管理库，现在比较流行

类似的，还有 redux

那么为什么要使用这个库呢？因为相对于其他的状态库，这个库比较**轻量，上手简单**

例子：

## count 加减的管理

1. 创建一个 store

```jsx
import {create} from 'zustand'

// 创建一个store
const useCountStore = create((set) => ({
    count: 0,
    add: () => set((state) => ({count: state.count + 1})),
    minus: () => set((state) => ({count: state.count - 1}))
}))

export default useCountStore
```

2. 在组件中使用

```jsx
import useCountStore from "../store/countStore";

const App = () => {
  // 这里真的相当简单，相比于其他的状态管理库
  const { count, add, minus } = useCountStore();

  return (
    <>
      <button onClick={add}>+1</button>
      <button onClick={minus}>-1</button>
      <div>{count}</div>
    </>
  );
};

export default App;
```

  

## 异步操作

简单的一个例子，没有实际意义

1. 创建一个 store

```jsx
import {create} from 'zustand'
import {getVals} from '../api'

const useNumStore = create((set) => ({
    list: [],
    init: async (vals) => {
        const data = await getVals(vals)
        set({list: data.values})
    } 
}))

export default useNumStore
```

2. 准备一个测试的 api

```jsx
export const getVals = async () => {
  // 这里的后端服务，用node express搭建的
  const {data} =  await fetch('http://localhost:81/').then((resq) => resq.json())
  return data;
}
```

3. 在组件中使用

```jsx
import { useEffect } from "react"
import useNumStore from "../store/numStore"

const App = () => {
  // 确实操作起来，相当简单，异步操作也很简单
  const {list, init} = useNumStore()
  useEffect(() => {
    init()
  }, [])

  return (
    <>
      <ul>{list.length > 0 && list.map((v, i) => (<li key={i}>{v}</li>))}</ul>
    </>
  )
}

export default App
```

## shallow

主要是用来性能优化的，

因为，Zustand 默认是用严格等于来判断两个状态值是否相等

- 相等 -> 不会重新渲染
- 不想等 -> 会造成重新渲染

那这里，其实就有一个问题：对象的比较（属性值不变的情况下），严格相等还是会判断为 false，但事实上，对象里面的内容没有变化。会导致，大量重复渲染，浪费性能。

shallow 比较是浅层比较，其实就是重写了 eaqul 函数（只比较值）

```jsx
import { useStore, shallow } from 'zustand';

// 定义store
const useMyStore = create((set) => ({
  user: { name: '张三', age: 20 },
  // ...其他状态
}));

// 组件中使用shallow优化
function UserInfo() {

  const { name, age } = useStore(
    (state) => ({ name: state.user.name, age: state.user.age }),
    shallow // 关键：添加shallow作为第二个参数
  );

  return <div>{name} - {age}</div>;
}
```