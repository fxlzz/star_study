React 18 意味着 React 从“单线程阻塞”转变为“可中断并发”

开启并发模式：
```js
// main.js
import { crateRoot } from "react-dom/client";

ReactDOM.render(<App />, root) ---> createRoot(root).render(<App />)
```

# 自动批处理


