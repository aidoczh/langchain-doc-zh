# Azure Cosmos DB 用于 Apache Gremlin

[**Azure Cosmos DB 用于 Apache Gremlin**](https://learn.microsoft.com/en-us/azure/cosmos-db/gremlin/introduction) 是一个图数据库服务，可用于存储具有数十亿个顶点和边的大规模图形。您可以使用毫秒级延迟查询图形，并轻松演变图形结构。

[Gremlin](https://en.wikipedia.org/wiki/Gremlin_(query_language)) 是由 `Apache TinkerPop` 的 `Apache Software Foundation` 开发的图遍历语言和虚拟机。

这个笔记本展示了如何使用LLMs为图数据库提供自然语言接口，您可以使用 `Gremlin` 查询语言进行查询。

## 设置

安装库：

```python
!pip3 install gremlinpython
```

您将需要一个 Azure CosmosDB 图数据库实例。一个选择是在 Azure 中创建一个[免费的 CosmosDB 图数据库实例](https://learn.microsoft.com/en-us/azure/cosmos-db/free-tier)。

当您创建 Cosmos DB 帐户和图形时，请使用 `/type` 作为分区键。

```python
cosmosdb_name = "mycosmosdb"
cosmosdb_db_id = "graphtesting"
cosmosdb_db_graph_id = "mygraph"
cosmosdb_access_Key = "longstring=="
```

```python
import nest_asyncio
from langchain.chains.graph_qa.gremlin import GremlinQAChain
from langchain_community.graphs import GremlinGraph
from langchain_community.graphs.graph_document import GraphDocument, Node, Relationship
from langchain_core.documents import Document
from langchain_openai import AzureChatOpenAI
```

```python
graph = GremlinGraph(
    url=f"=wss://{cosmosdb_name}.gremlin.cosmos.azure.com:443/",
    username=f"/dbs/{cosmosdb_db_id}/colls/{cosmosdb_db_graph_id}",
    password=cosmosdb_access_Key,
)
```

## 数据库初始化

假设您的数据库是空的，您可以使用 GraphDocuments 来填充它。

对于 Gremlin，始终为每个节点添加一个名为 'label' 的属性。

如果未设置标签，则将 Node.type 用作标签。

对于使用自然ID的 Cosmos，是有意义的，因为它们在图形浏览器中可见。

```python
source_doc = Document(
    page_content="Matrix is a movie where Keanu Reeves, Laurence Fishburne and Carrie-Anne Moss acted."
)
movie = Node(id="The Matrix", properties={"label": "movie", "title": "The Matrix"})
actor1 = Node(id="Keanu Reeves", properties={"label": "actor", "name": "Keanu Reeves"})
actor2 = Node(
    id="Laurence Fishburne", properties={"label": "actor", "name": "Laurence Fishburne"}
)
actor3 = Node(
    id="Carrie-Anne Moss", properties={"label": "actor", "name": "Carrie-Anne Moss"}
)
rel1 = Relationship(
    id=5, type="ActedIn", source=actor1, target=movie, properties={"label": "ActedIn"}
)
rel2 = Relationship(
    id=6, type="ActedIn", source=actor2, target=movie, properties={"label": "ActedIn"}
)
rel3 = Relationship(
    id=7, type="ActedIn", source=actor3, target=movie, properties={"label": "ActedIn"}
)
rel4 = Relationship(
    id=8,
    type="Starring",
    source=movie,
    target=actor1,
    properties={"label": "Strarring"},
)
rel5 = Relationship(
    id=9,
    type="Starring",
    source=movie,
    target=actor2,
    properties={"label": "Strarring"},
)
rel6 = Relationship(
    id=10,
    type="Straring",
    source=movie,
    target=actor3,
    properties={"label": "Strarring"},
)
graph_doc = GraphDocument(
    nodes=[movie, actor1, actor2, actor3],
    relationships=[rel1, rel2, rel3, rel4, rel5, rel6],
    source=source_doc,
)
```

```python
# 当在笔记本中运行时，底层的 python-gremlin 存在问题
# 以下行是解决问题的一种方法
nest_asyncio.apply()
# 将文档添加到 CosmosDB 图形中。
graph.add_graph_documents([graph_doc])
```

## 刷新图形模式信息

如果数据库的模式发生更改（更新后），您可以刷新模式信息。

```python
graph.refresh_schema()
```

```python
print(graph.schema)
```

## 查询图形

现在我们可以使用 Gremlin QA 链来询问图形问题。

```python
chain = GremlinQAChain.from_llm(
    AzureChatOpenAI(
        temperature=0,
        azure_deployment="gpt-4-turbo",
    ),
    graph=graph,
    verbose=True,
)
```

```python
chain.invoke("谁出演了《黑客帝国》？")
```

```python
chain.run("《黑客帝国》中有多少人参演？")
```