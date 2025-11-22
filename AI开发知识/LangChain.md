> 用来做 AI 应用程序的框架，可以做 agent、RAG 等， 内置了不少的组件，可以方便开发者去调用，各家 LLM 的 api

广义的 LangChain 包含了 LangGraph（进一步封装了 LangChain 中的 api） 和 LangSmith（用于监控、debug 使用 LangChain 开发中遇到的一些问题）

[https://docs.langchain.com/oss/javascript/langchain/overview](https://docs.langchain.com/oss/javascript/langchain/overview)

js 呢，用 langChain4j 这个框架，提供了 js 和 ts 的版本

那么话不多说，咱们直接开始 langChain 世界中的 helloworld

其实都是调 api，只不过咱之前调的是，LLM 的 api 现在调的是 langChain 给咱封装好的 api

1. 先安装

```bash
pnpm add @langchain/openai
```

2. 创建一个对话模型

```js
import { ChatOpenAI } from "@langchain/openai";

// 配置api_key可以通过配置文件/系统变量/硬编码的方式配置

const model = new ChatOpenAI({ 
  model: "gpt-4o-mini", // 模型名
  temperature: 0, // 控制输出的随机性与创建性的参数
  maxTokens: undefined, // 最大token
  timeout: undefined, // 请求时间
});

const response = await model.invoke("Hello, world!");
console.log(response.content);
```

temperature:

- 精确模式（0.5 或更低）生成文本更加安全可靠，但缺乏创意性
- 平衡模式（0.8）生成文本通过有一定的多样性，但又能保持较好的连贯性和准确性
- 创意模式（1）生成文本更加有创意，但也更容易出现语法错误或不合逻辑的内容

这里是以 openai 为例，学习使用阿里云的千问大模型，都是差不多的用法

可以前往 [https://docs.langchain.com/oss/javascript/integrations/chat#chat-completions-api](https://docs.langchain.com/oss/javascript/integrations/chat#chat-completions-api) 这个地址，找到对应的模型，参考一下即可。国内支持，阿里云、百度、腾讯、deepseek 等

# 千问大模型

```bash
pnpm add @langchain/community @langchain/core
```

创建对话模型实例

```js
import { ChatAlibabaTongyi } from "@langchain/community/chat_models/alibaba_tongyi";
import { HumanMessage } from "@langchain/core/messages";
// 加载配置文件
import dotenv from "dotenv";
dotenv.config();

const qwen = new ChatAlibabaTongyi({
  model: "qwen-plus",
  alibabaApiKey: process.env.ALIBABA_API_KEY,
  temperature: 0.8,
});

const test = async () => {
  const messages = [new HumanMessage("你是谁？")];
  const response = await qwen.invoke(messages);
  console.log(response);
};

test();
```

对话模型，返回的是 `AIMessage`格式的数据，非对话模型返回的是`string`类型的数据

```js
AIMessage {
  "content": "我是通义千问，阿里巴巴集团旗下的超大规模语言模型。我能够回答问题、创作文字，比如写故事、写公文、写邮件、写剧本等等，还能回答各种问题，提供帮助和信息。如果你有任何问题或需要帮助，尽管告诉我！",
  "additional_kwargs": {},
  "response_metadata": {
    "tokenUsage": {
      "promptTokens": 11,
      "completionTokens": 57,
      "totalTokens": 68
    }
  },
  "tool_calls": [],
  "invalid_tool_calls": []
}
```

# Message

在 langChain 的世界中，大致分为 4 种消息格式

[https://docs.langchain.com/oss/javascript/langchain/messages](https://docs.langchain.com/oss/javascript/langchain/messages)

- SystemMessage
- HumanMessage
- AIMessage
- ToolMessage

可接受字符串、数组的形式传给大模型

```js
import { initChatModel, HumanMessage, SystemMessage } from "langchain";

// 有两种方式创建大模型实例，这是其二
const model = await initChatModel("gpt-5-nano");

const systemMsg = new SystemMessage("You are a helpful assistant.");
const humanMsg = new HumanMessage("Hello, how are you?");

const messages = [systemMsg, humanMsg];
const response = await model.invoke(messages);  // Returns AIMessage
```

# 调用方式

感觉都不用写什么，官方文档说得很明白呀，又省事了~~

## invoke()

[https://docs.langchain.com/oss/javascript/langchain/models#invoke](https://docs.langchain.com/oss/javascript/langchain/models#invoke)

结果一次性返回

```js
const response = await model.invoke("Why do parrots have colorful feathers?");
console.log(response);
```

## stream()

[https://docs.langchain.com/oss/javascript/langchain/models#stream](https://docs.langchain.com/oss/javascript/langchain/models#stream)

流式输出，结果会一段一段的返回，返回的是一个可迭代对象 iterator，这里也就说明了，我们可以使用 for of 这样的来遍历每次结果

```js
const stream = await model.stream("Why do parrots have colorful feathers?");
for await (const chunk of stream) {
  console.log(chunk.text)
}
```

那其实返回的每一个 chunk 都是`AIMessageChunk`

```js
AIMessageChunk {
  "content": "嗨",
  "additional_kwargs": {},
  "response_metadata": {},
  "tool_calls": [],
  "tool_call_chunks": [],
  "invalid_tool_calls": []
}
```

比如：

```js
const systemMsg = `你是一个智力超群的AI助手，名字是star`;
const messages = [systemMsg, new HumanMessage("你可以帮我什么？")];

const chunks = await qwen.stream(messages);
for await (const chunk of chunks) {
  process.stdout.write(chunk?.content);
}
```

那么输出结果如下：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1763282804148-3e997cf1-9eab-4830-890b-3cb09ac8043e.png?x-oss-process=image%2Fcrop%2Cx_0%2Cy_75%2Cw_740%2Ch_364)

没输出完，是因为设置了 `maxToken：100`

### SSE

还可以支持 SSE 输出，使用`streamEvents`调用即可

```js
const stream = await model.streamEvents("Hello");
for await (const event of stream) {
  // 这是通过type或元数据进行筛选
    if (event.event === "on_chat_model_start") {
        console.log(`Input: ${event.data.input}`);
    }
    if (event.event === "on_chat_model_stream") {
        console.log(`Token: ${event.data.chunk.text}`);
    }
    if (event.event === "on_chat_model_end") {
        console.log(`Full message: ${event.data.output.text}`);
    }
}
```

暂时更多的，咱先不研究，先知道它可以支持 SSE 即可

## batch()

可以批量处理多个问题，返回的都是 `AIMessage`

比如：

```js
const messages = [
    "1+1等于什么？",
    "回溯算法致力于处理哪种问题？",
  ];
  const responses = await qwen.batch(messages, <options>);
  for (const response of responses) {
    console.log(response.content);
  }
```

还可以配置一次**至多**处理多少个问题，`maxConcurrency`

# prompt engineering

提示词相关配置，在 langSmith 中

根据官方描述，prompt enginerring 是非常重要的。改变 LLM 行为的方式，一个是 fine-tuning（可能不一定能达到最终满意效果，并且成本较高），另一个就是 prompt engineering（很简单也很高效）

其实，没有很高级

就是提前配置一些模版给到 LLM，prompt template 类似于模板字符串，可以插入变量的字符串

---

一些 prompt type：

- PromptTemplate：生成字符串提示
- ChatPromptTemplate：组合各种角色的消息模版
- xxxMessagePromptTemplate：SystemMessagePromptTemplate、HumanMessagePromtTemplate、AIMessagePromptTemplate、ChatMessagePromptTemplate 等
- FewShotPromptTemplate：样本提示词模版

API 文档

[https://v03.api.js.langchain.com/classes/_langchain_core.prompts.ChatPromptTemplate.html#partialVariables](https://v03.api.js.langchain.com/classes/_langchain_core.prompts.ChatPromptTemplate.html#partialVariables)

## PromptTemplate

创建提示词模版： 

```js
import { PromptTemplate } from "langchain/prompts";

// 方法一：使用 inputVariables 声明变量，在format调用时，再赋值
const prompt = new PromptTemplate({
  template: "hello {name}, you can help me to do {somethings}",
  inputVariables: ["name", "somethings"]
})

// 方法二： 赋初始值 partialVariables
const prompt = new PromptTemplate({
    template: "hi!, what are you doing, you are help me to solve the problem {problem}",
    inputVariables: ["problem"],
    partialVariables: { problem: "1 + 1 = ?" },
});

// 方法三：同时声明并赋值 partial
const prompt = await PromptTemplate.fromTemplate(
    "hi!, what are you doing, you are help me to solve the problem {problem}"
).partial({ problem: "1 + 1 = ?" });
console.log(prompt);
```

### 调用

可以定义提示词模板时，顺便赋值 `partialVariables` ，也可以调用提示词时（format、invoke）赋值

```js
async function test() {
  // 方法一：format()
  const formattedPrompt = await prompt.format({
    foo: 'banana'
  });

  // 方法二：invoke()
  const formattedPrompt = await prompt.invoke({
    foo: 'apple'
  })

  // 方法三: formatPromptValue()
  const response = await prompt.formatPromptValue({xxx: xxx});
}
test()
```

## ChatPromptTemplate
### 实例化
传递一个二维数组，`[role: content]`
role的取值：
+ ai
+ human
+ system
+ tool

```js
 const prompt = ChatPromptTemplate.fromMessages(
    [
      ["ai", "you are an helpful assistant, {name}"],
      ["human", "my question is {problem}"],
    ],
    { inputVariables: ["name", "problem"] }
  );

const response = await prompt.invoke({ name: "star", problem: "1+1=?" });
console.log(response);
```

invoke() 返回值：
```js
ChatPromptValue {
  lc_serializable: true,
        "additional_kwargs": {},
        "additional_kwargs": {},
        "additional_kwargs": {},
        "additional_kwargs": {},
        "additional_kwargs": {},
        "additional_kwargs": {},
        "response_metadata": {},
        "tool_calls": [],
        "invalid_tool_calls": []
      },
      HumanMessage {
        "content": "my question is 1+1=?",
        "additional_kwargs": {},
        "response_metadata": {}
      }
    ]
  },
  lc_namespace: [ 'langchain_core', 'prompt_values' ],        
  messages: [
    AIMessage {
      "content": "you are an helpful assistant, star",        
      "additional_kwargs": {},
      "response_metadata": {},
      "tool_calls": [],
      "invalid_tool_calls": []
    },
    HumanMessage {
      "content": "my question is 1+1=?",
      "additional_kwargs": {},
      "response_metadata": {}
    }
  ]
}
```



### 调用
有四种调用方式
+ `invoke()` 返回值为 object 类型是 chatPromptValue
+ `format()` 返回值为 string 
+ `formatMessages()` 返回值为 array 
+ `formatPrompt()` 返回值为 object, 跟invoke()调用结果一致 

#### format()
```js
// ... 上面的内容保持不变
const response = await prompt.format({ name: "star", problem: "1+1=?" });
console.log(response);
console.log(typeof response);
```

返回值
```js
AI: you are an helpful assistant, star
Human: my question is 1+1=?
string
```

#### formatMessages()
```js
const response = await prompt.formatMessages({ name: "star", problem: "1+1=?" });
console.log(response);
console.log(typeof response);
```

返回值 - 看起来就是 invoke() 返回值的message字段的值 
```js
[
  AIMessage {
    "content": "you are an helpful assistant, star",
    "additional_kwargs": {},
    "response_metadata": {},
    "tool_calls": [],
    "invalid_tool_calls": []
  },
  HumanMessage {
    "content": "my question is 1+1=?",
    "additional_kwargs": {},
    "response_metadata": {}
  }
]
object
```

#### formatPromptValue
```js
const response = await prompt.formatPromptValue({ name: "star", problem: "1+1=?" });
console.log(response);
```

返回值 - 这个返回的就跟invoke()差不多
```js
ChatPromptValue {
  lc_serializable: true,
  lc_kwargs: {
    messages: [
      AIMessage {
        "content": "you are an helpful assistant, star",
        "additional_kwargs": {},
        "response_metadata": {},
        "tool_calls": [],
        "invalid_tool_calls": []
      },
      HumanMessage {
        "content": "my question is 1+1=?",
        "additional_kwargs": {},
        "response_metadata": {}
      }
    ]
  },
  lc_namespace: [ 'langchain_core', 'prompt_values' ],
  messages: [
    AIMessage {
      "content": "you are an helpful assistant, star",
      "additional_kwargs": {},
      "response_metadata": {},
      "tool_calls": [],
      "invalid_tool_calls": []
    },
    HumanMessage {
      "content": "my question is 1+1=?",
      "additional_kwargs": {},
      "response_metadata": {}
    }
  ]
}
```

---
总结:
看似有四种调用方式, 其实只有三种返回值`[string | array | object]`, 根据调用场景来选择使用哪种调用方式吧

chatPromptTemplate 这个类实现了 ChatPromptValueInterface 这个接口, 有一些方法可用
https://v03.api.js.langchain.com/interfaces/_langchain_core.prompt_values.ChatPromptValueInterface.html

### 方法
#### toChatMessages()

formatPromptValue() 与 invoke() 可通过调用 `toChatMessages()` 方法转化成 formatMessage() 的调用结果(会转成数组的形式)

```js
const test = async () => {
  const prompt = ChatPromptTemplate.fromMessages([
    ["ai", "hi, my name is {name}, i can help you to doing something"],
    ["human", "i need you to solve this problem {q}"],
  ]);

  const res = await prompt.invoke({ name: "star", q: "1+1=?" });
  console.log(res.toChatMessages());
};
```

返回值`
```js
[
  AIMessage {
    "content": "hi, my name is star, i can help you to doing something",
    "additional_kwargs": {},
    "response_metadata": {},
    "tool_calls": [],
    "invalid_tool_calls": []
  },
  HumanMessage {
    "content": "i need you to solve this problem 1+1=?",
    "additional_kwargs": {},
    "response_metadata": {}
  }
]
```

#### toString()
同理, 也可转化成 format() 的调用结果
```js
const prompt = ChatPromptTemplate.fromMessages([
    ["ai", "hi, my name is {name}, i can help you to doing something"],
    ["human", "i need you to solve this problem {q}"],
]);

const res = await prompt.formatPromptValue({ name: "star", q: "1+1=?" });
console.log(res.toString());
```

返回值
```js
AI: hi, my name is star, i can help you to doing something
Human: i need you to solve this problem 1+1=?
```

--- 
调用大模型没什么好说的, 直接 invoke(prompt) 调用即可

现在, 还有一种情况, chatPromptTemplate 是需要指定 role 的, 那么如果, 不知道 role 是什么, 应该怎么写呢 ---> 用占位符

### messagePlaceholder

