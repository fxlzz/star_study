Retrieval-Augmentd Generation: 检索增强生成
好端端的，为什么会产生这么一个技术呢？
原因要追溯到 LLM 的缺陷

---
## LLM 的缺陷
这一切的前提是， 理解 LLM 到底是什么？ - 可以抽象成一个函数， 能识别自然语言，通过一些算法架构和训练资料，能够推断出用户想知道的答案是什么。

主要有两个： 
+ 功能性缺陷 - 知道用户想做什么，但是 LLM 无法做到 -> Function Calling / MCP
	+ 用户想知道北京今天的天气如何？ LLM的资料库没有今天的资料 -> 所以得去调接口，获取天气
+ 知识性缺陷 - LLM 不理解用户想干什么 -> prompt engineering / RAG
	+ 用户提问公司内部资料或市面上不存在的东西 -> LLM 根本不知道用户在讲什么，很有可能出现乱回答的情况(我们称之为：幻觉) -> 最根本的原因，不就是 LLM 不知道嘛， 让它知道即可

---
RAG 解决的呢，就是 LLM 的知识性缺陷，具体一点：
+ 上下文的限制
+ 内部资料、专业性
+ ...
所以，RAG 就很适合做 企业内部知识库、个人知识库、某专业领域的专业回答等

## 基本流程
一般的 RAG 流程类似于这样：
![](assets/RAG/file-20251206105308828.png)

1. 加载资源 loader - 文档过多/大？
2. 文档拆分 split - 怎么拆分? 语义不对？
3. 嵌入 embedding - 嵌入模型
4. 存储 store - 向量数据库
5. 检索 retrieval - 怎么提升精确度？召回率？

当然上述流程，是个人理解

## 文档加载
langchain 支持市面常见的文档，所有的 loader 都继承至 BaseDocumentLoader ， 这个抽象类实现了 DocumentLoader 这个接口，提供 load 方法。 

+ load()： 一次加载所有文档
+ loadAndSplit(): 一次加载所有文档，并拆分成更小的模块

langchain 支持多模态的用户输入， 不仅仅有文档，还有web相关，
具体可看官网：
https://docs.langchain.com/oss/javascript/integrations/document_loaders

### 以 pdf 文档为例
其实在内部调的是 pdf-parse@v1 ，将 pdf 上的文字解析出来
```js
import { PDFLoader } from "@langchain/community/document_loaders/fs/pdf";
import { dirname, join } from "path";
import { fileURLToPath } from "url";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const filename = join(__dirname, "../test.pdf");

const readPDFDocument = async (filename) => {
  const loader = new PDFLoader(filename);
  const docs = await loader.load();
  console.log(docs);
};

readPDFDocument(filename);
```

返回结果：
```js
Document {
  pageContent: 'xxxx'
  metadata: {
    source: '文件地址',
    pdf: {
      version: '1.10.100',
      info: [Object],
      metadata: null,
      totalPages: 文档页数
    },
    loc: { pageNumber: 1 } // 当前页数
  },
  id: undefined
}
```

## 文档拆分
langchain 中默认使用 `Splitting recursively`  这个函数进行拆分
`pnpm install @langchain/textsplitters`

大致分为三种拆分策略：
+ 基于文本结构的 - 根据句子、段落、词语等进行拆分
+ 基于长度的
	+ 基于token的
	+ 基于文字的
+ 基于文档结构的 - 一般用于已有结构的文档，像 html、json、markdown 等

### Text structure-based
默认的文本拆分是基于： ["\n\n", "\n", " ", ""]，为了处理不同语言的结束符号。像中文的 '。' 就需要用 ASCII 码的方式加到拆分规则中.

```js
 const separators = [
    "\n\n",
    "\n",
    " ",
    ".",
    ",",
    "\uff0c", // ，
    "\u3001", // 、
    "\uff0e", // .
    "\u3002", // 。
    ",",
  ];
```

```js
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";

const splitter = new RecursiveCharacterTextSplitter({ 
	separators,
	chunkSize: 100,  // pageContent 字符大小
	chunkOverlap: 0  // 是否重叠 - 为了防止拆分语义不全
})

// 这个 document 是 Document 对象
// [{pageContent: xxx}, {...}]
const texts = docs.map((doc) => doc.pageContent); // 有可能需要处理一下
const splitDocs = splitter.createDocuments(texts);
```

返回值：
```js
[
  Document {
    pageContent: 'xxx',
    metadata: { loc: [Object] },
    id: undefined
  },
  Document {
    pageContent: 'xxx',
    metadata: { loc: [Object] },
    id: undefined
  }
  ...
]
```

### Length based
以文档长度为基于拆分，最大的好处就是 **直接** 
#### Token
```js
import { TokenTextSplitter } from "@langchain/textsplitters";

const splitter = new TokenTextSplitter({ 
	encodingName: "cl100k_base", // 编码名称
	chunkSize: 100, 
	chunkOverlap: 0 
})
const texts = splitter.splitText(document)
```

#### characters

### Document structure-based

