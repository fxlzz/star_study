>  这套代码，不能适用于串行请求的重试，需保证代码中，请求顺序确实是*串行* 的
>  【优化】串行重试： 加特殊配置项 + 请求队列

这套方案是为了解决什么问题？
可见：  [Jwt 过期时间](../../interview/计算机网络.md#Jwt%20过期时间)

## 后端部分 
尚可优化的点：
1. 统一校验 token 中间件
2. 统一响应结构
3. 统一错误处理中间件

```js
const express = require("express");
const jwt = require("jsonwebtoken");
const cookieParser = require("cookie-parser");

const app = express();

// 模拟环境变量
const ACCESS_KEY = "aofiu23879y6219j01jlj";
const REFRESH_KEY = "asod187yuhi237huiy3487";

// 模拟数据库 用户数据
const USER_MSG = [{ id: 101, username: "admin", password: "admin", role: "editor" }];

// 模拟 refresh_token 存入数据库
const refreshData = [];

app.use(express.json());
app.use(cookieParser());

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "http://localhost:5173");
  res.header("Access-Control-Allow-Credentials", "true");
  res.header("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS");
  res.header("Access-Control-Allow-Headers", "Content-Type, Authorization");

  next();
});

app.post("/login", (req, res) => {
  /**
   * 1. 获取用户数据 --- 根据登录信息在数据库中查找
   * 2. 组装 payload --- 创建 AT、RT
   * 3. AT 返回给前端，RT 设置在 cookie 中（加入数据库）
   */
  const { username, password } = req.body;
  const user = USER_MSG.find((u) => u.username === username);
  if (!user || user.password !== password) {
    return res.status(401).json({ success: false, message: "账号或密码错误" });
  }
  
  const payload = {
    id: user.id,
    role: user.role,
  };
  const access_token = jwt.sign(payload, ACCESS_KEY, { expiresIn: "10s" });
  const refresh_token = jwt.sign(payload, REFRESH_KEY, { expiresIn: "7d" });

  res.cookie("refresh_token", refresh_token, {
    httpOnly: true,
    sameSite: "strict", // 防止 CSRF 攻击
    maxAge: 7 * 24 * 60 * 60 * 1000,
  });

  // 模拟 RT 存放在数据库中
  refreshData.push(refresh_token);
  
  return res.status(200).json({
    success: true,
    data: { access_token },
    message: "登录成功",
  });
});

  

app.get("/refresh", (req, res) => {
  /**
   * 1. 判断 RT 是否在数据库中存在 --- 从 cookies 中获取
   * 2. 判断 RT 是否过期
   * 3. 组装 payload --- 用户信息来自 RT 中
   * 4. 返回新的 AT
   */
  const { refresh_token } = req.cookies;
  if (!refresh_token || !refreshData.includes(refresh_token)) {
    return res.status(403).json({ success: false, message: "用户登录已过期" });
  }

  return jwt.verify(refresh_token, REFRESH_KEY, (err, user) => {
    if (err) {
      return res.status(403).json({ success: false, message: "用户登录已过期" });
    }
    
    const payload = {
      id: user.id,
      role: user.role,
    };
    const access_token = jwt.sign(payload, ACCESS_KEY, { expiresIn: "10s" });

    return res.status(200).json({
      success: true,
      data: { access_token },
    });
  });
});

app.get("/data", (req, res) => {
  /**
   * 1. 判断是否携带 token authorization
   * 2. 校验 AT 是否过期
   */
  const author = req.headers.authorization;
  if (!author) {
    return res.status(401).json({ success: false, message: "未传 token" });
  }
  
  const access_token = author.split(" ")[1];
  
  return jwt.verify(access_token, ACCESS_KEY, (err) => {
    if (err) {
      return res.status(401).json({ success: false, message: "用户登录已过期" });
    }

    const mock = [
      { id: 1, name: "tom", age: 18 },
      { id: 2, name: "jack", age: 29 },
      { id: 3, name: "marry", age: 38 },
      { id: 4, name: "mike", age: 19 },
    ];

    return res.status(200).json({
      success: true,
      data: mock,
      message: "获取数据成功",
    });
  });
});

app.listen(3003, () => {
  console.log("服务启动，端口 3003");
});
```

## 前端部分
### 封装 axios
>  默认无顺序重试 401 请求

尚可优化点：
	支持少量请求，按顺序发送 --- 支持配置项、请求队列
	 
```js
import axios from "axios";
import { message } form "antd";

const request = axios.create({
  baseURL: "http://localhost:3003",
  timeout: 5000,
  withCredentials: true, // RT 存储 cookies 中，请求需携带
});

let refreshPromise = null; // 处理多个请求同时过期

request.interceptors.request.use((config) => {
  /**
   * 1. 设置 token --- login 后存放在 localStorage
   * 2. 请求放行
   */
  const token = localStorage.getItem("access_token");
  if (token) {
    config.headers = config.headers || {};
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});


request.interceptors.response.use(
  (response) => response.data,
  async (error) => {
    const originalRequest = error.config;
    
    // 单独处理 login 接口
    if (error.response?.status === 401 && originalRequest.url === "/login") {
      const data = error.response?.data;
      if (!data?.success) {
        return message.error(data?.message);
      }
    }
    
    if (error.response?.status === 401 && originalRequest && !originalRequest._retry) 
{
      originalRequest._retry = true; // 失败的请求最多只帮你重试一次（防止token刷新失败、
后端异常，导致请求不断重发的问题）

      try {
        if (!refreshPromise) {
          refreshPromise = axios
            .get("http://localhost:3003/refresh", { withCredentials: true })
            .then((res) => {
              const accessToken = res.data?.data?.access_token;

              if (!accessToken) {
                throw new Error("用户token已过期");
              }

              localStorage.setItem("access_token", accessToken);
              return accessToken;
            })
            .finally(() => {
              refreshPromise = null;
            });
        }

        const accessToken = await refreshPromise;
        originalRequest.headers = originalRequest.headers || {};
        originalRequest.headers.Authorization = `Bearer ${accessToken}`;

        return request(originalRequest);
        
      } catch (refreshError) {
      	// RT 过期
        localStorage.clear();
        message.error("token过期，请重新登录", 1.5);
        setTimeout(() => {
          window.location.href = "/login";
        }, 1000);
        return Promise.reject(refreshError);
      }
    }
    return Promise.reject(error);
  },
);

export default request;
```

### 页面请求
```jsx
import { useState } from "react";
import request from "./utils/request";

const Login = () => {
  const [form, setForm] = useState({
    username: "admin",
    password: "admin",
  });

  const [list, setList] = useState([]);

  const handleChange = (e, type) => {
    setForm({
      ...form,
      [type]: e.target.value,
    });
  };

  const getList = async () => {
    const res = await request({
      url: "/data",
      method: "get",
    });

    return res.data || [];
  };

  const onSubmit = async () => {
    const res = await request({
      url: "/login",
      method: "post",
      data: {
        username: form.username,
        password: form.password,
      },
    });

    const accessToken = res?.data?.access_token;
    
    if (!accessToken) return;
    localStorage.setItem("access_token", accessToken);
  };

  // 模拟一个组件几个请求，同时 token 失效
  const runConcurrentRequests = async () => {
    try {
      // 立即并行发起 3 个请求，返回 3 个 Promise
      const tasks = Array.from({ length: 3 }, () => getList());
      const results = await Promise.all(tasks);

      setList(results[0] || []);
    } catch (error) {
      console.error("并发请求失败", error);
    }
  };

  return (
    <div style={{ padding: 24 }}>
      <div>
        Username:
        <input type="text" value={form.username} onChange={(e) => handleChange(e, "username")} />
      </div>
      <div style={{ marginTop: 12 }}>
        Password:
        <input
          type="password"
          value={form.password}
          onChange={(e) => handleChange(e, "password")}
        />
      </div>
      <div style={{ display: "flex", gap: 12, marginTop: 16 }}>
        <button onClick={onSubmit}>1. Login</button>
        <button onClick={runConcurrentRequests}>2. Concurrent /data</button>
      </div>
      <div style={{ marginTop: 12, color: "#666" }}>
        Demo flow: login, wait 10 seconds for AT expiry, then click concurrent request.
      </div>
      <hr style={{ margin: "24px 0" }} />
      <div>
        <h3>List</h3>
        <ul>
          {list.map((item) => (
            <li key={item.id}>
              {item.name} - {item.age}
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
};

export default Login;
```