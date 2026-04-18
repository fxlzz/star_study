# 学习指南
> 如果有条件可以用 AI 学习，让 AI 用苏格拉底式提问法，去学习

参考资料：
[react query](https://tanstack.com/query/latest/docs/framework/react/overview)
[deepWiki](https://deepwiki.com/TanStack/query)
[source](https://github1s.com/TanStack/query)

# 面临的问题
开发中，最终要的就是处理数据的流向，一般，有两种情况
+ 从接口获取数据，展示到页面上 -> 纯展示【useQuery】
+ 通过交互，修改数据，更新服务器上的数据 -> 变更【useMutation】

# 快速上手

安装依赖：
```js
npm install @tanstack/react-query
```

`main.js`
```jsx
import { createRoot } from "react-dom/client";
import { RouterProvider } from "react-router/dom";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

import router from "@/routes";

const queryClient = new QueryClient();

createRoot(document.getElementById("root")).render(
  <QueryClientProvider client={queryClient}>
    <RouterProvider router={router} />
  </QueryClientProvider>,
);
```

使用：
```jsx
import { useQuery } from "@tanstack/react-query";

const query = useQuery({
	requestKey: ["mockList"],
	requestFn: async () => {
		const res = await request({ url: "/mock/list", method: "get" })
		return res?.data ?? [],
	},
	enabled: false, // 初次不自动请求数据
})
```

返回值：
```js
  const { data = [], isLoading, isError, refetch, isFetched } = query;
```

+ data: 请求函数返回数据
+ isError： 请求过程中是否在错误
+ refetch： 调用可重新请求接口
+ isFetched： 数据是否正在请求中
+ isLoading: 数据是否正在加载

使用 refetch：
点击按钮后，在重发请求
```jsx
<button onClick={() => refetch()}>获取数据</button>
```

## 核心概念
React Query 强大的秘密在于它区分了数据的“新鲜度”（Fresh）和“缓存期”（Cached）**，这主要由两个核心配置项控制：staleTime 和 cacheTime。

🥛 **Stale Time（新鲜时间）**
+ 定义： 数据被视为“新鲜”（Fresh）的时间。只要数据处于“新鲜期”，任何组件请求它时，都会直接从缓存中读取，不会发送新的网络请求。
+ 默认值： 0 毫秒。这意味着数据一旦被获取，马上就会变为“陈旧”（Stale）。
+ “陈旧”（Stale）数据： 一旦数据变为陈旧，React Query 仍然会显示它（因为它在缓存中），但它会在后台静默地发起一次新的请求来更新数据（称为 "Stale-While-Revalidate" 策略）。

🗑️ **Cache Time（缓存时间）**
+ 定义： 数据在缓存中被保留的时间，即使它已经不再被任何组件使用（即数据查询处于 Inactive 状态）。
+ 默认值： 5 分钟 (300,000 毫秒)。
+ 机制： 一旦一个查询数据变为 Inactive（例如用户离开了使用该数据的页面），它就会进入这个倒计时。如果超过 cacheTime，该数据就会被垃圾回收（Garbage Collected），彻底从内存中移除。

| 特性  | staleTime (新鲜时间)           | cacheTime (缓存时间)                |
| --- | -------------------------- | ------------------------------- |
| 决定  | 何时重新获取 (Re-fetch)          | 何时被销毁 (Garbage Collect)         |
| 状态  | Fresh (在期限内) → Stale (过期后) | Inactive (在期限内) → Deleted (过期后) |
| 默认值 | 0 ms                       | 5 min                           |

---
总结来说，
+ 如果数据状态为 fresh，读取都从缓存中读取; 超过 stale time 进入 stale 状态，界面依然显示，但后台会静默发一次请求
+ 在缓存期限内 inactive，如果数据没有被使用，超过 cache time 进入 deleted 状态，彻底从内存中移出

## UseQuery
Tanstack/query 未 uesQuery 提供了很多 option

特点：
1. 请求失败后，会重发 5 次请求 --- 失败才会变成 isError 状态
2. 组件重新渲染，会自动发请求
