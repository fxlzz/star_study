# React Query 指南

> 本文档介绍 React Query（现名 TanStack Query）的核心概念、价值和最佳实践。

---

## 1. React Query 简介

**React Query**（现名 **TanStack Query**）是一个强大的数据获取和服务端状态管理库，专为 React 应用程序设计。它无需配置即可开箱即用，被认为是管理服务端状态的最佳解决方案之一。

### 核心特性

- **自动缓存** — 自动缓存服务端响应数据，显著减少不必要的 API 调用
- **后台重新获取** — 静默地在后台获取新数据，保持 UI 响应性
- **声明式 API** — 基于 Hooks 的简洁接口（`useQuery`、`useMutation`、`useInfiniteQuery`）
- **加载与错误状态** — 内置状态跟踪和自动重试机制
- **查询失效** — 手动缓存失效和自动重新获取
- **渲染优化** — 仅在数据实际变化时触发重新渲染
- **分页与无限滚动** — 内置 `useInfiniteQuery` 支持
- **TypeScript 支持** — 完整的类型推断
- **跨框架支持** — 支持 React、Vue、Solid、Svelte 和 Angular


---

## 2. React Query 的核心价值

React Query 解决的「核心痛点」：**React 本身的短板 + 前端数据请求的通用难题**

React 框架的核心定位是「UI 渲染库」，它只负责「状态驱动视图更新」，对「**服务端异步数据**」的管理是天然缺失的，没有提供任何官方的异步数据处理方案。

### 原有方案的困境

在 React Query 出现前，我们处理接口请求（异步数据）只有 2 种方案：

- **手写 `useState` + `useEffect`**
- **Redux / Zustand / Pinia** 等状态库存储异步数据

这两种方案都会让前端开发陷入「**全量的手动工作**」。React Query 就是为了解决这些问题而生，它是专门为「**服务端异步数据**」设计的状态管理器，只关注「从服务器获取、同步、缓存、更新的异步数据」，不负责本地 UI 状态（如按钮显隐、表单输入值）。

### React Query 解决的核心问题

1. 解决 **`useState` + `useEffect` 手写请求的样板代码冗余、逻辑分散** 问题
2. 解决 **异步请求的 loading / error / success 状态需要手动维护** 的问题
3. 解决 **重复请求、并发请求的竞态问题**（Race Condition）— 比如快速切换标签页触发多次请求，后返回的结果覆盖先返回的，导致页面数据错乱
4. 解决 **服务端数据的缓存、过期、失效、重新请求** 等手动维护成本极高的问题
5. 解决 **数据同步问题** — 页面切换 / 组件复用 / 窗口聚焦时，自动同步最新的服务端数据，无需手动写刷新逻辑
6. 解决 **分页 / 无限滚动 / 下拉刷新** 等高频业务场景的逻辑复用难题
7. 解决 **乐观更新（先更新 UI 再等接口响应）、请求取消、重试机制** 等业务需求的低成本实现问题
8. 解决 **全局异步数据共享** 问题，无需通过 Redux 写大量 action / reducer，组件间可直接复用缓存数据

---

## 3. React 原生方案 & React Query 对比实现

所有实例都是真实开发中 100% 会遇到的场景，对比「原生写法」和「React Query 写法」的差异，直观感受解决的问题。

### 基础异步请求 — 解决「样板代码冗余 + 手动维护请求状态」

#### 业务需求

调用 `/api/user` 获取用户信息，展示加载中、数据、错误提示，组件卸载时取消请求防止内存泄漏。

#### 原生 useState + useEffect 写法

```jsx
import { useState, useEffect } from 'react';
import axios from 'axios';

function User() {
  // 痛点1：手动声明 3+ 个状态，状态零散
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController(); // 取消请求的控制器
    const fetchUser = async () => {
      try {
        setLoading(true); // 痛点2：手动设置loading
        setError(null);   // 痛点3：手动清空错误状态
        const res = await axios.get('/api/user', { signal: controller.signal });
        setUser(res.data);
      } catch (err) {
        // 痛点4：手动捕获错误并赋值
        setError('获取用户信息失败，请重试');
      } finally {
        setLoading(false); // 痛点5：手动关闭loading
      }
    };
    fetchUser();

    // 痛点6：手动处理组件卸载的请求取消，否则会内存泄漏+报错
    return () => controller.abort();
  }, []);

  if (loading) return <div>加载中...</div>;
  if (error) return <div>{error}</div>;
  return <div>用户名：{user.name}</div>;
}
```

上面的代码，真正的业务逻辑只有 `axios.get('/api/user')` 一行，其余全是「重复的、无意义的样板代码」，且每写一个接口都要复制一遍这套逻辑。

#### React Query 写法

```jsx
import { useQuery } from '@tanstack/react-query';
import axios from 'axios';

// 封装请求函数，纯业务逻辑，无状态
const fetchUser = async () => {
  const res = await axios.get('/api/user');
  return res.data;
};

function User() {
  // loading/error/data 请求状态管理，无需手动维护
  // queryKey 全局缓存
  // 自动取消组件卸载的请求，自动捕获错误，无需手动处理
  const { isLoading, error, data: user } = useQuery({
    queryKey: ['user'],
    queryFn: fetchUser,
  });

  if (isLoading) return <div>加载中...</div>;
  if (error) return <div>获取用户信息失败，请重试</div>;
  return <div>用户名：{user.name}</div>;
}
```

对比结论：React Query 把「异步请求的状态管理」完全封装，开发者只需要关注「**怎么请求数据**」，不需要关注「**怎么管理请求状态**」。

---

### 解决「重复请求 + 数据缓存」问题（原生方案几乎无解）

#### 业务痛点

- 同一个组件在多个页面复用（比如「用户信息组件」在个人中心、首页、设置页都有），每次渲染都会发起一次 `/api/user` 请求 → 重复请求，浪费带宽 + 服务器资源
- 从「个人中心」切到「首页」再切回「个人中心」，组件重新渲染，又会发起请求 → 用户体验差，页面反复加载
- 原生方案想做缓存，需要手动用全局变量 / Redux 存数据，还要手动判断数据是否过期，逻辑极其复杂，容易出 Bug

#### React Query 的「零成本解决方案」

```jsx
const { isLoading, error, data: user } = useQuery({
  queryKey: ['user'], // 核心：相同的 queryKey 对应相同的缓存数据
  queryFn: fetchUser,
  staleTime: 5 * 60 * 1000, // 数据「新鲜时间」5分钟，可选配置
  gcTime: 30 * 60 * 1000, // 不活跃数据「回收时间」30分钟，可选配置
});
```

React Query 自动实现的能力：

- **自动缓存** — 第一次请求 `/api/user` 后，数据会被缓存到内存中，`queryKey=['user']` 对应的所有组件，都会复用这份缓存数据，不会发起重复请求
- **新鲜数据复用** — 在 `staleTime`（5 分钟）内，组件重新渲染 / 页面切换，直接用缓存数据，不会发起新请求，页面秒开
- **自动过期刷新** — 超过 `staleTime` 后，数据变为「过期状态」，此时组件渲染会在后台静默发起请求更新数据，页面不会出现加载中，用户无感知
- **自动清理缓存** — 超过 `gcTime`（30 分钟）后，缓存数据会被自动清理，释放内存

> 补充：原生方案要实现这个功能，需要手动写「缓存池 + 过期时间 + 清理逻辑」，代码量至少增加 50 行，且极易出错。

---

### 解决「请求竞态问题（Race Condition）」（原生方案难排查）

#### 业务痛点

- **竞态问题** — 多个相同接口的异步请求，请求发起顺序和响应返回顺序不一致，导致页面展示错误的数据
- **典型场景** — 一个「商品列表」组件，支持按「价格」「销量」排序，用户快速点击「价格升序」→「销量降序」→「价格降序」，会发起 3 次请求，假设：
  - 价格升序请求：发起最早，返回最慢（3 秒）
  - 价格降序请求：发起最晚，返回最快（1 秒）

此时原生方案中，页面会先展示「价格降序」的正确数据，3 秒后被「价格升序」的旧数据覆盖，页面数据错乱！这种 Bug 很难复现和排查，是前端异步请求的「噩梦」。

#### React Query 的「零成本解决」

React Query 对同一个 `queryKey` 的请求，会自动处理竞态问题：当新的请求发起时，自动取消旧的请求，旧请求的响应结果会被忽略，永远只会展示「最新一次请求」的结果，彻底杜绝竞态问题。

> 补充：无需任何额外配置，开箱即用！

---

### 解决「数据同步 + 后台刷新」问题（业务刚需，原生方案繁琐）

#### 业务痛点

- 用户打开页面后，切到其他浏览器标签页，过了一段时间切回来，页面数据已经过期（比如商品库存、订单状态），需要手动点击「刷新」按钮才能更新
- 移动端下拉刷新列表，需要手动写「下拉触发请求 + 刷新状态」逻辑
- 原生方案要实现「窗口聚焦刷新」，需要监听 `visibilitychange` 事件，手动判断数据是否过期，手动发起请求，逻辑分散且耦合

#### React Query 的「极简实现」

```jsx
const { data: goodsList, refetch } = useQuery({
  queryKey: ['goodsList'],
  queryFn: fetchGoodsList,
  refetchOnWindowFocus: true, // 窗口聚焦时，自动后台刷新数据
  refetchOnReconnect: true,   // 网络重连时，自动刷新数据
});
```

核心能力：

- **无感知刷新** — 窗口聚焦时，数据会在后台静默请求，不会展示加载中，请求完成后自动更新页面，用户体验极佳
- **手动刷新便捷** — React Query 会返回 `refetch` 方法，下拉刷新时直接调用 `refetch()` 即可，无需手动维护刷新状态
- 所有配置都是「声明式」的，无需写任何监听事件，逻辑解耦

---

### 解决「乐观更新」问题（提升用户体验，原生方案复杂）

#### 业务场景

用户点击「收藏商品」按钮，希望立刻看到收藏状态变化，而不是等待接口返回后再更新 UI（接口可能有 1-2 秒延迟），这种「先更新 UI，再等待接口响应」的方式就是**乐观更新**，是提升用户体验的核心手段。

#### 原生方案痛点

需要手动：
1. 复制一份数据并修改状态
2. 发起请求
3. 请求成功则无操作
4. 请求失败则回滚数据

代码量大且容易出错。

#### React Query 乐观更新（简洁、可靠）

```jsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

const queryClient = useQueryClient();
// 查询商品列表
const { data: goodsList } = useQuery({ queryKey: ['goodsList'], queryFn: fetchGoodsList });

// 收藏商品的修改请求（mutation 用于增删改）
const collectGoods = useMutation({
  mutationFn: async (goodsId) => await axios.post(`/api/collect/${goodsId}`),
  // 乐观更新核心：请求发起前，先更新缓存中的数据
  onMutate: async (goodsId) => {
    // 取消当前的请求，防止竞态
    await queryClient.cancelQueries({ queryKey: ['goodsList'] });
    // 获取缓存的旧数据
    const oldGoodsList = queryClient.getQueryData(['goodsList']);
    // 更新缓存数据，页面立刻刷新
    queryClient.setQueryData(['goodsList'], oldGoodsList.map(item => 
      item.id === goodsId ? { ...item, isCollected: true } : item
    ));
    // 返回旧数据，用于失败回滚
    return { oldGoodsList };
  },
  // 请求失败：回滚数据
  onError: (err, goodsId, context) => {
    queryClient.setQueryData(['goodsList'], context.oldGoodsList);
  },
  // 请求成功：刷新缓存，保证数据一致性
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['goodsList'] });
  },
});

// 点击收藏按钮
const handleCollect = (id) => collectGoods.mutate(id);
```

核心价值：React Query 提供了完整的乐观更新生命周期，结合「缓存修改」能力，让原本复杂的乐观更新逻辑变得清晰可控，且不易出错。

---

## 4. 社区最佳实践

### 规范 `queryKey` 的命名规则

`queryKey` 是 React Query 缓存的唯一标识，是整个库的基石，命名不规范会导致缓存失效、重复请求、数据错乱，社区统一最佳规范：

**规则**：`[资源名, 参数对象/唯一标识, 筛选条件]`，数组格式，语义化清晰

**正确示例**：
  ```jsx
  // 单条用户数据：资源名+用户ID
  queryKey: ['user', 1001]
  // 商品列表：资源名+筛选条件（分页、排序、搜索）
  queryKey: ['goodsList', { page: 1, size: 10, sort: 'price', keyword: '手机' }]
  // 嵌套数据：资源名+父ID+子资源
  queryKey: ['order', 2002, 'detail']
  ```

**错误示例**：`queryKey: ['userInfo1001']`、`queryKey: ['goodsListPage1']`（硬编码，无法复用）

**好处**：参数变化时，`queryKey` 自动变化，React Query 会自动发起新请求，同时保留不同参数的缓存（比如分页 1 和分页 2 的商品列表会分开缓存）。

---

### 全局初始化 React Query 客户端，统一配置默认参数

项目入口文件（`main.tsx` / `index.tsx`）中初始化 QueryClient，配置全局默认行为，避免每个组件重复配置，社区标配配置如下：

```jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'; // 开发必备调试工具

// 创建全局客户端
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1 * 60 * 1000,     // 全局默认 1 分钟新鲜时间，减少请求
      gcTime: 5 * 60 * 1000,        // 全局默认 5 分钟不活跃数据回收时间，减少内存占用
      refetchOnWindowFocus: import.meta.env.DEV, // 开发环境开启窗口聚焦刷新，生产环境关闭
      retry: 1,                     // 请求失败后重试 1 次，适合网络抖动场景
      retryDelay: (attempt) => Math.min(1000 * attempt, 3000), // 指数退避重试，避免频繁请求
      refetchOnReconnect: true,     // 网络重连自动刷新
    },
  },
});

// 包裹根组件
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <RootComponent />
      <ReactQueryDevtools initialIsOpen={false} /> {/* 开发环境显示调试工具 */}
    </QueryClientProvider>
  );
}
```

---

### 抽离请求逻辑，封装「自定义 Hook」，实现逻辑复用

核心原则：**永远不要在组件内直接写 `useQuery` / `useMutation`，而是封装成自定义 Hook**，这是社区的「铁律」，理由如下：

- **组件只关注「使用数据」，不关注「怎么获取数据」** — 解耦 UI 渲染和业务逻辑
  - 组件的核心职责：根据状态渲染 UI + 响应用户操作，不应该承载「请求逻辑、异步状态管理、缓存更新、乐观更新」等业务逻辑

- **同一个接口的请求逻辑只写一次** — 所有组件复用
  - 一些业务数据请求逻辑，在项目中大概率会在多个组件复用，抽取到 Hook 实现一处编写到处复用

- **后续接口地址、参数变化** — 只需要修改一个地方，无需全局替换
  - 所有和「增删改」相关的逻辑：请求函数、请求参数处理、loading 状态、错误提示、乐观更新、缓存失效、重试机制... 全部收敛到一个自定义 Hook 中

- **统一项目编码规范** — 降低团队协作成本
  - 一个团队的项目，必须有统一的规范：所有 React Query 相关的 API（`useQuery` / `useMutation` / `useInfiniteQuery`），全部封装到自定义 Hook 中。新人接手项目时，不用去各个组件里找请求逻辑，只需要去 hooks 目录找对应的自定义 Hook 即可，心智负担极低

- **避免组件冗余** — 保持组件代码极简
  - `useMutation` 会返回 `isPending`、`error`、`mutate`、`mutateAsync` 等一堆状态和方法，如果直接写在组件里，组件会充斥大量和请求相关的代码，而组件本身的渲染逻辑被淹没，可读性极差

- **方便做统一的异常处理、公共逻辑抽离**
  - 比如：所有的 `POST` 请求失败后，都要弹出统一的错误提示（如「操作失败，请重试」），所有的提交按钮点击后都要禁用防止重复提交，这些公共逻辑可以在自定义 Hook 里统一处理，不用在每个组件里写重复的 `if(error)` 判断

#### 最佳实践示例

```jsx
// hooks/useUser.ts 抽离请求逻辑
import { useQuery } from '@tanstack/react-query';
import { fetchUser, fetchUserList } from '@/api/user';

// 单个用户信息Hook
export const useUser = (userId) => {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId, // 关键：userId 存在时才发起请求，避免无意义请求
  });
};

// 用户列表Hook
export const useUserList = (params) => {
  return useQuery({
    queryKey: ['userList', params],
    queryFn: () => fetchUserList(params),
  });
};

// 组件中使用，极简！
function UserDetail({ userId }) {
  const { isLoading, data: user } = useUser(userId);
  // ...渲染逻辑
}
```

---

### 合理使用 enabled 配置项，控制请求的执行时机

`enabled` 是 `useQuery` 的核心配置，值为 boolean，表示「是否启用该查询」，这是解决「条件请求」的最佳方案：

- **场景 1** — 依赖某个参数存在才请求（如上面的 `userId`）：`enabled: !!userId`
- **场景 2** — 表单筛选后，点击「查询」按钮才请求列表：`enabled: isSearchClicked`
- **场景 3** — 依赖其他查询的结果才请求：`enabled: !!otherData`
- 不要用 `if` 判断再写 `useQuery`，违反 React Hook 的调用规则

---

### 增删改查分离：`useQuery` 只读，`useMutation` 写操作

- 所有 GET 请求用 `useQuery`，所有 `POST` / `PUT` / `DELETE` 用 `useMutation`
- 执行写操作后，通过 `queryClient.invalidateQueries({ queryKey: ['资源名'] })` 让对应缓存失效，React Query 会自动发起新请求刷新数据，这是「修改数据后刷新列表」的标准写法，比手动调用请求函数优雅 10 倍

---

### 开启 React Query Devtools 调试工具（开发必备）

React Query 提供了官方调试工具，能直观看到：缓存的所有数据、请求状态、查询键、缓存时间、是否过期等，开发时开启，能快速定位缓存和请求问题，生产环境自动关闭即可，无性能损耗。

---

## 5. 避坑指南

| 坑点 | 避坑方案 |
|------|----------|
| 把「本地 UI 状态」交给 React Query 管理 | React Query 只负责「**服务端异步数据**」，本地状态（如按钮是否禁用、表单输入值、弹窗显隐）用 `useState` / `useReducer` 管理，不要混用 |
| 忽略 `enabled` 配置，导致「空参数请求」 | 请求 `/api/user/${id}` 时，当 `id` 为 `undefined` / `null` 时，一定要设置 `enabled: !!id`，否则会发起 `/api/user/undefined` 的无效请求 |
| 滥用 `refetchOnWindowFocus`，生产环境频繁请求 | 开发环境可以开启，**生产环境建议关闭（设置为 false）**，否则用户频繁切标签页会导致大量后台请求，增加服务器压力 |
| 缓存数据修改后，不主动失效，导致数据不一致 | 执行增删改操作后，必须调用 `invalidateQueries` 让对应缓存失效，比如新增商品后，失效 `['goodsList']` 缓存，确保列表数据是最新的 |
| 设置过长的 `staleTime`，导致数据一直不更新 | `staleTime` 是「数据新鲜时间」，不是「缓存时间」，根据业务场景合理设置：比如「商品列表」设置 10 分钟，「实时库存」设置 10 秒，「静态数据」设置 1 小时，不要无脑设置成 1 天 |

---

## 6. `useQuery` / `useMutation` 完整封装实战

> 注意：**演示功能代码存放在 `src/features/goods` 目录下，实际目录结构根据项目情况、规范等自行调整**

### 场景 1：基础场景 — 简单的增 / 删 / 改（无乐观更新，无复杂逻辑）

#### API 纯请求函数

```jsx
// api.js
import axios from 'axios';

// 纯请求函数，只负责发请求，无任何状态
export const fetchGoodsList = (params) => axios.get('/api/goods', { params });
export const deleteGoods = (goodsId: number) => axios.delete(`/api/goods/${goodsId}`);
export const addGoods = (goodsData) => axios.post('/api/goods', goodsData);
```

#### 封装自定义 Hook

```jsx
// hooks.js
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { fetchGoodsList, deleteGoods, addGoods } from './api';

// ------ 封装 useQuery：处理 GET 查询 ------
export const useGoodsList = (params) => {
  return useQuery({
    queryKey: ['goodsList', params],
    queryFn: () => fetchGoodsList(params),
  });
};

// ------ 封装 useMutation：处理 DELETE 删除 ------
export const useDeleteGoods = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: deleteGoods, // 直接传入 api 层的纯请求函数
    // 请求失败的统一错误提示
    onError: (error) => {
      console.error('删除商品失败：', error);
      alert('删除失败，请重试！');
    },
    // 请求成功后：让商品列表缓存失效，自动刷新最新数据
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['goodsList'] });
      alert('删除成功！');
    },
  });
};

// ------ 封装 useMutation：处理 POST 新增 ------
export const useAddGoods = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: addGoods,
    onError: () => alert('新增商品失败，请重试！'),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['goodsList'] });
      alert('新增成功！');
    },
  });
};
```

#### 在组件中消费

```jsx
// components/GoodsList.jsx
import { useGoodsList, useDeleteGoods } from '../hooks';

export const GoodsList = () => {
  // 消费查询 Hook：获取商品列表
  const { isLoading, data: goodsList } = useGoodsList({ page: 1, size: 10 });
  // 消费删除 Hook：获取删除的状态和方法
  const { isPending: isDeleting, mutate: handleDelete } = useDeleteGoods();

  if (isLoading) return <div>加载中...</div>;

  return (
    <div>
      {goodsList.map(item => (
        <div key={item.id}>
          <span>{item.name}</span>
          {/* 点击触发删除，isDeleting 控制按钮禁用，防止重复点击 */}
          <button
            onClick={() => handleDelete(item.id)}
            disabled={isDeleting}
          >
            {isDeleting ? '删除中...' : '删除'}
          </button>
        </div>
      ))}
    </div>
  );
};
```

> 组件层的作用极致纯粹：**组件里没有任何请求逻辑，没有任何 `useMutation` 原生代码，只关心「渲染什么」和「点击按钮调用方法」，这就是封装的终极价值**

---

### 场景 2：进阶场景 — 带「乐观更新」的 `useMutation` 封装（高频业务场景）

这是实际开发中最常用的复杂场景：比如「收藏商品、点赞、修改订单状态」，需要先更新 UI，再等待接口响应（乐观更新），失败后还要回滚数据。这类复杂逻辑必须全部封装在自定义 Hook 里，组件里依然极简调用，这也是封装的核心意义。

#### 在 hooks 中新增

```jsx
// 收藏商品的 mutation 封装，带乐观更新
export const useCollectGoods = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (goodsId: number) => axios.post(`/api/goods/collect/${goodsId}`),
    // 乐观更新核心：请求发起前，先修改缓存数据，页面立刻更新
    onMutate: async (goodsId) => {
      await queryClient.cancelQueries({ queryKey: ['goodsList'] });
      const oldData = queryClient.getQueryData(['goodsList']);
      // 修改缓存，实现 UI 即时更新
      queryClient.setQueryData(['goodsList'], (prev) =>
        prev.map(item => item.id === goodsId ? { ...item, isCollected: true } : item)
      );
      return { oldData };
    },
    // 请求失败：回滚数据
    onError: (err, goodsId, context) => {
      queryClient.setQueryData(['goodsList'], context.oldData);
      alert('收藏失败，请重试！');
    },
    // 请求完成：刷新缓存保证一致性
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['goodsList'] });
    },
  });
};
```

#### 在组件中消费

```jsx
import { useGoodsList, useCollectGoods } from '../hooks';

const GoodsList = () => {
  const { data: goodsList } = useGoodsList({ page: 1, size: 10 });
  const { mutate: handleCollect } = useCollectGoods();

  return (
    <div>
      {goodsList.map(item => (
        <div key={item.id}>
          <span>{item.name}</span>
          <button onClick={() => handleCollect(item.id)}>
            {item.isCollected ? '已收藏' : '收藏'}
          </button>
        </div>
      ))}
    </div>
  );
};
```

> 核心亮点：组件里**完全感知不到「乐观更新、失败回滚」这些复杂逻辑**，只需要调用 `handleCollect` 即可，所有复杂逻辑都被封装在 Hook 里。

---

## 7. `useMutation` 最佳实践 & 避坑指南

> 注意：**最佳实践不代表项目规范，请根据自身项目实际情况，酌情考虑是否遵循**

### `useMutation` 的「条件触发」处理

`useQuery` 有 `enabled` 属性控制是否自动发起请求，而 `useMutation` 是手动触发（调用 `mutate` 方法），所以「条件触发」的逻辑写在组件里即可，比如：表单校验通过后再提交、有选中项才删除，这是合理的，不会破坏封装。

```jsx
// 组件内：条件触发，合理！
const handleSubmit = () => {
  if (!formData.name || !formData.price) {
    alert('请填写完整商品信息');
    return;
  }
  // 校验通过后再调用 mutation
  addGoodsMutate(formData);
};
```

### 统一区分 `mutate` 和 `mutateAsync`

`useMutation` 会返回两个核心调用方法，封装时无需做特殊处理，直接返回即可，组件按需使用：

- **`mutate`** — 无返回值，适合普通的同步触发（比如点击按钮），错误通过 `onError` 处理
- **`mutateAsync`** — 返回 Promise，适合异步逻辑中调用（比如需要在请求成功后执行其他异步操作），错误通过 `try` / `catch` 处理

```jsx
// Hook 里正常返回，组件按需解构
export const useAddGoods = () => {
  return useMutation({ mutationFn: addGoods });
};
// 组件里使用 mutateAsync
const { mutateAsync } = useAddGoods();
const handleSubmit = async () => {
  try {
    await mutateAsync(formData);
    router.push('/goods/list'); // 成功后跳转页面
  } catch (err) {
    console.error(err);
  }
};
```

### 不要在组件里处理「缓存失效」逻辑

执行完增删改后，需要刷新列表数据，这个逻辑**必须放在 Hook 的 `onSuccess` / `onSettled` 里**，而不是组件里。组件的职责只是「触发操作」，不应该关心「操作后要不要刷新数据」，这能彻底解耦。

### 对相同业务模块的 query 和 mutation 做「聚合封装」

比如 `goods` 功能模块中，所有的 Hook 都封装在 `hooks.js` 里（也可以按照其他细分规则，创建 hooks 目录，然后在目录下细分 hook），同时封装「查询商品列表、查询商品详情、新增商品、修改商品、删除商品、收藏商品」的所有逻辑，这样做的好处是：**同一个业务模块的所有异步操作都在一个文件里**，维护时不用到处找，这是社区的标准做法。

### 封装时统一处理「loading 状态命名」

`useMutation` 的加载状态是 `isPending`（React Query v5）/ `isLoading`（v4），建议在封装时可以重命名为业务语义化的状态，比如 `isDeleting`、`isAdding`，组件里使用时更直观。

```jsx
// Hook 里重命名，可选优化
export const useDeleteGoods = () => {
  const { isPending: isDeleting, mutate } = useMutation({ mutationFn: deleteGoods });
  return { isDeleting, deleteGoods: mutate };
};
// 组件里使用，语义化拉满
const { isDeleting, deleteGoods } = useDeleteGoods();
```

### 避坑：永远不要在组件里写「重复的 mutation 逻辑」

哪怕某个 `useMutation` 只在一个组件里用到，也必须封装到自定义 Hook 中！不要抱有「就这一个组件用，直接写吧」的想法，因为：

- 项目规范的一致性比什么都重要
- 需求永远会变，今天只用一次，明天可能就会在另一个组件复用
- 后期加功能（比如 loading、错误提示）时，修改成本极低

#### `useQuery` / `useMutation` 封装的终极价值

封装后，你的项目会形成「API 层 → Hook 层 → UI 层」的三层清晰结构，每层职责单一，互不耦合：

- **API 层** — 只发请求
- **Hook 层** — 只做异步状态管理和业务逻辑封装
- **UI 层** — 只做渲染和用户交互

这种结构下，你的项目会变得极度易维护、易扩展、易协作，这也是 React Query 社区能形成这个最佳实践的核心原因。

---

### `useMutation` 获取接口返回值 & 组件差异化处理

#### 问题

组件调用 `useMutation` 执行增删改后，需要拿到接口的返回值，根据返回数据做「**跳转、弹窗、更新本地状态、二次请求**」等差异化业务处理，该如何实现？

#### 答案

- `useMutation` 执行后，完全可以轻松获取接口的完整返回值，有 2 种官方推荐的核心方案：
  - **方案一**：`mutate` + 回调参数
  - **方案二**：`mutateAsync` + `try` / `catch` / `await`
- 两种方案都不破坏封装规范，自定义 Hook 的封装方式不变，所有公共逻辑（缓存失效、全局错误提示、乐观更新）依然收敛在 Hook 中
- 组件的「差异化处理逻辑（跳转、弹窗等）」只写在组件内部，做到「公共逻辑复用、个性化逻辑解耦」，完美兼顾复用性和灵活性
- 两种方案可以共存使用、按需选择，没有优劣之分，只是适配不同的业务场景

##### 前置知识：useMutation 的返回值 & 执行逻辑铺垫

我们封装的 `useMutation` 自定义 Hook，最终都会返回 `useMutation` 的原生返回对象，里面包含核心的：

| 返回值 | 说明 |
|--------|------|
| `mutate` | 最常用的手动触发方法（无返回值，适合普通点击触发） |
| `mutateAsync` | 返回 `Promise` 对象的触发方法（适合异步逻辑，可结合 `try` / `catch` / `await` 使用） |
| `isPending` | 请求中的加载状态（v5 版本，v4 是 `isLoading`） |
| `error` | 请求失败的错误信息 |
| `data` | 接口成功返回的结果（本次请求的最新返回值，核心） |

> 补充：接口的返回值，就是你 `api` 层请求函数 `return` 的内容（比如 axios 的 `res.data`），React Query 会原封不动的帮你保存。

---

#### 方案一：mutate + 回调参数（最常用，推荐 90% 场景使用）

##### 适用场景

所有普通的同步触发场景：点击按钮提交表单、点击删除、点击收藏等，组件需要在请求成功后，根据返回值做跳转、弹窗、更新本地状态等差异化处理，是业务开发中最主流的方案。

##### 核心原理

`useMutation` 的 `mutate` 方法支持传入两个参数：

- **第一个参数** — 传给 `mutationFn` 的业务参数（比如删除的 id、新增的表单数据）
- **第二个参数** — 可选的回调配置对象，包含 `onSuccess`、`onError`、`onSettled`，这是「组件级的个性化回调」

**回调执行顺序**：

Hook 内部定义的 `onSuccess` / `onError`（公共逻辑）→ 组件内 `mutate` 传的 `onSuccess` / `onError`（个性化逻辑）

先执行公共逻辑（比如缓存失效、全局提示「操作成功」），再执行组件的差异化逻辑（比如跳转页面、打开弹窗），完美契合业务需求。

**回调函数的参数**：

组件内的 `onSuccess` 回调，第一个形参就是接口的完整返回值，第二个形参是调用 `mutate` 时传入的业务参数，想怎么用就怎么用。

##### 完整示例

###### 步骤一：封装 Hook

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { addGoods } from '@/api/goods';

// 封装新增商品的 useMutation，公共逻辑都在这里
export const useAddGoods = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: addGoods, // 传入 api 层的请求函数，参数是表单数据
    // 公共成功逻辑：全局提示 + 失效缓存刷新列表，所有组件共用
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['goodsList'] });
      window.alert('商品新增成功（公共提示）');
    },
    // 公共失败逻辑：全局错误提示，所有组件共用
    onError: (err) => {
      window.alert(`新增失败：${err.message}`);
    },
  });
};
```

###### 步骤二：组件内调用 — 拿到返回值 + 差异化处理

组件里导入 Hook 后，解构出 `mutate` 和 `isPending`，调用 `mutate` 时传入第二个参数（回调对象），在 `onSuccess` 中拿到接口返回值，做任意差异化处理，比如：跳转页面、打开成功弹窗、存本地状态、传参跳转等。

```jsx
import { useNavigate } from 'react-router-dom'; // react-router v6
import { useState } from 'react';
import { useAddGoods } from '@/hooks/useGoods';

export const GoodsAdd = () => {
  const navigate = useNavigate();
  const [formData, setFormData] = useState({ name: '', price: 0 });
  // 1. 导入封装好的Hook，解构核心方法和状态
  const { mutate: handleAddGoods, isPending: isSubmitting } = useAddGoods();

  // 提交表单的核心方法
  const submitForm = () => {
    if (!formData.name || !formData.price) return;
    // 2. 调用mutate，传入【业务参数】+【组件级回调对象】
    handleAddGoods(formData, {
      // 这里拿到接口的完整返回值！res就是接口返回的数据
      onSuccess: (res) => {
        console.log('接口返回的完整数据：', res);
        // ------- 以下都是【组件差异化处理】，想写什么写什么 -------
        // 场景1：根据返回的商品ID，跳转到商品详情页（最常用）
        navigate(`/goods/detail/${res.id}`);
        // 场景2：打开自定义的成功弹窗（不是全局的alert）
        // openSuccessModal(`商品【${res.name}】创建成功`);
        // 场景3：更新组件本地状态
        // setSuccessMsg(`新增成功，商品ID：${res.id}`);
        // 场景4：调用其他组件方法
        // resetForm();
      },
      // 可选：组件级的错误兜底（如果需要和全局提示不同的逻辑）
      onError: (err) => {
        console.log('组件内的错误处理：', err);
      },
    });
  };

  return (
    <div>
      <input value={formData.name} onChange={(e) => setFormData({...formData, name: e.target.value})} placeholder="商品名称" />
      <input type="number" value={formData.price} onChange={(e) => setFormData({...formData, price: e.target.value})} placeholder="商品价格" />
      <button onClick={submitForm} disabled={isSubmitting}>
        {isSubmitting ? '提交中...' : '新增商品'}
      </button>
    </div>
  );
};
```

#### 方案二：mutateAsync + try/catch/await（异步方案，适配复杂场景）

##### 适用场景

组件需要在异步逻辑中执行请求，且需要根据请求结果做「串行的异步处理」，比如：

- 提交表单后，拿到返回值 → 再发起一个新的请求 → 再跳转页面
- 执行删除后，拿到返回值 → 再调用父组件的回调方法 → 再关闭弹窗
- 需要用 `await` 等待请求完成后，再执行后续逻辑的场景

简单说：**需要把请求当成「异步 Promise」处理时**，用这个方案。

##### 核心原理

- `useMutation` 除了 `mutate`，还提供了 `mutateAsync` 方法，这个方法本身会返回一个 Promise 对象
- 可以用 `async` / `await` 调用 `mutateAsync`，在 `try` 块中直接拿到接口返回值，在 `catch` 块中捕获错误
- 依然遵守「公共逻辑在 Hook，个性化逻辑在组件」的原则，Hook 封装不变

##### 完整示例

###### 步骤一：封装 Hook

```jsx
// 和方案一的封装完全一致，无任何修改
export const useAddGoods = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: addGoods,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['goodsList'] });
      window.alert('商品新增成功（公共提示）');
    },
    onError: (err) => {
      window.alert(`新增失败：${err.message}`);
    },
  });
};
```

###### 步骤二：组件内调用 — mutateAsync + await 拿返回值

组件里解构出 `mutateAsync` 替代 `mutate`，用 `async` / `await` 调用，在 `try` 中直接获取返回值，做差异化处理，写法完全符合前端异步编程习惯。

```jsx
import { useNavigate } from 'react-router-dom';
import { useState } from 'react';
import { useAddGoods } from '@/hooks/useGoods';

export const GoodsAdd = () => {
  const navigate = useNavigate();
  const [formData, setFormData] = useState({ name: '', price: 0 });
  // 1. 解构出 mutateAsync 方法
  const { mutateAsync: addGoodsAsync, isPending: isSubmitting } = useAddGoods();

  // 2. 定义异步的提交方法
  const submitForm = async () => {
    if (!formData.name || !formData.price) return;
    try {
      // 直接await，拿到接口返回值！这是最直观的方式
      const res = await addGoodsAsync(formData);
      console.log('接口返回的完整数据：', res);
      // ------- 组件差异化处理，串行执行异步逻辑 -------
      // 场景1：先调用其他异步方法，再跳转
      // await fetchGoodsDetail(res.id);
      // 场景2：带参数跳转
      navigate(`/goods/detail/${res.id}`);
      // 场景3：更新本地状态
      // setSuccessId(res.id);
    } catch (err) {
      // 组件级的错误处理（可选）
      console.log('组件内捕获错误：', err);
    }
  };

  return (
    <div>
      {/* 表单和按钮和方案一完全一致 */}
      <input value={formData.name} onChange={(e) => setFormData({...formData, name: e.target.value})} placeholder="商品名称" />
      <input type="number" value={formData.price} onChange={(e) => setFormData({...formData, price: e.target.value})} placeholder="商品价格" />
      <button onClick={submitForm} disabled={isSubmitting}>
        {isSubmitting ? '提交中...' : '新增商品'}
      </button>
    </div>
  );
};
```

#### 补充方案：直接读取 mutation 的 data 属性（兜底方案）

适用场景

简单的业务场景，不需要回调，只想在组件中「被动读取」本次请求的返回值（比如请求成功后，在页面上展示返回的 ID / 名称）。

核心原理

useMutation 的返回对象中，自带一个 data 属性，这个属性会存储本次请求成功后的接口返回值，请求失败 / 未发起时为 undefined。

组件内使用示例

```jsx
const { mutate: handleAddGoods, isPending, data: addRes } = useAddGoods();

// 点击提交
const submitForm = () => {
  handleAddGoods(formData);
};

// 页面上直接展示返回值
return (
  <div>
    {addRes && <div className="success">新增成功，商品ID：{addRes.id}</div>}
    {/* 表单和按钮 */}
  </div>
);
```

**特点**：最简单，无需回调，但灵活性差，适合「只展示返回值，无其他复杂逻辑」的场景，一般作为兜底方案使用。

---

## 8. 解决 `useQuery` 封装后「处理函数中按需查询数据」的问题

### 问题描述

- 组件的顶层渲染阶段，可以完美用封装的 `useUser(id)` Hook 获取数据，享受缓存 / 状态管理 / 去重等所有优势
- 组件的处理函数（事件回调、异步方法、点击事件）中，需要动态传参、按需查询数据（比如根据选中的 userId 查询用户信息，用返回值初始化表单 / 其他对象），但 Hook 不能在处理函数内调用（违反 React Hook 调用规则）
- 退回到直接调用 `getUser(id)` 原生请求函数，会出现「两套数据获取方式」：组件渲染用 Hook，业务逻辑用原生 API → 彻底破坏封装、丢失 React Query 所有核心能力（缓存、去重、竞态处理、请求状态）、还要手动写 `try` / `catch` / `loading`，违背封装初衷

### 造成问题的根本原因

- **React Hook 调用规则** — 所有 Hook（包括 `useQuery` / 自定义 Hook）只能在组件顶层、其他 Hook 顶层调用，绝对不能在 `onClick` / `async` 函数 / `if` 判断 / 循环 等「处理函数」中调用 → 这是 React 的硬性规则，无法突破
- **`useQuery` 的设计定位** — `useQuery` 是「声明式的、组件渲染阶段的查询 Hook」，它的核心是「组件依赖什么数据，我就声明式的获取什么数据」，天生不是为「手动按需触发的业务查询」设计的

### 核心需求

**在处理函数中「手动、按需、带参」获取异步数据，且复用 React Query 的缓存和能力**

---

### 方案一：预加载查询 + 缓存读取（推荐 90% 业务场景使用）

#### 适用场景

90% 的业务场景都适用，尤其是：

- 需要查询的 id 是可预知的、有限的、低基数的（比如：选中列表的用户 ID、下拉选择的商品 ID、弹窗传入的订单 ID）
- 处理函数中查询的数据，大概率已经被缓存过（比如：列表页已经加载过用户 1 / 2 / 3 的信息，详情弹窗里需要按需读取）
- 核心诉求：极致性能、零重复请求、零冗余代码

#### 核心思想

- 提前「声明式预加载」：在组件顶层调用封装好的 useUser(id) Hook，利用 useQuery 的 enabled 配置项，只声明查询、但不自动发起请求；
- 手动「按需触发查询」：在处理函数中，调用 Hook 返回的 refetch() 方法，手动触发查询，返回 Promise 可以直接 await 拿到接口返回值；
- 自动复用缓存：如果该用户数据已经被缓存过，refetch() 会直接读取缓存、不发起新请求，瞬间拿到数据；如果缓存不存在，才会发起真实请求，完美复用 React Query 的缓存能力。
  
核心优势

- 完全遵守所有规则：Hook 在组件顶层调用，符合 React 规则；
- 完全复用封装：所有查询逻辑依然在 useUser(id) 中，无双写法；
- 完全保留 React Query 所有能力：缓存、去重、竞态处理、loading/error 状态；
- 极致性能：缓存命中时零请求，缓存未命中时自动发起请求；
- 写法极简：组件代码几乎无冗余，处理函数中逻辑清晰；

完整示例

步骤 1：你的 useUser Hook 封装 完全不需要修改

```jsx
import { useQuery } from '@tanstack/react-query';
import { getUser } from './api';

// 标准封装的用户查询Hook
export const useUser = (userId: number) => {
  return useQuery({
    queryKey: ['user', userId], // 缓存唯一标识，核心
    queryFn: () => getUser(userId),
    enabled: !!userId, // 有userId才启用查询，避免无效请求
    staleTime: 5 * 60 * 1000, // 5分钟缓存新鲜期
  });
};
```

#### 步骤二：组件中「顶层预加载 + 处理函数中按需调用」

这是完整的业务场景：组件中有一个下拉框选择用户 ID，点击「查询用户并初始化表单」按钮，在按钮的点击事件（处理函数）中，根据选中的 ID 查询用户数据，用返回值初始化表单对象。

```jsx
import { useState } from 'react';
import { useUser } from './hooks';

export const UserForm = () => {
  // 1. 组件本地状态：存储选中的用户 ID、表单数据
  const [selectedUserId, setSelectedUserId] = useState<number>(0);
  const [formData, setFormData] = useState({ name: '', age: 0, email: '' });

  // 核心：组件顶层调用封装好的 useUser Hook 【预加载】
  // 此时只是声明查询，enabled 会根据 selectedUserId 自动控制是否启用
  const {
    data: userData,
    isLoading,
    refetch // 关键方法：手动触发查询的核心，返回 Promise
  } = useUser(selectedUserId);

  // 处理函数（点击事件）：按需查询数据、初始化表单 【你的核心需求】
  const handleInitForm = async () => {
    if (!selectedUserId) {
      alert('请选择用户 ID');
      return;
    }

    try {
      // 核心：调用 refetch()，手动触发查询，await 直接拿到【接口返回值】
      // 核心特性：如果该用户数据已缓存 → 直接读缓存，无请求；无缓存 → 发起请求
      const user = await refetch().data;

      // 拿到返回值后，做差异化处理：初始化表单对��（你的业务需求）
      setFormData({
        name: user.name,
        age: user.age,
        email: user.email,
      });
      console.log('表单初始化成功：', formData);
    } catch (err) {
      alert('查询用户失败，请重试');
      console.error(err);
    }
  };

  return (
    <div>
      <select 
        value={selectedUserId} 
        onChange={(e) => setSelectedUserId(Number(e.target.value))}
      >
        <option value={0}>请选择用户</option>
        <option value={1}>用户1</option>
        <option value={2}>用户2</option>
        <option value={3}>用户3</option>
      </select>

      <button onClick={handleInitForm} disabled={isLoading}>
        {isLoading ? '加载中...' : '查询用户并初始化表单'}
      </button>

      <div style={{ marginTop: 20 }}>
        <div>姓名：{formData.name}</div>
        <div>年龄：{formData.age}</div>
        <div>邮箱：{formData.email}</div>
      </div>
    </div>
  );
};
```

##### 补充：`refetch()` 方法的核心特性

- `useQuery` 返回的 `refetch()` 是解决这个问题的核心 API，你必须了解它的特性，才能用好这个方案
- `refetch()` 是异步方法，返回 Promise，可以用 `await` 直接获取返回值，返回值格式是 `{ data: T, isSuccess: true }`，所以 `refetch().data` 就是接口返回的真实数据
- `refetch()` 会完全复用你在 Hook 中配置的所有参数：`staleTime` / `gcTime` / `retry` 等，无需重新配置
- `refetch()` 会自动更新 React Query 的缓存：请求成功后，缓存会被更新，其他组件用 `useUser(id)` 也能拿到最新数据
- `refetch()` 会自动处理竞态问题：多次调用时，自动取消旧请求，只保留最新的一次
- `refetch()` 的 loading 状态，会自动同步到 `useQuery` 返回的 `isLoading`，无需手动维护

---

### 方案二：用 useMutation 封装「手动查询逻辑」（推荐 10% 高动态场景）

#### 适用场景

剩下 10% 的业务场景，也是方案一的完美补充，尤其是：

- 需要查询的 id 是完全不可预知、动态生成的（比如：用户手动输入任意用户 ID、批量查询海量用户 ID、后端返回的随机 ID）
- 不想在组件顶层预加载多个 `useQuery` Hook，组件逻辑比较复杂，预加载会增加心智负担
- 核心诉求：极致灵活、按需查询、无预加载、依然复用封装和缓存

#### 核心思想

- 打破「只有 `useQuery` 能做查询」的思维定式 → React Query 中，`useMutation` 不是只能做增删改，也可以做「手动触发的查询」
  - `useQuery` = 声明式、组件渲染阶段、自动触发的查询
  - `useMutation` = 命令式、手动触发、无调用时机限制的异步操作

基于这个思想，我们在原有的 `useUser.ts` Hook 文件中，新增一个封装的「手动查询用户」的 `useFetchUser` Hook，内部用 `useMutation` 封装查询逻辑，复用同一个 `getUser` 原生请求函数，组件中在「处理函数」中调用这个 Hook 的 `mutateAsync` 方法，就能完美解决问题。

#### 核心优势

- 完全遵守 React 规则：Hook 在组件顶层调用，处理函数中只调用 `mutateAsync` 方法
- 完全复用封装：所有查询逻辑依然收敛在 `useUser.ts` 中，项目中只有一套数据获取逻辑，无双写法
- 完全保留 React Query 能力：可以结合 `queryClient` 读取缓存，避免重复请求
- 极致灵活：无需预加载，想查什么 ID 就传什么 ID，处理函数中调用无任何限制
- 无侵入：原有的 `useUser` Hook 完全不变，组件按需选择用哪个 Hook

#### 完整示例

##### 步骤一：在 useUser.ts 中新增封装 useFetchUser Hook（原 useUser 不变）

```jsx
import { useQuery } from '@tanstack/react-query';
import { getUser } from './api';

// 标准封装的用户查询 Hook
export const useUser = (userId: number) => {
  return useQuery({
    queryKey: ['user', userId], // 缓存唯一标识，核心
    queryFn: () => getUser(userId),
    enabled: !!userId, // 有 userId 才启用查询，避免无效请求
    staleTime: 5 * 60 * 1000, // 5 分钟缓存新鲜期
  });
};

// 新增：命令式手动查询 Hook，处理函数中用，按需查询用 【核心】
export const useFetchUser = () => {
  return useMutation({
    mutationFn: (userId: number) => getUser(userId), // 复用同一个 api 请求函数
  });
};
```

##### 步骤二：组件中「顶层调用 useFetchUser + 处理函数中按需查询」（核心写法）

同样是你的核心需求：处理函数中查询用户数据，初始化表单，无预加载、极致灵活。

```jsx
import { useState } from 'react';
import { useFetchUser } from './hooks';

const UserForm = () => {
  const [selectedUserId, setSelectedUserId] = useState<number>(0);
  const [formData, setFormData] = useState({ name: '', age: 0, email: '' });

  // 组件顶层调用：新增的手动查询Hook
  const { mutateAsync: fetchUser, isPending } = useFetchUser();

  // 处理函数中：按需查询、初始化表单 【你的核心需求】
  const handleInitForm = async () => {
    if (!selectedUserId) {
      alert('请选择用户ID');
      return;
    }

    try {
      // 核心：调用 mutateAsync，传参userId，await直接拿到返回值
      const user = await fetchUser(selectedUserId);
      
      // 业务处理：初始化表单
      setFormData({
        name: user.name,
        age: user.age,
        email: user.email,
      });
    } catch (err) {
      alert('查询用户失败');
      console.error(err);
    }
  };

  return (
    <div>
      <select 
        value={selectedUserId} 
        onChange={(e) => setSelectedUserId(Number(e.target.value))}
      >
        <option value={0}>请选择用户</option>
        <option value={1}>用户1</option>
        <option value={2}>用户2</option>
      </select>
      <button onClick={handleInitForm} disabled={isPending}>
        {isPending ? '加载中...' : '查询并初始化'}
      </button>
    </div>
  );
};
```

##### 进阶优化：给 useMutation 查询加「缓存优先」逻辑

这个方案唯一的小瑕疵：默认情况下，`useMutation` 会每次都发起真实请求，不会主动读取缓存。我们可以在封装时，结合 `useQueryClient` 加一行代码，实现「缓存优先，无缓存再请求」，完美补齐这个短板，让这个方案的能力和方案一持平！

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { getUser } from './api';

export const useFetchUser = () => {
  const queryClient = useQueryClient(); // 获取全局 QueryClient
  return useMutation({
    mutationFn: async (userId: number) => {
      // 核心优化：先读缓存，如果有缓存，直接返回缓存数据，不发起请求
      const cacheUser = queryClient.getQueryData(['user', userId]);
      if (cacheUser) {
        return cacheUser;
      }
      // 无缓存时，再发起真实请求
      return await getUser(userId);
    },
  });
};
```

---

### 方案三：全局 QueryClient 直接读取 / 请求（适合组件外查询）

#### 适用场景

- 特殊的小众场景，也是 React Query 的「终极兜底方案」，几乎能解决所有数据获取问题
- 需要在组件外查询数据（比如：工具函数、全局路由守卫、Redux action、utils 方法）
- 组件内逻辑极其复杂，不想用任何 Hook，只想用纯函数的方式获取数据
- 核心诉求：脱离组件、全局可用、依然复用缓存和请求逻辑

#### 核心思想

React Query 的所有缓存和请求能力，都来自于全局的 QueryClient 实例。我们可以在项目入口文件，把 QueryClient 实例导出为全局变量，然后在任意地方（组件内 / 组件外、处理函数 / 工具函数）调用它的两个核心方法：

- `queryClient.getQueryData(['user', id])` → 同步读取缓存数据，无请求，瞬间返回
- `queryClient.fetchQuery(['user', id], () => getUser(id))` → 异步查询数据，缓存优先（有缓存读缓存，无缓存发请求），返回 Promise，可 await 拿到值

#### 完整示例

##### 步骤一：全局导出 QueryClient 实例（项目入口文件 main.tsx / App.tsx）

```jsx
// src/main.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

// 创建并导出全局 QueryClient 实例
export const queryClient = new QueryClient({
  defaultOptions: { queries: { staleTime: 5 * 60 * 1000 } },
});

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <RootComponent />
    </QueryClientProvider>
  );
};
export default App;
```

##### 步骤二：在「任意地方」查询数据（组件处理函数 / 工具函��）

```jsx
import { useState } from 'react';
import { queryClient } from '@/main'; // 导入全局 QueryClient
import { getUser } from '@/api/user';

const UserForm = () => {
  const [selectedUserId, setSelectedUserId] = useState(0);
  const [formData, setFormData] = useState({});

  // 处理函数中：直接用全局 QueryClient 查询，无任何 Hook 限制
  const handleInitForm = async () => {
    if (!selectedUserId) return;
    try {
      // 核心：fetchQuery 是异步方法，缓存优先，返回 Promise
      const user = await queryClient.fetchQuery({
        queryKey: ['user', selectedUserId],
        queryFn: () => getUser(selectedUserId),
      });
      // 初始化表单
      setFormData(user);
    } catch (err) {
      console.error(err);
    }
  };

  return <div>...</div>;
};
```

#### 核心优势

- 完全脱离 Hook 限制，可以在任何地方查询数据
- 完全复用 React Query 的缓存和请求能力
- 完全遵守封装原则，请求逻辑还是在 `api/user.ts` 中
- 是 React Query 官方推荐的「全局数据获取方案」

---

### 总结

#### 三大方案的选型优先级

| 优先级 | 方案 | 适用场景 | 特点 |
|--------|------|----------|------|
| 首选 | 方案一（预加载 + refetch） | 90% 的业务场景 | 性能最优、代码最简洁、最符合 React Query 设计思想 |
| 次选 | 方案二（useMutation 封装手动查询） | 10% 的高动态场景 | 灵活度拉满，无预加载负担 |
| 兜底 | 方案三（全局 QueryClient） | 特殊场景（组件外查询） | 终极兜底，万能方案 |

#### React Query 项目封装原则

- **原则 1** — 所有「服务端数据获取逻辑」，统一收敛到 `src/api/` 层（纯请求函数，无状态）
- **原则 2** — 所有「React Query 状态管理逻辑」，统一收敛到 `src/hooks/` 层（自定义 Hook，封装 `useQuery` / `useMutation`）
- **原则 3** — 组件中只消费 Hook，永远不直接调用 api 层函数，永远不在组件中写原生 `useQuery` / `useMutation`
- **原则 4** — 组件顶层渲染数据 → 用封装的 `useXXX`（`useQuery`）
- **原则 5** — 组件处理函数按需查询 → 用「方案一 / 二 / 三」，永不退用原生 api
