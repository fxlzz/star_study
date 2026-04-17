适合做什么项目：
- SEO 项目（官网、博客）
- 电商
- SaaS
- 全栈项目
...
不太适合做什么项目：
+ 纯 SPA 后台系统
+ 强实时系统
...

# 路由规则
App Router
`app/` 目录自动生成（约定大于配置）

>  总之： Nextj 基于文件系统的路由机制，在 App Router 中通过 app 目录生成路由树。
>  当请求到达时，会按优先级匹配：静态路由> 动态路由 > 兜底路由，同时通过 pagejs 作为入口组件，layoutjs 控制页面布局

```js
app/
	page.js       -> /
	about/page.js -> /about
```

> 规则：
>  - 文件夹 = 路由路径 
>  - `page.js` = 页面入口

## *路由判定规则*
- 静态路由
```js
app/blog/123 ---> app/blog/123/page.tsx
```
- 动态路由 `[id]`
```js
app/blog/[id]/page.js

function Page({ params }) {
	console.log(params.id);
}
```
- （Catch-all）兜底匹配
```js
app/blog/[...slug]/page.js

app/blog/a
app/blog/a/b
app/blog/../../ 任意多层都会被 page.js 这个hander捕获到

params --- ["a", "b", ..]
```
+ （Optional Catch-all）可选
```js
app/blog/[[...slug]]/page.js

app/blog
app/blog/a
```

## *特殊路由文件*
+ `layout.js` —— 包裹页面
+ `loading.js` —— 自动作为 Suspense fallback
+ `error.js` —— 错误边界
+ `not-found.js` —— 404 界面

## *路由分组*
```js
app/(auth)/login/page.js  
app/(auth)/register/page.js

(auth) 不会出现在 URL 中
```

## *并行路由*
```js
app/@model/(...)login/page.js
```

# Route Handlers（API Route） 
在 Nextjs 中创建一个 api，支持原生 [Response](AJAX.md#Response) 和 [Request](AJAX.md#Request)
对象.  *这点非常重要!!! 所有的行为都于原生的保持一致* 
Nestjs 在此基础上封装了一层 NextResponse 和 NextRequest

支持 HTTP 方法： GET、POST、PUT、PATCH、DELETE、OPTIONS、HEAD，其他返回 405

Nextjs 配合路由实现 api 函数
```jsx
// node.js
app.get("/api/user/list", (req, res) => {
	res.json({ msg: "hello world"});
})

// nextjs
// 文件路由 api/user/list/route.ts
export const GET = async (request: NextRequest) => {
	return Response.json({ msg: "hello world" })
}
```

### *获取 query 参数* 
—— `/api/user?id=123`
```ts
export const GET = async (req) => {
	// serachParams ---> URLSearchParams 对象
	const { serachParams } = new URL(req.url);
	const id = serachParams.get("id");
	// ...
}	
```

### *获取 body*
```ts
export const POST = async (req) => {
	const body = await req.json();
	// ...
}
```

### *获取 headers*
```js
export const GET = async (req) => {
	const token = req.headers.get("authorization");
	// ...
}
```

### *获取 params 参数*
—— `/api/user/123`
注意： Nextjs 16 params 参数是异步的

```js
export const GET = async (req, { params: Promise<{id: string}> }) => {
	const { id } = await params;
	// ...
}
```

## 关于缓存
Route Handlers 默认不缓存，可以缓存 `GET` 方法
开启配置：
```js
export const dynamic = 'force-static'
 
export async function GET() {
  const res = await fetch('https://data.mongodb-api.com/...")
  const data = await res.json()
 
  return Response.json({ data })
}
```

### 启动缓存
需要配置缓存组件，通过在 `next.config.ts` 文件中设置 `cacheComponent: true`

Usage： 
+ Data 缓存一个获取或计算数据的函数
+ UI 缓存整个组件

*Data:*
```js
import { cacheLife } from 'next/cache'
 
export async function getUsers() {
  'use cache'
  cacheLife('hours')
  return db.query('SELECT * FROM users')
}
```

*UI:*
```js
import { cacheLife } from 'next/cache'
 
export default async function Page() {
  'use cache'
  cacheLife('hours')
 
  const users = await db.query('SELECT * FROM users')
 
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

# Zod
[Intro \| Zod](https://zod.dev/)
Api 参数校验，确实非常强，开发体验也不错（相比于 json-schema）

简单示例：
```js
import * as zod from "zod";

const querySchema = zod.object({
	name: zod.string().max(10).min(3),
	email: zod.string().email()
})

// 校验 —— 两种写法
try {
	const result = zod.parse({name: ..., email: ...})
	// ...
} catch(error) {
	// error instanseof ZodError
}

const result = zod.safeParse({name: ..., email: ...})

if (!result.sucess) {}
```

配合 drizzle-orm 校验
[Drizzle ORM - zod](https://orm.drizzle.team/docs/zod)

