==LLM==：large language model 大语言模型

==RAG==：retrieval-augmented generation 检索增强生成

==MCP==：model context protocol 模型上下文协议，利用 JSON-RCP 格式传输，在 agent 外部调用服务，MCP 服务端在将返回交给 LLM 使用

==Function Calling==：函数调用，用 JSON 数据描述调用函数的参数、输入、输出等，需要在内部手动调用，再将返回结果交给 LLM

RAG 、MCP、Function Calling 都可以理解是为了增强 LLM 的能力而诞生的增强工具

---

==agent==：可以理解为拥有更多功能或能力的 LLM（比如，deepseek、豆包的一些联网功能丰富了 llm）

==Text Models==： 非对话模型 不能进行多轮对话（没有上下文记忆）

==Chat Models==：对话模型 可进行多轮对话（拥有上下文记忆）

==Embedding Modells== 嵌入模型：该模型可将文本/图像/语言向量化

==token==：可以理解为提示词的一小段词（1token = 1-1.8 个汉字/3-4 个英文）

==langchain==：一个 AI 应用开发框架

==prompt==：提示词，简单理解为用户输入

==fine-tuning==：微调

==ROI==：return on investment 投资召回率