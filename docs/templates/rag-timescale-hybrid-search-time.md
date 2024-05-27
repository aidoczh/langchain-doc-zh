

# 使用时序向量的 RAG 与混合搜索

这个模板展示了如何使用时序向量和自查询检索器执行相似性和时间的混合搜索。

当您的数据具有强烈的基于时间的组成部分时，这将非常有用。一些这样的数据示例包括：

- 新闻文章（政治、商业等）

- 博客文章、文档或其他已发布材料（公开或私密）。

- 社交媒体帖子

- 任何类型的变更日志

- 消息

这些项目通常会被同时按相似性和时间搜索。例如：展示我所有关于 2022 年丰田卡车的新闻。

[时序向量](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 通过利用自动表分区来隔离特定时间范围的数据，在搜索特定时间范围内的嵌入时提供了卓越的性能。

Langchain 的自查询检索器允许从用户查询的文本中推断时间范围（以及其他搜索条件）。

## 什么是时序向量？

**[时序向量](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 是用于 AI 应用的 PostgreSQL++。**

时序向量使您能够在 `PostgreSQL` 中高效存储和查询数十亿个向量嵌入。

- 通过 DiskANN 灵感的索引算法，提高了 `pgvector` 在 10 亿个以上向量上的相似性搜索速度和准确性。

- 通过自动基于时间的分区和索引实现快速的基于时间的向量搜索。

- 为查询向量嵌入和关系数据提供了熟悉的 SQL 接口。

时序向量是用于 AI 的云 PostgreSQL，从概念验证到生产环境都能满足您的需求：

- 通过使您能够在单个数据库中存储关系元数据、向量嵌入和时间序列数据，简化了操作。

- 基于坚实的 PostgreSQL 基础构建，具有企业级功能，如流式备份和复制、高可用性和行级安全性。

- 通过企业级安全性和合规性，提供无忧体验。

### 如何访问时序向量

时序向量可在 [Timescale](https://www.timescale.com/products?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)，即云 PostgreSQL 平台上使用。（目前没有自托管版本。）

- LangChain 用户可以免费试用 Timescale Vector 90 天。

- 要开始使用，请[注册](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) Timescale，创建一个新数据库，然后按照此笔记本的步骤进行操作！

- 查看[安装说明](https://github.com/timescale/python-vector)以获取有关在 Python 中使用 Timescale Vector 的更多详细信息。

## 环境设置

此模板使用 Timescale Vector 作为向量存储，并需要 `TIMESCALES_SERVICE_URL`。如果您还没有帐户，请在[此处](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral)注册以获取 90 天的试用期。

要加载示例数据集，请设置 `LOAD_SAMPLE_DATA=1`。要加载您自己的数据集，请参阅下面的部分。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 用法

要使用此软件包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行以下操作：

```shell
langchain app new my-app --package rag-timescale-hybrid-search-time
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-timescale-hybrid-search-time
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_timescale_hybrid_search.chain import chain as rag_timescale_hybrid_search_chain
add_routes(app, rag_timescale_hybrid_search_chain, path="/rag-timescale-hybrid-search")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/rag-timescale-hybrid-search/playground](http://127.0.0.1:8000/rag-timescale-hybrid-search/playground)访问 playground。

我们可以通过以下代码从代码中访问模板：

```python

from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-timescale-hybrid-search")

## 加载自己的数据集

要加载自己的数据集，您需要修改`chain.py`中的`DATASET SPECIFIC CODE`部分的代码。

这段代码定义了集合的名称、如何加载数据，以及集合内容和所有元数据的人类语言描述。这些人类语言描述被自查询检索器用来帮助LLM将问题转换为对元数据的过滤器，从而在Timescale-vector中搜索数据。