# Diffbot

[Diffbot](https://docs.diffbot.com/docs/getting-started-with-diffbot) 是一套基于机器学习的产品，可以轻松地结构化网络数据。

Diffbot的[Extract API](https://docs.diffbot.com/reference/extract-introduction)是一个服务，可以从网页中结构化和规范化数据。

与传统的网络抓取工具不同，`Diffbot Extract`不需要任何规则来读取页面上的内容。它使用计算机视觉模型将页面分类为20种可能的类型，然后将原始的HTML标记转换为JSON。生成的结构化JSON遵循一致的[基于类型的本体论](https://docs.diffbot.com/docs/ontology)，这使得可以轻松地从具有相同模式的多个不同网络来源提取数据。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/integrations/document_loaders/diffbot.ipynb)

## 概述

本指南介绍了如何使用[Diffbot Extract API](https://www.diffbot.com/products/extract/)从一组URL中提取数据，转换为我们可以在下游使用的结构化JSON。

## 设置

首先安装所需的软件包。

```python
%pip install --upgrade --quiet langchain-community
```

Diffbot的Extract API需要一个API令牌。按照以下说明[获取免费的API令牌](/docs/integrations/providers/diffbot#installation-and-setup)，然后设置一个环境变量。

```python
%env DIFFBOT_API_TOKEN 替换为您的令牌
```

## 使用文档加载器

导入DiffbotLoader模块，并使用URL列表和您的Diffbot令牌实例化它。

```python
import os
from langchain_community.document_loaders import DiffbotLoader
urls = [
    "https://python.langchain.com/",
]
loader = DiffbotLoader(urls=urls, api_token=os.environ.get("DIFFBOT_API_TOKEN"))
```

使用`.load()`方法，您可以查看加载的文档。

```python
loader.load()
```

```output
[Document(page_content="LangChain是一个由大型语言模型（LLMs）驱动的应用程序开发框架。\nLangChain简化了LLM应用程序生命周期的每个阶段：\n开发：使用LangChain的开源构建块和组件构建您的应用程序。利用第三方集成和模板快速启动。\n生产化：使用LangSmith检查、监视和评估您的链，以便您可以持续优化和自信地部署。\n部署：使用LangServe将任何链转换为API。\nlangchain-core：基本抽象和LangChain表达语言。\nlangchain-community：第三方集成。\n合作伙伴包（例如langchain-openai、langchain-anthropic等）：一些集成已进一步拆分为自己的轻量级包，仅依赖于langchain-core。\nlangchain：构成应用程序认知架构的链、代理和检索策略。\nlanggraph：通过将步骤建模为图中的边缘和节点，使用LLMs构建健壮且有状态的多参与者应用程序。\nlangserve：将LangChain链部署为REST API。\n更广泛的生态系统包括：\nLangSmith：一个开发者平台，让您可以调试、测试、评估和监视LLM应用程序，并与LangChain无缝集成。\n入门\n我们建议您按照我们的快速入门指南，通过构建第一个LangChain应用程序来熟悉框架。\n请参阅此处的说明，了解如何安装LangChain、设置您的环境并开始构建。\n注意\n这些文档侧重于Python LangChain库。请查看此处的JavaScript LangChain库文档。\n用例\n如果您想构建特定内容或更喜欢动手学习者，请查看我们的用例。这些是常见端到端任务的演练和技术，例如：\n使用RAG进行问题回答\n提取结构化输出\n聊天机器人\n等等！\n表达语言\nLangChain表达语言（LCEL）是许多LangChain组件的基础，是一种声明性的组合链条的方式。LCEL从第1天开始设计，支持将原型投入生产，无需进行任何代码更改，从最简单的“提示+LLM”链到最复杂的链。\n入门：LCEL及其优势\n可运行接口：LCEL对象的标准接口\n基元：LCEL包含的更多基元\n等等！\n生态系统\n🦜🛠️ LangSmith\n跟踪和评估您的语言模型应用程序和智能代理，帮助您从原型转向生产。\n🦜🕸️ LangGraph\n使用LLMs构建有状态的多参与者应用程序，建立在LangChain基元之上（并打算与之一起使用）。\n🦜🏓 LangServe\n将LangChain可运行和链作为REST API部署。\n安全\n阅读我们的安全最佳实践，确保您在LangChain上安全开发。\n其他资源\n组件\nLangChain为许多不同组件提供标准的可扩展接口和集成，包括：\n集成\nLangChain是与我们的框架集成并在其之上构建的丰富工具生态系统的一部分。查看我们不断增长的集成列表。\n指南\n使用LangChain开发的最佳实践。\nAPI参考\n前往参考部分，查看LangChain和LangChain实验性Python包中所有类和方法的完整文档。\n贡献\n查看开发人员指南，了解有关贡献的准则以及设置开发环境的帮助。\n通过提供对此文档页面的反馈来帮助我们。", metadata={'source': 'https://python.langchain.com/'})]
```

## 将提取的文本转换为图形文档

结构化页面内容可以通过 `DiffbotGraphTransformer` 进一步处理，将实体和关系提取到图形中。

```python
%pip install --upgrade --quiet langchain-experimental
```

```python
from langchain_experimental.graph_transformers.diffbot import DiffbotGraphTransformer
diffbot_nlp = DiffbotGraphTransformer(
    diffbot_api_key=os.environ.get("DIFFBOT_API_TOKEN")
)
graph_documents = diffbot_nlp.convert_to_graph_documents(loader.load())
```

要继续将数据加载到知识图中，请参考[`DiffbotGraphTransformer`指南](/docs/integrations/graphs/diffbot/#loading-the-data-into-a-knowledge-graph)。

```