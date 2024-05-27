# 用 Cypher 查询 Amazon Neptune

[Amazon Neptune](https://aws.amazon.com/neptune/) 是一个高性能的图分析和无服务器数据库，具有卓越的可伸缩性和可用性。

这个例子展示了使用 `openCypher` 查询 `Neptune` 图数据库并返回人类可读的响应的 QA 链。

[Cypher](https://en.wikipedia.org/wiki/Cypher_(query_language)) 是一种声明式图查询语言，允许在属性图中进行表达丰富且高效的数据查询。

[openCypher](https://opencypher.org/) 是 Cypher 的开源实现。

# Neptune Open Cypher QA Chain

这个 QA 链使用 openCypher 查询 Amazon Neptune，并返回人类可读的响应。

LangChain 支持 [Neptune Database](https://docs.aws.amazon.com/neptune/latest/userguide/intro.html) 和 [Neptune Analytics](https://docs.aws.amazon.com/neptune-analytics/latest/userguide/what-is-neptune-analytics.html)，并提供 `NeptuneOpenCypherQAChain`。

Neptune Database 是一个无服务器图数据库，专为实现最佳可伸缩性和可用性而设计。它为需要每秒扩展到 100,000 次查询、多可用区高可用性和多区域部署的图数据库工作负载提供了解决方案。您可以将 Neptune Database 用于社交网络、欺诈警报和客户 360 应用程序。

Neptune Analytics 是一个分析数据库引擎，可以快速分析大量图数据以获取见解和发现趋势。Neptune Analytics 是快速分析现有图数据库或存储在数据湖中的图数据集的解决方案。它使用流行的图分析算法和低延迟的分析查询。

## 使用 Neptune Database

```python
from langchain_community.graphs import NeptuneGraph
host = "<neptune-host>"
port = 8182
use_https = True
graph = NeptuneGraph(host=host, port=port, use_https=use_https)
```

### 使用 Neptune Analytics

```python
from langchain_community.graphs import NeptuneAnalyticsGraph
graph = NeptuneAnalyticsGraph(graph_identifier="<neptune-analytics-graph-id>")
```

## 使用 NeptuneOpenCypherQAChain

这个 QA 链使用 openCypher 查询 Neptune 图数据库，并返回人类可读的响应。

```python
from langchain.chains import NeptuneOpenCypherQAChain
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(temperature=0, model="gpt-4")
chain = NeptuneOpenCypherQAChain.from_llm(llm=llm, graph=graph)
chain.invoke("奥斯汀机场有多少出站航线？")
```

```output
'奥斯汀机场有 98 条出站航线。'
```