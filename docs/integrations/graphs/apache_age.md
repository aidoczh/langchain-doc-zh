# Apache AGE

[Apache AGE](https://age.apache.org/) 是一个 PostgreSQL 扩展，提供图数据库功能。AGE 是 A Graph Extension 的缩写，受到 Bitnine 公司的 PostgreSQL 10 分支 AgensGraph 的启发，后者是一个多模型数据库。该项目的目标是创建一个可以处理关系型和图模型数据的单一存储，使用户可以同时使用标准的 ANSI SQL 和图查询语言 openCypher。`Apache AGE` 存储的数据元素包括节点、连接它们的边以及节点和边的属性。

这个笔记本展示了如何使用 LLM（Large Language Models）为图数据库提供自然语言接口，您可以使用 `Cypher` 查询语言进行查询。

[Cypher](https://en.wikipedia.org/wiki/Cypher_(query_language)) 是一种声明式图查询语言，允许在属性图中进行表达丰富且高效的数据查询。

## 设置

您需要运行一个带有 AGE 扩展的 `Postgre` 实例。测试的一个选项是使用官方 AGE docker 镜像运行一个 docker 容器。

您可以通过运行以下脚本来运行本地 docker 容器：

```bash
docker run \
    --name age  \
    -p 5432:5432 \
    -e POSTGRES_USER=postgresUser \
    -e POSTGRES_PASSWORD=postgresPW \
    -e POSTGRES_DB=postgresDB \
    -d \
    apache/age
```

有关在 docker 中运行的附加说明可以在[这里](https://hub.docker.com/r/apache/age)找到。

```python
from langchain.chains import GraphCypherQAChain
from langchain_community.graphs.age_graph import AGEGraph
from langchain_openai import ChatOpenAI
```

```python
conf = {
    "database": "postgresDB",
    "user": "postgresUser",
    "password": "postgresPW",
    "host": "localhost",
    "port": 5432,
}
graph = AGEGraph(graph_name="age_test", conf=conf)
```

## 填充数据库

假设您的数据库是空的，您可以使用 Cypher 查询语言来填充它。以下的 Cypher 语句是幂等的，这意味着如果您运行一次或多次，数据库信息将是相同的。

```python
graph.query(
    """
MERGE (m:Movie {name:"Top Gun"})
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

如果数据库的模式发生变化，您可以刷新生成 Cypher 语句所需的模式信息。

```python
graph.refresh_schema()
```

```python
print(graph.schema)
```

```output
        Node properties are the following:
        [{'properties': [{'property': 'name', 'type': 'STRING'}], 'labels': 'Actor'}, {'properties': [{'property': 'property_a', 'type': 'STRING'}], 'labels': 'LabelA'}, {'properties': [], 'labels': 'LabelB'}, {'properties': [], 'labels': 'LabelC'}, {'properties': [{'property': 'name', 'type': 'STRING'}], 'labels': 'Movie'}]
        Relationship properties are the following:
        [{'properties': [], 'type': 'ACTED_IN'}, {'properties': [{'property': 'rel_prop', 'type': 'STRING'}], 'type': 'REL_TYPE'}]
        The relationships are the following:
        ['(:`Actor`)-[:`ACTED_IN`]->(:`Movie`)', '(:`LabelA`)-[:`REL_TYPE`]->(:`LabelB`)', '(:`LabelA`)-[:`REL_TYPE`]->(:`LabelC`)']
```

## 查询图

现在我们可以使用图 cypher QA 链来询问图的问题。

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True
)
```

```python
chain.invoke("Who played in Top Gun?")
```

```output
> Entering new GraphCypherQAChain chain...
Generated Cypher:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
WHERE m.name = 'Top Gun'
RETURN a.name
Full Context:
[{'name': 'Tom Cruise'}, {'name': 'Val Kilmer'}, {'name': 'Anthony Edwards'}, {'name': 'Meg Ryan'}]
> Finished chain.
```

```output
{'query': 'Who played in Top Gun?',
 'result': 'Tom Cruise, Val Kilmer, Anthony Edwards, Meg Ryan played in Top Gun.'}
```

## 限制结果数量

您可以使用 `top_k` 参数来限制 Cypher QA 链的结果数量。默认值为 10。

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, top_k=2
)
```

```python
chain.invoke("Who played in Top Gun?")
```

```output
> Entering new GraphCypherQAChain chain...
Generated Cypher:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie {name: 'Top Gun'})
RETURN a.name
Full Context:
[{'name': 'Tom Cruise'}, {'name': 'Val Kilmer'}]
> Finished chain.
```

```output
{'query': 'Who played in Top Gun?',
 'result': 'Tom Cruise, Val Kilmer played in Top Gun.'}
```

## 返回中间结果

您可以使用 `return_intermediate_steps` 参数从 Cypher QA 链中返回中间步骤的结果。

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, return_intermediate_steps=True
)
```

```python
result = chain("Top Gun电影中由谁出演？")
print(f"中间步骤: {result['intermediate_steps']}")
print(f"最终答案: {result['result']}")
```

```output
> 进入新的GraphCypherQAChain链...
生成的Cypher语句:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
WHERE m.name = 'Top Gun'
RETURN a.name
完整上下文:
[{'name': '汤姆·克鲁斯'}, {'name': '瓦尔·基尔默'}, {'name': '安东尼·爱德华兹'}, {'name': '梅格·瑞恩'}]
> 链结束。
中间步骤: [{'query': "MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)\nWHERE m.name = 'Top Gun'\nRETURN a.name"}, {'context': [{'name': '汤姆·克鲁斯'}, {'name': '瓦尔·基尔默'}, {'name': '安东尼·爱德华兹'}, {'name': '梅格·瑞恩'}]}]
最终答案: 汤姆·克鲁斯、瓦尔·基尔默、安东尼·爱德华兹、梅格·瑞恩出演了Top Gun电影。
```

## 返回直接结果

您可以使用`return_direct`参数从Cypher QA Chain中返回直接结果

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, return_direct=True
)
```

```python
chain.invoke("Top Gun电影中由谁出演？")
```

```output
> 进入新的GraphCypherQAChain链...
生成的Cypher语句:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie {name: 'Top Gun'})
RETURN a.name
> 链结束。
```

```output
{'query': 'Top Gun电影中由谁出演？',
 'result': [{'name': '汤姆·克鲁斯'},
  {'name': '瓦尔·基尔默'},
  {'name': '安东尼·爱德华兹'},
  {'name': '梅格·瑞恩'}]}
```

## 在Cypher生成提示中添加示例

您可以为特定问题定义要生成的Cypher语句

```python
from langchain_core.prompts.prompt import PromptTemplate
CYPHER_GENERATION_TEMPLATE = """任务:生成用于查询图数据库的Cypher语句。
说明:
仅使用提供的模式中的关系类型和属性。
不要使用未提供的其他关系类型或属性。
模式:
{schema}
注意: 在您的回复中不要包含任何解释或道歉。
不要回答任何问题，除非要求您构建Cypher语句。
除了生成的Cypher语句外，不要包含任何文本。
示例: 这里是一些特定问题生成的Cypher语句示例:
# Top Gun电影中有多少人出演？
MATCH (m:Movie {{title:"Top Gun"}})<-[:ACTED_IN]-()
RETURN count(*) AS numberOfActors
问题是:
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
chain.invoke("Top Gun电影中有多少人出演？")
```

```output
> 进入新的GraphCypherQAChain链...
``````output
生成的Cypher语句:
MATCH (:Movie {name:"Top Gun"})<-[:ACTED_IN]-(:Actor)
RETURN count(*) AS numberOfActors
完整上下文:
[{'numberofactors': 4}]
> 链结束。
```

```output
{'query': 'Top Gun电影中有多少人出演？',
 'result': "我不知道答案。"}
```

## 使用不同的LLM进行Cypher和答案生成

您可以使用`cypher_llm`和`qa_llm`参数定义不同的LLM

```python
chain = GraphCypherQAChain.from_llm(
    graph=graph,
    cypher_llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo"),
    qa_llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo-16k"),
    verbose=True,
)
```

```python
chain.invoke("Top Gun电影中由谁出演？")
```

```output
> 进入新的GraphCypherQAChain链...
``````output
生成的Cypher语句:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
WHERE m.name = 'Top Gun'
RETURN a.name
完整上下文:
[{'name': '汤姆·克鲁斯'}, {'name': '瓦尔·基尔默'}, {'name': '安东尼·爱德华兹'}, {'name': '梅格·瑞恩'}]
> 链结束。
```

```output
{'query': 'Top Gun电影中由谁出演？',
 'result': '汤姆·克鲁斯、瓦尔·基尔默、安东尼·爱德华兹和梅格·瑞恩出演了Top Gun电影。'}
```

## 忽略指定的节点和关系类型

您可以使用`include_types`或`exclude_types`来在生成Cypher语句时忽略图模式的部分内容

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
节点属性如下:
Actor {name: STRING},LabelA {property_a: STRING},LabelB {},LabelC {}
关系属性如下:
ACTED_IN {},REL_TYPE {rel_prop: STRING}
关系如下:
(:LabelA)-[:REL_TYPE]->(:LabelB),(:LabelA)-[:REL_TYPE]->(:LabelC)
```

## 验证生成的Cypher语句

您可以使用`validate_cypher`参数来验证和纠正生成的Cypher语句中的关系方向

```python
chain = GraphCypherQAChain.from_llm(
    llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo"),
    graph=graph,
    verbose=True,
    validate_cypher=True,
)
```

```python
chain.invoke("Top Gun的演员是谁？")
```

```output
> 进入新的GraphCypherQAChain链...
生成的Cypher查询语句:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
WHERE m.name = 'Top Gun'
RETURN a.name
完整上下文:
[{'name': '汤姆·克鲁斯'}, {'name': '瓦尔·基尔默'}, {'name': '安东尼·爱德华兹'}, {'name': '梅格·瑞恩'}]
> 链条结束。
```

```output
{'query': 'Top Gun的演员是谁？',
 'result': '汤姆·克鲁斯, 瓦尔·基尔默, 安东尼·爱德华兹, 梅格·瑞恩出演了Top Gun。'}
```