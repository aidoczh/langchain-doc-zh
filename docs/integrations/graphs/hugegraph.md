# HugeGraph

[HugeGraph](https://hugegraph.apache.org/) 是一个方便、高效、适应性强的图数据库，兼容 `Apache TinkerPop3` 框架和 `Gremlin` 查询语言。

[Gremlin](https://en.wikipedia.org/wiki/Gremlin_(query_language)) 是由 `Apache TinkerPop` 的 `Apache Software Foundation` 开发的图遍历语言和虚拟机。

本笔记展示了如何使用LLMs为[HugeGraph](https://hugegraph.apache.org/cn/)数据库提供自然语言接口。

## 设置

您需要运行一个HugeGraph实例。您可以通过执行以下脚本在本地运行一个docker容器：

```shell
docker run \
    --name=graph \
    -itd \
    -p 8080:8080 \
    hugegraph/hugegraph
```

如果我们想在应用程序中连接HugeGraph，我们需要安装python sdk：

```shell
pip3 install hugegraph-python
```

如果您使用的是docker容器，需要等待一段时间让数据库启动，然后我们需要为数据库创建模式并写入图数据。

```python
from hugegraph.connection import PyHugeGraph
client = PyHugeGraph("localhost", "8080", user="admin", pwd="admin", graph="hugegraph")
```

首先，我们为一个简单的电影数据库创建模式：

```python
"""schema"""
schema = client.schema()
schema.propertyKey("name").asText().ifNotExist().create()
schema.propertyKey("birthDate").asText().ifNotExist().create()
schema.vertexLabel("Person").properties(
    "name", "birthDate"
).usePrimaryKeyId().primaryKeys("name").ifNotExist().create()
schema.vertexLabel("Movie").properties("name").usePrimaryKeyId().primaryKeys(
    "name"
).ifNotExist().create()
schema.edgeLabel("ActedIn").sourceLabel("Person").targetLabel(
    "Movie"
).ifNotExist().create()
```

```output
'create EdgeLabel success, Detail: "b\'{"id":1,"name":"ActedIn","source_label":"Person","target_label":"Movie","frequency":"SINGLE","sort_keys":[],"nullable_keys":[],"index_labels":[],"properties":[],"status":"CREATED","ttl":0,"enable_label_index":true,"user_data":{"~create_time":"2023-07-04 10:48:47.908"}}\'"'
```

然后我们可以插入一些数据。

```python
"""graph"""
g = client.graph()
g.addVertex("Person", {"name": "Al Pacino", "birthDate": "1940-04-25"})
g.addVertex("Person", {"name": "Robert De Niro", "birthDate": "1943-08-17"})
g.addVertex("Movie", {"name": "The Godfather"})
g.addVertex("Movie", {"name": "The Godfather Part II"})
g.addVertex("Movie", {"name": "The Godfather Coda The Death of Michael Corleone"})
g.addEdge("ActedIn", "1:Al Pacino", "2:The Godfather", {})
g.addEdge("ActedIn", "1:Al Pacino", "2:The Godfather Part II", {})
g.addEdge(
    "ActedIn", "1:Al Pacino", "2:The Godfather Coda The Death of Michael Corleone", {}
)
g.addEdge("ActedIn", "1:Robert De Niro", "2:The Godfather Part II", {})
```

```output
1:Robert De Niro--ActedIn-->2:The Godfather Part II
```

## 创建 `HugeGraphQAChain`

现在我们可以创建 `HugeGraph` 和 `HugeGraphQAChain`。要创建 `HugeGraph`，我们只需要将数据库对象传递给 `HugeGraph` 构造函数。

```python
from langchain.chains import HugeGraphQAChain
from langchain_community.graphs import HugeGraph
from langchain_openai import ChatOpenAI
```

```python
graph = HugeGraph(
    username="admin",
    password="admin",
    address="localhost",
    port=8080,
    graph="hugegraph",
)
```

## 刷新图模式信息

如果数据库的模式发生变化，您可以刷新生成Gremlin语句所需的模式信息。

```python
# graph.refresh_schema()
```

```python
print(graph.get_schema)
```

```output
Node properties: [name: Person, primary_keys: ['name'], properties: ['name', 'birthDate'], name: Movie, primary_keys: ['name'], properties: ['name']]
Edge properties: [name: ActedIn, properties: []]
Relationships: ['Person--ActedIn-->Movie']
```

## 查询图

现在我们可以使用图Gremlin QA链来询问图的问题。

```python
chain = HugeGraphQAChain.from_llm(ChatOpenAI(temperature=0), graph=graph, verbose=True)
```

```python
chain.run("Who played in The Godfather?")
```

```output
'Al Pacino played in The Godfather.'
```