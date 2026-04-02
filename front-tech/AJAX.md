>  2026-4-1 复习一下 AJAX，补点笔记

对于前端而言，发送 AJAX 的两大神器
+ axios 基于 xhr
+ fetch 基于 promise

## Axios
>  基于 promise 的 HTTP 客户端，可以在浏览器和 nodejs 环境使用

官网： [Axios Docs](https://axios-http.com/zh/docs/intro)

---
注意： axios 默认请求下，会将 JS 对象序列化为`JSON`

## 三种写法
+ `axios.[method](url, [config])`
+ `axios(config) | axios.request(config)`
+ `const curl = axios.create(config)`
```js
// 创建 axios 实例，实例也能使用前两种写法调用
const curl = axios.create({
	baseURL: 'http://localhost:3003',
	timeout: 6000,
	headers: {...}
})
```

## 配置说明 
###  `method`
支持 HTTP 协议请求报文中定义的方法
+ `get | post | delete | patch | put | options`

###  `config`
列举常见的配置[请求配置 \| Axios Docs](https://axios-http.com/zh/docs/req_config)

+ `url: string`
+ `method: enum`
+ `baseURL: string`
+ `headers: object`
+ `params: object` 应该是 query 参数 
	+ query 参数： `/api/files?a=1&b=2 ---> ?a=1&b=2` 
	+ params 参数： `/api/user:id ---> :id` 动态参数
+ `data: object(常见)`
	+ 在没有设置 `transformRequest` 时，则必须是以下类型之一: 
	- `string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams `
	- 浏览器专属: `FormData, File, Blob `
	- Node 专属: `Stream, Buffer`
+ `timeout: 0 number`
+ `withCredentials: false boolean` 跨域携带 cookie
+ `responseType: arraybuffer | document | json(默认) | text | stream`
	+ 浏览器专属： `blob`

### 响应结构
[响应结构 \| Axios Docs](https://axios-http.com/zh/docs/res_schema)

+ `data`
+ `status` HTTP 的状态码
+ `statusText`
+ `headers: object`  
	+ header 都是小写
	+ 访问： `response.headers['content-type']`
+ `config: object` axios 配置信息

## 拦截器

*请求拦截器*
```js
axios.interceptors.request.use((config) => {
	// 在请求之前做什么
	// 一般是加 token 
}, (err) => {
	// 对请求错误做什么
	return promise.reject(err);
})
```

*响应拦截器*
```js
axios.interceptors.response.use((response) => {
	// 对响应数据做些什么
	// 响应结构 {data, status, statusText, headers...}
	// 处理后端返回的 code 
	// code == 442 -> message.error("参数校验失败")
}, (err) => {
	// 不是 200 才会被捕获, http报文有问题
	// err.toJSON() 更多http错误信息
	return promise.reject(err)
})
```

请求超时：
`axios.isAxiosError(error) && error.code === "ECONNABORTED"` 


## 取消请求
几个场景：
+ 快速点击按钮 -- 发送请求 -- 导致服务器资源浪费 --> 只保留最后一个请求
+ tab 切换 -- 资源浪费 --> 前一个 tab 的请求取消
+ 手动中断请求 -- 图片上传至一半

原生 JS fetch API 方式 -- `AbortController` -- 传 `signal: controller.signal`

例如：
```js
const controller = new AbortController();

axios.get("/api/test", {
	signal: controller.signal
}).then(res => {})

// 取消请求
controller.abort();
```

# Fetch
浏览器提供的原生 HTTP 请求 API，使用 promise 方式代替 xhr 发送请求

---
<font color="#ff0000">注意：</font>
+ *query 参数不会自动拼接*（axios 会自动拼接 -> 设置 params 配置项）
+ 请求体 *body 参数*，必须是*字符串*`JSON.stringify`
+ 不会自动*携带 cookie* `credentials: "include"`
+  没有*超时机制*，需要配合 `abortController`
 
## 基础用法
`fetch` 函数的参数与 [Request]( #Request ) 对象保持一致 

```js
fetch(url, [options])
// 因为响应错误的HTTP状态码不会被拒绝，所以，then方法中需要检查 response.ok 或 response.status
	.then(response => response.json())
	.then(data => {
		console.log(data);
	})
	.catch(err => {
// 注意： HTTP响应状态码不为 200 时，也不会进入catch，只有 URL 格式错误或网络错误时，promise才会被拒绝
		console.error(err);
	})
```

调用流程：
```txt
fetch() 
  ↓
返回 Response 对象（不是数据！）
  ↓
调用 response.json()
  ↓
才是真正数据
```

## 超时机制 
```js
const controller = new AbortController();

setTimeout(() => {
	controller.abort();
}, 5000)

fetch("/api", {
	signal: controller.signal
})
```

## Request
官网： [Request() - Web API \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Request/Request)

`fetch` 的 `options` 配置：
+ `method` 
+ `headers`
+ `body` : 必须序列化
+ `mode: cors | no-cors | same-origin | navigate` 请求的模式
+ `credentials: include | omit | same-origin` 携带 cookie
+ `cache: default | no-store | reload | no-cache | force-cache` 
	+ [Request.cache - Web API \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Request/cache)
+ `redirect: follow | error | manual`
+ `signal: controller.signal` controller 为 abortController 的实例


## Response
官网：  [Response - Web API \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)

响应常见属性:
+ `headers` [Headers - Web API \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Headers)
+ `ok: boolean`： 200-299
+ `status: number`
+ `statusText`

常见方法：
+ `response.json()` 解析为 JSON 格式
+ `response.text()` 解析为 USVString 格式
+ `response.blob()` 解析为 Blob 格式
+ `response.arrayBuffer()` 解析为 ArrayBuffer 格式

# 相关 API
## `encodeURIComponent`
用于编码 query 参数的 value 值，因为 URL 只能包含 `A-Z a-z 0-9 - _ . ~`
例如，
+ `/api?name=张三` -> 请求失败，乱码
+ `params: {key: "a&b"}` -> 会解析为 `?key=a&b -> {key: "a", b: undefined}` 导致后端解析错误

这个 API 就是来解决这个问题的
经过处理后，
```js
encodeURIComponent("张三")
// → "%E5%BC%A0%E4%B8%89"
```

##  `URLSearchParams`
[URLSearchParams - Web API \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams)

`URLSearchParams(params) -> params: {key, value} | 二维数组 [[key, value]] `

*核心优势*
- **自动转义**：自动处理特殊字符（如空格变 `%20`，`&` 变成 `%26`）
- **可迭代**：可以直接把*对象、数组*转换成查询参数

### 初始化与序列化
```js
const params = new URLSearchParams({
  name: "张三",
  age: 18,
  city: "New York" // 空格解析成了 +
});

console.log(params.toString()); 
// 输出: name=%E5%BC%A0%E4%B8%89&age=18&city=New+York
```

注意：
+ URL 标准，查询字符串里的 `+` 等同于*空格*

### 增删改查
```js
const p = new URLSearchParams("?id=123");

p.append("type", "admin"); // 添加：?id=123&type=admin
p.set("id", "456");        // 修改：?id=456&type=admin
p.has("type");             // 检查：true
p.delete("id");            // 删除
```

### 重复键
处理数组，也没问题
```js
const p = new URLSearchParams();
p.append("tags", "js");
p.append("tags", "python");
console.log(p.toString()); // tags=js&tags=python
```