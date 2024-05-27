---

sidebar_position: 0

---

# 在图数据库上构建问答应用程序

在本指南中，我们将介绍在图数据库上创建问答链的基本方法。这些系统将允许我们询问有关图数据库中数据的问题，并得到自然语言的答案。

## ⚠️ 安全提示 ⚠️

构建图数据库的问答系统需要执行模型生成的图查询。这样做存在固有风险。确保您的数据库连接权限始终尽可能地针对您的链/代理的需求进行了范围限定。这将减轻但不会消除构建基于模型的系统的风险。有关一般安全最佳实践，[请参阅此处](/docs/security)。

## 架构

在高层次上，大多数图链的步骤如下：

1. **将问题转换为图数据库查询**：模型将用户输入转换为图数据库查询（例如 Cypher）。

2. **执行图数据库查询**：执行图数据库查询。

3. **回答问题**：模型使用查询结果回应用户输入。

![sql_usecase.png](../../static/img/graph_usecase.png)

## 设置

首先，获取所需的软件包并设置环境变量。

在本示例中，我们将使用 Neo4j 图数据库。

```python
%pip install --upgrade --quiet  langchain langchain-community langchain-openai neo4j
```

在本指南中，我们默认使用 OpenAI 模型。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# 取消下面的注释以使用 LangSmith。非必需。
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

```output
········
```

接下来，我们需要定义 Neo4j 凭据。

按照[这些安装步骤](https://neo4j.com/docs/operations-manual/current/installation/)设置 Neo4j 数据库。

```python
os.environ["NEO4J_URI"] = "bolt://localhost:7687"
os.environ["NEO4J_USERNAME"] = "neo4j"
os.environ["NEO4J_PASSWORD"] = "password"
```

下面的示例将创建与 Neo4j 数据库的连接，并将其填充有关电影及其演员的示例数据。

```python
from langchain_community.graphs import Neo4jGraph
graph = Neo4jGraph()
# 导入电影信息
movies_query = """
LOAD CSV WITH HEADERS FROM 
'https://raw.githubusercontent.com/tomasonjo/blog-datasets/main/movies/movies_small.csv'
AS row
MERGE (m:Movie {id:row.movieId})
SET m.released = date(row.released),
    m.title = row.title,
    m.imdbRating = toFloat(row.imdbRating)
FOREACH (director in split(row.director, '|') | 
    MERGE (p:Person {name:trim(director)})
    MERGE (p)-[:DIRECTED]->(m))
FOREACH (actor in split(row.actors, '|') | 
    MERGE (p:Person {name:trim(actor)})
    MERGE (p)-[:ACTED_IN]->(m))
FOREACH (genre in split(row.genres, '|') | 
    MERGE (g:Genre {name:trim(genre)})
    MERGE (m)-[:IN_GENRE]->(g))
"""
graph.query(movies_query)
```

```output
[]
```

## 图模式

为了使 LLM 能够生成 Cypher 语句，它需要有关图模式的信息。当您实例化一个图对象时，它会检索有关图模式的信息。如果您稍后对图进行任何更改，可以运行 `refresh_schema` 方法来刷新模式信息。

```python
graph.refresh_schema()
print(graph.schema)
```

```output
节点属性如下：
Movie {imdbRating: FLOAT, id: STRING, released: DATE, title: STRING},Person {name: STRING},Genre {name: STRING},Chunk {id: STRING, question: STRING, query: STRING, text: STRING, embedding: LIST}
关系属性如下：
关系如下：
(:Movie)-[:IN_GENRE]->(:Genre),(:Person)-[:DIRECTED]->(:Movie),(:Person)-[:ACTED_IN]->(:Movie)
```

太棒了！我们有一个可以查询的图数据库。现在让我们尝试将其连接到一个 LLM。

## 链

让我们使用一个简单的链，它接受一个问题，将其转换为 Cypher 查询，执行查询，并使用结果来回答原始问题。

![graph_chain.webp](../../static/img/graph_chain.webp)

LangChain 提供了一个内置链，用于与 Neo4j 配合使用的工作流：[GraphCypherQAChain](/docs/integrations/graphs/neo4j_cypher)

```python
from langchain.chains import GraphCypherQAChain
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
chain = GraphCypherQAChain.from_llm(graph=graph, llm=llm, verbose=True)
response = chain.invoke({"query": "What was the cast of the Casino?"})
response
```

```output
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher：
MATCH (:Movie {title: "Casino"})<-[:ACTED_IN]-(actor:Person)
RETURN actor.name
完整上下文：
[{'actor.name': 'Joe Pesci'}, {'actor.name': 'Robert De Niro'}, {'actor.name': 'Sharon Stone'}, {'actor.name': 'James Woods'}]
> 链完成。
```

```output
{'query': 'What was the cast of the Casino?',
 'result': 'The cast of Casino included Joe Pesci, Robert De Niro, Sharon Stone, and James Woods.'}
```

# 验证关系方向

LLMs 在生成的 Cypher 语句中可能会出现关系方向的问题。由于图模式是预定义的，我们可以使用 `validate_cypher` 参数验证并在必要时纠正生成的 Cypher 语句中的关系方向。

```python
chain = GraphCypherQAChain.from_llm(
    graph=graph, llm=llm, verbose=True, validate_cypher=True
)
response = chain.invoke({"query": "What was the cast of the Casino?"})
response
```

```output
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher 语句：
MATCH (:Movie {title: "Casino"})<-[:ACTED_IN]-(actor:Person)
RETURN actor.name
完整上下文：
[{'actor.name': 'Joe Pesci'}, {'actor.name': 'Robert De Niro'}, {'actor.name': 'Sharon Stone'}, {'actor.name': 'James Woods'}]
> 链结束。
```

```output
{'query': 'What was the cast of the Casino?',
 'result': 'The cast of Casino included Joe Pesci, Robert De Niro, Sharon Stone, and James Woods.'}
```

### 下一步

对于更复杂的查询生成，我们可能希望创建少量提示或添加查询检查步骤。要了解更多关于高级技术的信息，请查看：

* [提示策略](/docs/how_to/graph_prompting)：高级提示工程技术。

* [映射数值](/docs/how_to/graph_mapping)：从问题到数据库的数值映射技术。

* [语义层](/docs/how_to/graph_semantic)：实现语义层的技术。

* [构建图](/docs/how_to/graph_constructing)：构建知识图谱的技术。