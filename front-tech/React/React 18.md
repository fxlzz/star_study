React 18 意味着 React 从“单线程阻塞”转变为“可中断并发”

开启并发模式：
```js
// main.js
import { crateRoot } from "react-dom/client";

ReactDOM.render(<App />, root) ---> createRoot(root).render(<App />)
```

# 自动批处理
### React 17

只有 React 事件中才会批处理：
```js
SetA (1);  
SetB (2);  
// ✅ 批处理
```

但在异步中不会：
```js
SetTimeout (() => {  
  SetA (1);  
  SetB (2);  
});  
// ❌ 不会批处理（会渲染两次）
```

---
### React 18
所有场景都自动批处理：
```js
SetTimeout (() => {  
  SetA (1);  
  SetB (2);  
});  
// ✅ 只渲染一次
```

# UseTransition 过渡更新
[useTransition – React 中文文档](https://zh-hans.react.dev/reference/react/useTransition)

什么时候用？
>  当某次更新“很耗性能”，但不会影响当前交互结果时
>  例如：
>  - 搜索 + 大列表（输入框紧急、列表非紧急）
>  - 切换路由/tab 栏（点击事件紧急、页面内容非紧急）

语法：
```js
const [isPending, startTransition] = useTransistion();
```
+ `isPeding`: 正在处理的 transition 的状态
+ `void startTransition(xxxAction)`: 使用此方法将更新标记为 transition *非阻塞特性*

传递给 `startTransition` 的函数被称为“Action”，可以在 Action 中更新状态和执行副作用——会在后台执行，不会阻塞用户交互
一个 Transition 可以包含多个 Action

对于 `isPending`：
会在 `startTransition` 函数调用后切换为 true，在所有 Action 完成且呈现给用户前一直保持为 true

可通过 `useOptimistic` 即使反馈 transition 的状态

言而总之，`useTransition` 用于标记“非紧急更新”，让 React 优先保证用户交互

# UseDeferredValue 延迟值
让某个值“慢一点更新”

语法：
```js
const deferredValue = useDeferredValue(value);
```

使用场景：
+ 搜索框
+ 大列表过滤

# Suspense
允许子组件完成加载前展示“降级”方案
可配合 `React.lazy` 懒加载

```jsx
<Suspense fallback={<Loading />}>  
<SomeComponent />  
</Suspense>
```




