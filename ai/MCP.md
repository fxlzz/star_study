## 前置知识

- **sdtio** 通信协议
- **JSON-RPC** 通信格式

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1755785122977-84c86291-d1c6-4f29-8bb4-8b8532185aa4.png)

用 node.js 来模拟一下，stdio 通信：

终端模拟父进程【client】，node 服务模拟子进程【server】

```js
// 标准输入流
process.stdin.on("data", (data) => {
  process.stdout.write(`父进程发送的信息： ${data}`);
});

// 标准输出流
process.stdout.write("hello world");
```

在 stdio 协议中，父进程是什么都可以，只要能创建子进程，能监听标准输入输出即可

```js
// client.js ---> 父进程
const { spawn } = require("child_process");
const child = spawn("node", ["server.js"]);
const list = ["你好", "我很好", "I am fine, thank you. And you?"];

list.forEach((msg, index) => {
  setTimeout(() => {
    console.log(`-->${msg}`);
    // 给子进程发送信息
    child.stdin.write(msg);
  }, index * 1000);
});

// 监听子进程发送的信息
child.stdout.on("data", (data) => {
  console.log(data.toString());
});

// server.js ---> 子进程
// 监听父进程发送的信息
process.stdin.on("data", (data) => {
  process.stdout.write(`AI:  ${data}`);
});
```

## MCP

MCP(model context protocol) 模型上下文协议

如何通信：

1. stdio：本地、推荐
2. http：远程

通信格式：JSON-RPC

### 基本规范

1. 初始化（initialize）：

MCP 服务器与客户端确认眼神，看能不能使用 MCP 协议

**request**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    // MCP 版本
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "roots": {
        "listChanged": true
      },
      "sampling": {},
      "elicitation": {}
    },
    // 客户端信息
    "clientInfo": {
      "name": "ExampleClient",
      "title": "Example Client Display Name",
      "version": "1.0.0"
    }
  }
}
```

**response**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "logging": {},
      "prompts": {
        "listChanged": true
      },
      "resources": {
        "subscribe": true,
        "listChanged": true
      },
      "tools": {
        "listChanged": true
      }
    },
    // 服务器信息
    "serverInfo": {
      "name": "ExampleServer",
      "title": "Example Server Display Name",
      "version": "1.0.0"
    },
    "instructions": "Optional instructions for the client"
  }
}
```

2. 发现工具（tools/list）：

**request:**

```json
{
  "method": "tools/list",
  "params": {}
}
```

**response:**

```json
{
  "tools": [
    {
      "name": "add",
      "title": "两数之和",
      "description": "两个数相加",
      "inputSchema": {
        "type": "object",
        "properties": {},
        "additionalProperties": false,
        "$schema": "http://json-schema.org/draft-07/schema#"
      }
    },
    {
      "name": "createFile",
      "title": "创建文件",
      "description": "输入文件绝对地址和内容，创建文件",
      "inputSchema": {
        "type": "object",
        "properties": {},
        "additionalProperties": false,
        "$schema": "http://json-schema.org/draft-07/schema#"
      }
    }
  ]
}
```

3. 调用工具（tools/call）：

**request:**

```json
{
  "method": "tools/call",
  "params": {
    "name": "add",
    "arguments": {},
    "_meta": {
      "progressToken": 0
    }
  }
}
```

### 效能工具

MCP server 的调式工具：

```bash
npx @modelcontextprotocol/inspector
```