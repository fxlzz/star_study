前言：之前学过一遍 node.js 时间也过的挺久了，25-8-21 今天来复习一下，写点笔记

**node.js 是什么呢？**

是一个跨平台、开源的 js 运行时环境，允许 js 在服务端运行。也是单线程的，有事件循环机制，异步处理不会阻塞。

**学习 node.js 的目的是什么呢？**

- 很多**构建工具**都是基于 node.js 的 --> webpack、vite 等 --> 理解工程化
- 很多**效能工具**也是基于 node.js 的 --> 性能工具、内存检测、AI 效能工具(MCP server)等 --> 自主开发
- 独立开发 --> express、kao --> 自己搭建服务器
- 多学一门技能不压身 --> 提高自己的竞争力
- ...

**其常见用途：**

- **Web 服务器**（例如使用 Express 框架）
- **实时应用**（例如聊天室、在线游戏）
- **API 服务**（例如 RESTful API 或 GraphQL）
- **命令行工具**（CLI 工具）

# 全局对象 process

一些常见的 process 属性

```js
console.log(global); // node.js 中的全局对象 就像浏览器中的 window 对象
console.log("目录名：", __dirname); // 获得当前模块的目录名 D:\study\html文件夹\review\Node\全局对象
console.log("文件名", __filename); // 获得当前文件的路径  D:\study\html文件夹\review\Node\全局对象\process.js
console.log("当前命令行：", process.cwd()); // D:\study\html文件夹\review\Node\全局对象

setTimeout(() => {
  console.log("abc");
}, 1000);

process.exit(); // 强制退出程序 同步的

console.log(process.argv); // 获得命令行后的参数，返回的是数组
/** 
  [
    'C:\\Program Files\\nodejs\\node.exe',
    'D:\\study\\html文件夹\\review\\Node\\全局对象\\process.js',
    'hello',
    'world'
  ]
*/

console.log(process.platform); // 标识编译 Node.js 二进制文件的操作系统平台的字符串 win32

console.log(process.pid); // 输出本文件运行的进程端口

process.kill(10428); // 手动结束某个进程

const paths = process.env.path; // 返回用户系统环境的对象
console.log(paths.split(";"));

// 标准输入输出流 每个进程都会有输入输出两个接口
process.stdout.write("hello, world"); // 标准输出流
process.stdin.on("data", (data) => {
  const resp = `AI: ${data}`;
  process.stdout.write(resp);
}); // 标准输入流
```

# 内置对象

## 操作系统 os

```js
// 一些关于操作系统的操作
const os = require("os");
console.log(os.EOL); // 展示不同操作系统的换行符 windows --> \r\n linux --> \n

console.log(os.arch()); // 返回编译node.js二进制文件的操作系统架构 x64
console.log(process.arch); // x64

console.log(os.cpus()); // 返回cpu的信息
console.log(os.cpus().length); // 12 返回cpu的数量

console.log(os.freemem()); // 返回内容的字节数

console.log(os.homedir()); // 返回当前用户的主路径 C:\Users\晓兰

console.log(os.hostname()); // 返回主机名 LAPTOP-MA30IBDF

console.log(os.tmpdir()); // 返回终端系统的temp文件夹目录 C:\Users\晓兰\AppData\Local\Temp
```

## 路径处理 path

```js
const path = require("path");

// basename(url, suffix) ==> 返回 url 最后一段文件路径(无suffix)
//                       ==> 去除 suffix 返回最后一段路径
const basename = path.basename("a/sdff/fg/e/ds/f/asd.html"); // asd.html
const basename = path.basename("fdg/dfgdfg/adfaf/fdgdfgd/a.html", ".html"); // a

console.log(path.sep); // 获得路径连接符 windows --> \ linux --> /
console.log("a\\b\\c\\index.html".split(path.sep)); // [ 'a', 'b', 'c', 'index.html' ]

console.log(path.delimiter); // 获得分隔路径的符号 window --> ; linux --> :
console.log(process.env.path.split(path.delimiter));

const dir = path.dirname("a/b/c/d");
console.log(dir); // a/b/c 返回目录路径

const ext = path.extname("a/b/c/a.js");
console.log(ext); // .js 获得扩展名

const fullpath = path.join("a/b", "../", "d.js");
console.log(fullpath); // a\d.js 用 path.seq 的结果拼接路径

const absPath = path.resolve(__dirname, "./a.js");
console.log(absPath); // d:\study\html文件夹\review\Node\内置库\a.js

// path.resolve(...path) 给一段路径，返回绝对路径
// 处理逻辑：从右到左处理路径 -> 遇到相对路径 -> 继续向左处理
//                          -> 遇到绝对路径 -> 直接返回（左边的路径都不看了）+ 右边的连接的相对路径
//                          -> 一直没遇到绝对路径 -> 返回当前工作目录和右边拼接的相对路径
console.log(path.resolve("/foo", "/bar", "bar")); // \bar\bar
console.log(path.resolve("/foo/bar", "/tmp/file/")); // \tmp\file
console.log(path.resolve("a", "b/png/", "../gif/image.gif")); // currentWorking\a\b\gif\image.gif
console.log(path.resolve()); // 返回当前工作目录路径
```

## URL 处理

```
┌────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                              href                                              │
├──────────┬──┬─────────────────────┬────────────────────────┬───────────────────────────┬───────┤
│ protocol │  │        auth         │          host          │           path            │ hash  │
│          │  │                     ├─────────────────┬──────┼──────────┬────────────────┤       │
│          │  │                     │    hostname     │ port │ pathname │     search     │       │
│          │  │                     │                 │      │          ├─┬──────────────┤       │
│          │  │                     │                 │      │          │ │    query     │       │
"  https:   //    user   :   pass   @ sub.example.com : 8080   /p/a/t/h  ?  query=string   #hash "
│          │  │          │          │    hostname     │ port │          │                │       │
│          │  │          │          ├─────────────────┴──────┤          │                │       │
│ protocol │  │ username │ password │          host          │          │                │       │
├──────────┴──┼──────────┴──────────┼────────────────────────┤          │                │       │
│   origin    │                     │         origin         │ pathname │     search     │ hash  │
├─────────────┴─────────────────────┴────────────────────────┴──────────┴────────────────┴───────┤
│                                              href                                              │
└────────────────────────────────────────────────────────────────────────────────────────────────┘
(All spaces in the "" line should be ignored. They are purely for formatting.)
```

```js
const url = new URL("https://example.org:81/foo?a=1&b=2#abc");
console.log(url.protocol); // https:
console.log(url.host); // example.org:81
console.log(url.hostname); // example.org
console.log(url.port); // 81
console.log(url.pathname); // /foo
console.log(url.search); // ?a=1&b=2
console.log(url.hash); // #abc
console.log(url.origin); // https://example.org:81 “源” ：协议+域名+端口
```

## 工具库

```js
const util = require("util");
const obj1 = {
  a: 1,
  b: {
    c: 3,
    d: {
      e: 5,
    },
  },
};

const obj2 = {
  a: 1,
  b: {
    c: 3,
    d: {
      e: 5,
    },
  },
};

console.log(util.isDeepStrictEqual(obj1, obj2)); // true
```

# 文件 IO

读写文件，每个方法都有三种实现方式（回调函数、promise【推荐】、同步【最好不用，容易阻塞 js 进程】）

## 读

1. 读**目录下的文件名** `readdir(absoluatePath, [option-encode])`

```js
const fs = require("fs");
const path = require("path");

/**
 * 返回该目录下的所有文件名
 * @param {*} absolutePath 目录绝对路径
 * @returns {promise}
 */
const readdirFunc = async (absolutePath) => {
  const paths = [];
  const res = await fs.promises.readdir(absolutePath);
  for (const r of res) {
    const p = path.resolve(absolutePath, r);
    const isdir = await isDirectory(p);
    if (isdir) {
      const result = await readdirFunc(p);
      paths.push(result);
    } else {
      paths.push(r);
    }
  }
  return paths;
};

const dirPath = path.resolve(__dirname, "./myfiles");
readdirFunc(dirPath).then((res) => console.log(res));

/**
 * 判断文件是否是目录
 * @param {*} absolutePath
 * @returns {promise}
 */
const isDirectory = async (absolutePath) => {
  const stat = await fs.promises.stat(absolutePath);
  return stat.isDirectory();
};
```

2. 读**文件内容** `readFile(filename, [option-encode])`

```js
const fs = require("fs");
const path = require("path");

const fliePath = path.resolve(__dirname, "./test.txt");

// 方法一: 用回调函数的方式
fs.readFile(fliePath, "utf-8", (err, data) => {
  console.log(data);
});

// 方式二: 使用异步 promises
// 返回的是一个promise
async function test() {
  const res = await fs.promises.readFile(fliePath, "utf-8");
  console.log(res);
}
test();

// 方法三: 用同步的方式
// 但是同步的方法最好不要用,容易阻塞js的进程
const res = fs.readFileSync(fliePath, "utf-8");
console.log(res);
```

## 写

1. 创建**目录**`mkdir(dirpath)`

```js
const fs = require("fs");
const path = require("path");
const dirPath = path.resolve(__dirname, "./myfiles/");

async function test() {
  try {
    await fs.promises.mkdir(dirPath);
    console.log("创建目录成功");
  } catch (error) {
    console.log(error.message);
  }
}

test();
```

2. 创建**文件**`wirteFile(filePath, data, [options])`

```js
const fs = require("fs");
const path = require("path");

const filePath = path.resolve(__dirname, "./test.txt");
fs.promises.writeFile(filePath, "你好呀~", { flag: "w" }).then(() => {
  console.log("搞定了");
});

// fs.writeFile(filename, content, [option])
// option: encode -> 'utf-8'
//         flag -> 'a' : 追加
//              -> 'r' : 读文件
//              -> 'w' : 写入操作，如果文件不存在，则创建文件；如果文件存在，覆盖原来内容
```

## 查看文件属性 -> 可以用于判断是文件还是目录

fs.promises.stat(filename)

```js
const fs = require("fs");
const path = require("path");
const filename = path.resolve(__dirname, "./myfiles/1.jpg");

async function test() {
  const stat = await fs.promises.stat(filename);
  console.log(stat);

  // 可以用来判断是不是 目录 / 文件
  console.log("是否是目录", stat.isDirectory());
  console.log("是否是文件", stat.isFile());
}

test();
```

## 练习：复制文件

```js
const fs = require("fs");
const path = require("path");

/**
 * 复制文件
 * @param {*} copypath 被复制文件绝对路径
 */
const copy = async (copypath) => {
  try {
    const data = await fs.promises.readFile(copypath);
    const extname = path.extname(path.basename(copypath));
    const filename = path.basename(copypath, extname);

    const copiedPath = path.resolve(path.dirname(copypath), `./${filename}.copy${extname}`);
    await fs.promises.writeFile(copiedPath, data);
  } catch (error) {
    throw new Error(error.message);
  }
};

const filename = "./myfiles/1.jpg";
// const filename = "./test.txt";
const basePath = path.resolve(__dirname, filename);

copy(basePath)
  .then(() => console.log("已完成"))
  .catch((err) => console.log(err));
```