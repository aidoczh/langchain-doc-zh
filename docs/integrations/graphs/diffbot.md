# Diffbot

[Diffbot](https://docs.diffbot.com/docs/getting-started-with-diffbot) 是一套基于机器学习的产品，可以轻松地对网络数据进行结构化处理。

Diffbot的[自然语言处理 API](https://www.diffbot.com/products/natural-language/) 可以从非结构化文本数据中提取实体、关系和语义含义。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/integrations/graphs/diffbot.ipynb)

## 使用案例

文本数据通常包含丰富的关系和见解，可用于各种分析、推荐引擎或知识管理应用。

通过将 `Diffbot的NLP API` 与图数据库 `Neo4j` 结合使用，您可以基于从文本中提取的信息创建强大的动态图结构。这些图结构是完全可查询的，并可以集成到各种应用程序中。

这种组合可以实现以下用例：

- 从文档、网站或社交媒体源构建知识图（如[Diffbot的知识图](https://www.diffbot.com/products/knowledge-graph/)）

- 基于数据中的语义关系生成推荐

- 创建能理解实体之间关系的高级搜索功能

- 构建允许用户探索数据中隐藏关系的分析仪表板

## 概述

LangChain 提供了与图数据库交互的工具：

1. 使用图转换器和存储集成从文本构建知识图

2. 使用链进行查询创建和执行图数据库查询

3. 使用代理进行强大而灵活的图数据库交互

## 设置

首先，获取所需的软件包并设置环境变量：

```python
%pip install --upgrade --quiet  langchain langchain-experimental langchain-openai neo4j wikipedia
```

### Diffbot NLP API

`Diffbot的NLP API` 是一个从非结构化文本数据中提取实体、关系和语义上下文的工具。

这些提取的信息可以用于构建知识图。

要使用该 API，您需要从 Diffbot 获取一个[免费的 API 令牌](https://app.diffbot.com/get-started/)。

```python
from langchain_experimental.graph_transformers.diffbot import DiffbotGraphTransformer
diffbot_api_token = "DIFFBOT_API_TOKEN"
diffbot_nlp = DiffbotGraphTransformer(diffbot_api_token=diffbot_api_token)
```

这段代码获取关于 "Warren Buffett" 的维基百科文章，然后使用 `DiffbotGraphTransformer` 提取实体和关系。

`DiffbotGraphTransformer` 输出结构化数据 `GraphDocument`，可用于填充图数据库。

请注意，由于 Diffbot 每个 API 请求的[字符限制](https://docs.diffbot.com/reference/introduction-to-natural-language-api)，避免了文本分块。

```python
from langchain_community.document_loaders import WikipediaLoader
query = "Warren Buffett"
raw_documents = WikipediaLoader(query=query).load()
graph_documents = diffbot_nlp.convert_to_graph_documents(raw_documents)
```

## 将数据加载到知识图中

您需要运行一个 Neo4j 实例。一种选择是在他们的 Aura 云服务中创建一个[免费的 Neo4j 数据库实例](https://neo4j.com/cloud/platform/aura-graph-database/)。您还可以使用[Neo4j Desktop 应用程序](https://neo4j.com/download/)在本地运行数据库，或者运行一个 Docker 容器。您可以通过执行以下脚本来运行本地的 Docker 容器：

```bash
docker run \
    --name neo4j \
    -p 7474:7474 -p 7687:7687 \
    -d \
    -e NEO4J_AUTH=neo4j/pleaseletmein \
    -e NEO4J_PLUGINS=["apoc"]  \
    neo4j:latest
```

如果您使用的是 Docker 容器，需要等待一段时间数据库启动。

```python
from langchain_community.graphs import Neo4jGraph
url = "bolt://localhost:7687"
username = "neo4j"
password = "pleaseletmein"
graph = Neo4jGraph(url=url, username=username, password=password)
```

`GraphDocuments` 可以使用 `add_graph_documents` 方法加载到知识图中。

```python
graph.add_graph_documents(graph_documents)
```

## 刷新图模式信息

如果数据库的模式发生变化，您可以刷新生成 Cypher 语句所需的模式信息。

```python
graph.refresh_schema()
```

## 查询图

现在我们可以使用图 Cypher QA 链来询问图的问题。建议使用 **gpt-4** 来构建 Cypher 查询以获得最佳体验。

```python
from langchain.chains import GraphCypherQAChain
from langchain_openai import ChatOpenAI
chain = GraphCypherQAChain.from_llm(
    cypher_llm=ChatOpenAI(temperature=0, model_name="gpt-4"),
    qa_llm=ChatOpenAI(temperature=0, model_name="gpt-3.5-turbo"),
    graph=graph,
    verbose=True,
)
```

```python
chain.run("沃伦·巴菲特曾就读于哪所大学？")
```

```output
'沃伦·巴菲特曾就读于内布拉斯加大学。'
```

```python
chain.run("谁在伯克希尔·哈撒韦工作或曾经工作？")
```

```output
'查理·芒格、奥利弗·查斯、霍华德·巴菲特、苏珊·巴菲特和沃伦·巴菲特在伯克希尔·哈撒韦工作或曾经工作。'
```