

# Neo4j

[Neo4j](https://neo4j.com/docs/getting-started/) 是由 `Neo4j, Inc` 开发的图数据库管理系统。

`Neo4j` 存储的数据元素包括节点、连接节点的边以及节点和边的属性。开发人员将其描述为符合 ACID 的事务性数据库，具有本地图存储和处理功能。`Neo4j` 有一个非开源的“社区版”许可证，根据 GNU 通用公共许可证的修改进行许可，同时提供在线备份和高可用性扩展，这些扩展则根据封闭源商业许可证进行许可。Neo还以封闭源商业条款许可这些带有扩展功能的 `Neo4j`。

这个笔记本展示了如何使用LLMs为图数据库提供自然语言接口，您可以使用 `Cypher` 查询语言进行查询。

[Cypher](https://en.wikipedia.org/wiki/Cypher_(query_language)) 是一种声明式图查询语言，允许在属性图中进行表达丰富且高效的数据查询。

## 设置

您需要一个运行中的 `Neo4j` 实例。一种选择是在他们的Aura云服务中创建一个[免费的Neo4j数据库实例](https://neo4j.com/cloud/platform/aura-graph-database/)。您也可以使用[Neo4j桌面应用程序](https://neo4j.com/download/)在本地运行数据库，或者运行一个docker容器。

您可以通过运行以下脚本来运行本地docker容器：

```bash
docker run \
    --name neo4j \
    -p 7474:7474 -p 7687:7687 \
    -d \
    -e NEO4J_AUTH=neo4j/password \
    -e NEO4J_PLUGINS=["apoc"]  \
    neo4j:latest
```

如果您使用docker容器，需要等待几秒钟让数据库启动。

```python
from langchain.chains import GraphCypherQAChain
from langchain_community.graphs import Neo4jGraph
from langchain_openai import ChatOpenAI
```
```python
graph = Neo4jGraph(url="bolt://localhost:7687", username="neo4j", password="password")
```

## 填充数据库

假设您的数据库是空的，您可以使用Cypher查询语言填充它。以下Cypher语句是幂等的，这意味着如果您运行一次或多次，数据库信息将保持不变。

```python
graph.query(
    """
MERGE (m:Movie {name:"Top Gun", runtime: 120})
WITH m
UNWIND ["Tom Cruise", "Val Kilmer", "Anthony Edwards", "Meg Ryan"] AS actor
MERGE (a:Actor {name:actor})
MERGE (a)-[:ACTED_IN]->(m)
"""
)
```
```output
[]
```

## 刷新图模式信息

如果数据库的模式发生更改，您可以刷新生成Cypher语句所需的模式信息。

```python
graph.refresh_schema()
```
```python
print(graph.schema)
```
```output
节点属性:
电影 {runtime: 整数, name: 字符串}
演员 {name: 字符串}
关系属性:
关系:
(:演员)-[:ACTED_IN]->(:电影)
```

## 增强模式信息

选择增强模式版本可以使系统自动扫描数据库中的示例值并计算一些分布度量。例如，如果一个节点属性具有少于10个不同的值，我们将在模式中返回所有可能的值。否则，每个节点和关系属性仅返回一个示例值。

```python
enhanced_graph = Neo4jGraph(
    url="bolt://localhost:7687",
    username="neo4j",
    password="password",
    enhanced_schema=True,
)
print(enhanced_graph.schema)
```
```output
节点属性:
- **电影**
  - `runtime: 整数` 最小值: 120, 最大值: 120
  - `name: 字符串` 可用选项: ['Top Gun']
- **演员**
  - `name: 字符串` 可用选项: ['Tom Cruise', 'Val Kilmer', 'Anthony Edwards', 'Meg Ryan']
关系属性:
关系:
(:演员)-[:ACTED_IN]->(:电影)
```

## 查询图

现在我们可以使用图Cypher QA链来向图询问问题

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True
)
```
```python
chain.invoke({"query": "谁出演了Top Gun?"})
```
```output
> 进入新的GraphCypherQAChain链...
生成的Cypher:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
WHERE m.name = 'Top Gun'
RETURN a.name
完整上下文:
[{'a.name': 'Anthony Edwards'}, {'a.name': 'Meg Ryan'}, {'a.name': 'Val Kilmer'}, {'a.name': 'Tom Cruise'}]
> 链完成。
```
```output
{'query': '谁出演了Top Gun?',
 'result': 'Anthony Edwards, Meg Ryan, Val Kilmer, Tom Cruise出演了Top Gun。'}
```

## 限制结果数量

您可以使用 `top_k` 参数限制Cypher QA链的结果数量。

默认值为10。

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, top_k=2
)
```
```python
chain.invoke({"query": "谁出演了Top Gun?"})
```
```output
> 进入新的GraphCypherQAChain链...
生成的Cypher:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
WHERE m.name = 'Top Gun'
RETURN a.name
完整上下文:
[{'a.name': 'Anthony Edwards'}, {'a.name': 'Meg Ryan'}]
> 链完成。
## 返回中间结果
您可以使用 `return_intermediate_steps` 参数从 Cypher QA Chain 返回 Cypher 生成的中间步骤。
```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, return_intermediate_steps=True
)
```
```python
result = chain.invoke({"query": "Who played in Top Gun?"})
print(f"中间步骤: {result['intermediate_steps']}")
print(f"最终答案: {result['result']}")
```
```output

> 进入新的 GraphCypherQAChain 链...

生成的 Cypher 语句:

MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)

WHERE m.name = 'Top Gun'

RETURN a.name

完整上下文:

[{'a.name': 'Anthony Edwards'}, {'a.name': 'Meg Ryan'}, {'a.name': 'Val Kilmer'}, {'a.name': 'Tom Cruise'}]

> 链条处理完毕.

中间步骤: [{'query': "MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)\nWHERE m.name = 'Top Gun'\nRETURN a.name"}, {'context': [{'a.name': 'Anthony Edwards'}, {'a.name': 'Meg Ryan'}, {'a.name': 'Val Kilmer'}, {'a.name': 'Tom Cruise'}]}]

最终答案: Anthony Edwards, Meg Ryan, Val Kilmer, Tom Cruise 出演了《壮志凌云》。

```
## 返回直接结果
您可以使用 `return_direct` 参数从 Cypher QA Chain 返回 Cypher 生成的直接结果。
```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, return_direct=True
)
```
```python
chain.invoke({"query": "Who played in Top Gun?"})
```
```output

> 进入新的 GraphCypherQAChain 链...

生成的 Cypher 语句:

MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)

WHERE m.name = 'Top Gun'

RETURN a.name

> 链条处理完毕.

```
```output

{'query': 'Who played in Top Gun?',

 'result': [{'a.name': 'Anthony Edwards'},

  {'a.name': 'Meg Ryan'},

  {'a.name': 'Val Kilmer'},

  {'a.name': 'Tom Cruise'}]}

```
## 在 Cypher 生成提示中添加示例
您可以定义您希望 LLM 为特定问题生成的 Cypher 语句。
```python
from langchain_core.prompts.prompt import PromptTemplate
CYPHER_GENERATION_TEMPLATE = """Task:Generate Cypher statement to query a graph database.
Instructions:
Use only the provided relationship types and properties in the schema.
Do not use any other relationship types or properties that are not provided.
Schema:
{schema}
Note: Do not include any explanations or apologies in your responses.
Do not respond to any questions that might ask anything else than for you to construct a Cypher statement.
Do not include any text except the generated Cypher statement.
Examples: Here are a few examples of generated Cypher statements for particular questions:
# How many people played in Top Gun?
MATCH (m:Movie {{title:"Top Gun"}})<-[:ACTED_IN]-()
RETURN count(*) AS numberOfActors
The question is:
{question}"""
CYPHER_GENERATION_PROMPT = PromptTemplate(
    input_variables=["schema", "question"], template=CYPHER_GENERATION_TEMPLATE
)
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0),
    graph=graph,
    verbose=True,
    cypher_prompt=CYPHER_GENERATION_PROMPT,
)
```
```python
chain.invoke({"query": "How many people played in Top Gun?"})
```
```output

> 进入新的 GraphCypherQAChain 链...

生成的 Cypher 语句:

MATCH (:Movie {name:"Top Gun"})<-[:ACTED_IN]-()

RETURN count(*) AS numberOfActors

完整上下文:

[{'numberOfActors': 4}]

> 链条处理完毕.

```
```output

{'query': 'How many people played in Top Gun?',

 'result': '《壮志凌云》中有 4 位演员参演。'}

```
## 使用不同的 LLM 进行 Cypher 和答案生成
您可以使用 `cypher_llm` 和 `qa_llm` 参数定义不同的 LLM。
```python
chain = GraphCypherQAChain.from_llm(
    graph=graph,
    cypher_llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo"),
    qa_llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo-16k"),
    verbose=True,
)
```
```python
chain.invoke({"query": "Who played in Top Gun?"})
```
```output

> 进入新的 GraphCypherQAChain 链...

生成的 Cypher 语句:

MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)

WHERE m.name = 'Top Gun'

RETURN a.name

完整上下文:

[{'a.name': 'Anthony Edwards'}, {'a.name': 'Meg Ryan'}, {'a.name': 'Val Kilmer'}, {'a.name': 'Tom Cruise'}]

> 链条处理完毕.

```
```output

{'query': 'Who played in Top Gun?',

 'result': '《壮志凌云》中有 Anthony Edwards, Meg Ryan, Val Kilmer 和 Tom Cruise 参演。'}

```
## 忽略指定的节点和关系类型
您可以使用 `include_types` 或 `exclude_types` 在生成 Cypher 语句时忽略图谱架构的部分。
```python
chain = GraphCypherQAChain.from_llm(
    graph=graph,
    cypher_llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo"),
    qa_llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo-16k"),
    verbose=True,
    exclude_types=["Movie"],
)
```
```python
# 检查图模式
print(chain.graph_schema)
```
```output

节点属性如下：

Actor {name: STRING}

关系属性如下：

以下是关系：

```
## 验证生成的 Cypher 语句
您可以使用 `validate_cypher` 参数来验证和纠正生成的 Cypher 语句中的关系方向
```python
chain = GraphCypherQAChain.from_llm(
    llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo"),
    graph=graph,
    verbose=True,
    validate_cypher=True,
)
```
```python
chain.invoke({"query": "Who played in Top Gun?"})
```
```output

> 进入新的 GraphCypherQAChain 链...

生成的 Cypher 语句:

MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)

WHERE m.name = 'Top Gun'

RETURN a.name

完整上下文:

[{'a.name': 'Anthony Edwards'}, {'a.name': 'Meg Ryan'}, {'a.name': 'Val Kilmer'}, {'a.name': 'Tom Cruise'}]

> 链结束。

```
```output

{'query': 'Who played in Top Gun?',

 'result': 'Anthony Edwards, Meg Ryan, Val Kilmer, Tom Cruise 出演了《壮志凌云》。'}

```