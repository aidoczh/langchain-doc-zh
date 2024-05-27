# 2Markdown

[2markdown](https://2markdown.com/) 服务可以将网站内容转换为结构化的 markdown 文件。

```python
# 你需要获取自己的 API 密钥。请参阅 https://2markdown.com/login
api_key = ""
```

```python
from langchain_community.document_loaders import ToMarkdownLoader
```

```python
loader = ToMarkdownLoader(url="/docs/get_started/introduction", api_key=api_key)
```

```python
docs = loader.load()
```

```python
print(docs[0].page_content)
```

**LangChain** 是一个用于开发由语言模型驱动的应用程序的框架。它可以实现以下应用：

- **上下文感知**：将语言模型连接到上下文来源（提示说明、少量示例、内容等）。

- **推理**：依赖语言模型进行推理（根据提供的上下文进行回答，采取什么行动等）。

该框架由几个部分组成。

- **LangChain 库**：Python 和 JavaScript 库。包含各种组件的接口和集成，用于将这些组件组合成链和代理的基本运行时，以及链和代理的现成实现。

- **[LangChain 模板](/docs/templates)**：用于各种任务的易于部署的参考架构集合。

- **[LangServe](/docs/langserve)**：用于将 LangChain 链部署为 REST API 的库。

- **[LangSmith](https://docs.smith.langchain.com)**：一个开发平台，可让您调试、测试、评估和监视基于任何 LLM 框架构建的链，并与 LangChain 无缝集成。

![Diagram outlining the hierarchical organization of the LangChain framework, displaying the interconnected parts across multiple layers.](https://python.langchain.com/assets/images/langchain_stack-f21828069f74484521f38199910007c1.svg)

这些产品共同简化了整个应用程序生命周期：

- **开发**：使用 LangChain/LangChain.js 编写您的应用程序。使用模板作为参考，快速上手。

- **生产化**：使用 LangSmith 检查、测试和监视您的链，以便您可以不断改进并自信地部署。

- **部署**：使用 LangServe 将任何链转换为 API。

## LangChain 库 [​](\#langchain-libraries "LangChain 库的直接链接")

LangChain 包的主要价值在于：

1. **组件**：用于处理语言模型的可组合工具和集成。这些组件是模块化且易于使用的，无论您是否使用 LangChain 框架的其他部分。

2. **现成的链**：用于完成更高级任务的内置组合。

现成的链使得入门变得容易。组件使得定制现有链和构建新链变得容易。

LangChain 库本身由几个不同的包组成。

- **`langchain-core`**：基本抽象和 LangChain 表达语言。

- **`langchain-community`**：第三方集成。

- **`langchain`**：组成应用程序认知架构的链、代理和检索策略。

## 入门 [​](\#get-started "入门的直接链接")

[这里](/docs/installation)是如何安装 LangChain、设置您的环境并开始构建的方法。

我们建议您按照我们的[快速入门](/docs/tutorials/llm_chain)指南，通过构建您的第一个 LangChain 应用程序来熟悉该框架。

阅读我们的[安全](/docs/security)最佳实践，以确保您在 LangChain 上安全开发。

注意

这些文档侧重于 Python LangChain 库。[点击此处](https://js.langchain.com)查看 JavaScript LangChain 库的文档。

## LangChain 表达语言 (LCEL) [​](\#langchain-expression-language-lcel "LangChain 表达语言 (LCEL) 的直接链接")

LCEL 是一种声明性的链组合方式。LCEL 从一开始就被设计用于支持将原型投入生产，无需进行任何代码更改，从最简单的“提示 + LLM”链到最复杂的链。

- **[概述](/docs/concepts#langchain-expression-language)**：LCEL 及其优势

- **[接口](/docs/concepts#interface)**：LCEL 对象的标准接口

- **[如何](/docs/expression_language/how_to)**：LCEL 的关键特性

- **[示例](/docs/expression_language/cookbook)**：完成常见任务的示例代码

## 模块 [​](\#modules "模块的直接链接")

LangChain 为以下模块提供了标准的可扩展接口和集成：

#### [模型 I/O](/docs/modules/model_io/) [​](\#model-io "模型 I/O 的直接链接")

与语言模型进行接口

#### [检索](/docs/modules/data_connection/) [​](\#retrieval "检索的直接链接")

与特定于应用程序的数据进行接口

#### [代理](/docs/tutorials/agents) [​](\#agents "代理的直接链接")

让模型根据高级指令选择使用哪些工具

## 示例、生态系统和资源 [​](\#examples-ecosystem-and-resources "示例、生态系统和资源的直接链接")

### [用例](/docs/how_to#qa-with-rag) [​](\#use-cases "用例的直接链接")

常见端到端用例的演练和技术，如：

- [文档问答](/docs/how_to#qa-with-rag)

- [聊天机器人](/docs/use_cases/chatbots/)

- [分析结构化数据](/docs/how_to#qa-over-sql--csv)

- 等等...

### [集成](/docs/integrations/providers/) [​](\#integrations "集成的直接链接")

LangChain 是丰富的工具生态系统的一部分，这些工具与我们的框架集成并在其基础上构建。查看我们不断增长的[集成](/docs/integrations/providers/)列表。

### [指南](/docs/how_to/debugging) [​](\#guides "指南的直接链接")

使用 LangChain 进行开发的最佳实践。

### [API 参考](https://api.python.langchain.com) [​](\#api-reference "API 参考的直接链接")

转到参考部分，查看 LangChain 和 LangChain 实验性 Python 包中所有类和方法的完整文档。

### [开发者指南](/docs/contributing) [​](\#developers-guide "开发者指南的直接链接")

查看开发者指南，了解有关贡献和帮助设置开发环境的指南。

转到[社区导航器](/docs/community)查找提问、分享反馈、与其他开发人员会面以及梦想 LLM 未来的地方。