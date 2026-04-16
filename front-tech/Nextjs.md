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
app/blog/a/b/c
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

