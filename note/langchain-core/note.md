langchain-core 是整个 LangChain 生态系统的**心脏**和**协议层**。

它不包含具体的第三方集成（如 OpenAI API 调用逻辑），而是定义了所有其他包（如 `langchain-openai`, `langchain-community`）必须遵守的**接口（Interfaces）**、**基类（Base Classes）**以及**核心运行时（Runtime）**。

以下是对 `langchain-core` 的详细模块拆解：

### 1. 核心架构：LCEL (LangChain Expression Language)

这是 `core` 中最重要的部分。它定义了如何将不同的组件“链接”在一起。

*   **Runnables (`src/runnables`)**:
    *   这是所有 LangChain 组件的基类。无论是 Model、Prompt、Retriever 还是 Tool，它们都继承自 `Runnable`。
    *   它统一了调用方式：`.invoke()`, `.stream()`, `.batch()`。
    *   它实现了 `.pipe()` 方法，允许你写出类似 `prompt.pipe(model).pipe(parser)` 的链式代码。
    *   **关键文件**: index.ts

*   **Context (`src/context.ts`)**:
    *   在复杂的链式调用中，需要在不同步骤间传递配置（如 `callbacks` 或 `tags`）。`core` 使用类似 `AsyncLocalStorage` 的机制来管理这些上下文，确保在异步操作中不会丢失状态。
    *   **参考**: 你提供的 context.test.ts 展示了如何在嵌套的 Runnable 中获取和设置上下文变量。

### 2. 标准数据结构 (The Protocol)

为了让不同的模型提供商（OpenAI, Anthropic, Google）能够互换，`core` 定义了标准的数据格式。

*   **Messages (`src/messages`)**:
    *   定义了 `HumanMessage`, `AIMessage`, `SystemMessage` 等。
    *   **多模态标准化**: 你提供的 bedrock_converse.ts 展示了 `core` 如何处理复杂的输入（如文档、图片），并将它们转换为标准化的 `ContentBlock`。这意味着上层应用不需要关心底层是 Bedrock 还是 OpenAI，只需要处理标准的 Message 对象。

*   **Documents (`src/documents`)**:
    *   定义了 `Document` 类（包含 `pageContent` 和 `metadata`）。这是 RAG（检索增强生成）应用中数据流动的基本单位。

### 3. 抽象基类 (Base Classes)

所有的具体实现都必须继承这些类：

*   **Language Models (`src/language_models`)**:
    *   `BaseChatModel`: 定义了聊天模型必须实现 `_generate` 方法。
    *   `BaseLLM`: 定义了纯文本补全模型。
    *   这些基类处理了通用的逻辑，比如缓存（Caching）、回调（Callbacks）触发和重试逻辑。

*   **Retrievers (`src/retrievers`)**:
    *   定义了 `BaseRetriever`，规定了 `_getRelevantDocuments` 接口。

*   **Tools (`src/tools`)**:
    *   定义了 Agent 可以调用的工具的结构（名称、描述、Schema）。

### 4. 输入/输出处理 (I/O)

*   **Prompts (`src/prompts`)**:
    *   `PromptTemplate`, `ChatPromptTemplate`。负责将用户输入和变量组合成最终发送给模型的字符串或消息列表。

*   **Output Parsers (`src/output_parsers`)**:
    *   负责将模型的文本输出（String）转换为结构化数据（JSON, XML 等）。
    *   例如 `StringOutputParser` (在 `README.md` 中提到) 只是简单地提取文本。

### 5. 序列化与工具 (`src/load`, `src/utils`)

*   **Serializable (`src/load/serializable.ts`)**:
    *   LangChain 的对象大多是可序列化的。这允许将链的结构保存为 JSON，并在不同语言（Python/JS）之间共享，或者上传到 LangSmith 进行监控。

### 总结：如何阅读 `langchain-core`

如果你想深入理解 LangChain 的工作原理，建议关注以下路径：

1.  **理解执行流**: 阅读 `src/runnables` 下的代码，理解 `pipe` 是如何构造 `RunnableSequence` 的。
2.  **理解数据流**: 查看 `src/messages` 和 `src/outputs`，理解数据如何在组件间传递。
3.  **理解扩展性**: 查看 `src/language_models/chat_models.ts`，看它是如何定义标准接口，从而让 `langchain-openai` 和 `langchain-anthropic` 可以无缝切换的。

简单来说，`langchain-core` 是**规则的制定者**，而其他包是**规则的执行者**。