>  2026-4-1 复习一下 AJAX，补点笔记

对于前端而言，发送 AJAX 的两大神器
+ axios 基于 xhr
+ fetch

## Axios
>  基于 promise 的 HTTP 客户端，可以在浏览器和 nodejs 环境使用

官网： [Axios Docs](https://axios-http.com/zh/docs/intro)

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
+ `method: enmu`
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
