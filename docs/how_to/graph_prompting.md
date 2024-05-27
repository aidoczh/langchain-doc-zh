---

sidebar_position: 2

---

# 如何最好地提示图形-RAG

在本指南中，我们将讨论促进图数据库查询生成的提示策略。我们将主要关注获取与数据库特定信息相关的提示方法。

## 设置

首先，获取所需的软件包并设置环境变量：

```python
%pip install --upgrade --quiet langchain langchain-community langchain-openai neo4j
```

```output
注意：您可能需要重新启动内核以使用更新后的软件包。
```

在本指南中，我们默认使用 OpenAI 模型，但您可以将其替换为您选择的模型提供商。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# 取消下面的注释以使用 LangSmith。不是必需的。
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

# 过滤图模式

有时，在生成 Cypher 语句时，您可能需要专注于图模式的特定子集。

假设我们正在处理以下图模式：

```python
graph.refresh_schema()
print(graph.schema)
```

```output
节点属性如下：
Movie {imdbRating: FLOAT, id: STRING, released: DATE, title: STRING},Person {name: STRING},Genre {name: STRING}
关系属性如下：
以下是关系：
(:Movie)-[:IN_GENRE]->(:Genre),(:Person)-[:DIRECTED]->(:Movie),(:Person)-[:ACTED_IN]->(:Movie)
```

假设我们希望在传递给 LLM 的模式表示中排除 _Genre_ 节点。

我们可以使用 GraphCypherQAChain 链的 `exclude` 参数来实现。

```python
from langchain.chains import GraphCypherQAChain
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
chain = GraphCypherQAChain.from_llm(
    graph=graph, llm=llm, exclude_types=["Genre"], verbose=True
)
```

```python
print(chain.graph_schema)
```

```output
节点属性如下：
Movie {imdbRating: FLOAT, id: STRING, released: DATE, title: STRING},Person {name: STRING}
关系属性如下：
以下是关系：
(:Person)-[:DIRECTED]->(:Movie),(:Person)-[:ACTED_IN]->(:Movie)
```

## 少量示例

在提示中包含自然语言问题转换为针对我们数据库的有效 Cypher 查询的示例，通常会提高模型性能，特别是对于复杂查询。

假设我们有以下示例：

```python
examples = [
    {
        "question": "有多少艺术家？",
        "query": "MATCH (a:Person)-[:ACTED_IN]->(:Movie) RETURN count(DISTINCT a)",
    },
    {
        "question": "哪些演员出演了电影《赌城》？",
        "query": "MATCH (m:Movie {{title: 'Casino'}})<-[:ACTED_IN]-(a) RETURN a.name",
    },
    {
        "question": "汤姆·汉克斯出演了多少部电影？",
        "query": "MATCH (a:Person {{name: 'Tom Hanks'}})-[:ACTED_IN]->(m:Movie) RETURN count(m)",
    },
    {
        "question": "列出电影《辛德勒名单》的所有流派。",
        "query": "MATCH (m:Movie {{title: 'Schindler\\'s List'}})-[:IN_GENRE]->(g:Genre) RETURN g.name",
    },
    {
        "question": "哪些演员曾在喜剧和动作两种流派的电影中工作过？",
        "query": "MATCH (a:Person)-[:ACTED_IN]->(:Movie)-[:IN_GENRE]->(g1:Genre), (a)-[:ACTED_IN]->(:Movie)-[:IN_GENRE]->(g2:Genre) WHERE g1.name = 'Comedy' AND g2.name = 'Action' RETURN DISTINCT a.name",
    },
    {
        "question": "哪些导演曾与至少三位名为 'John' 的演员合作拍电影？",
        "query": "MATCH (d:Person)-[:DIRECTED]->(m:Movie)<-[:ACTED_IN]-(a:Person) WHERE a.name STARTS WITH 'John' WITH d, COUNT(DISTINCT a) AS JohnsCount WHERE JohnsCount >= 3 RETURN d.name",
    },
    {
        "question": "识别那些导演也在电影中扮演角色。",
        "query": "MATCH (p:Person)-[:DIRECTED]->(m:Movie), (p)-[:ACTED_IN]->(m) RETURN m.title, p.name",
    },
    {
        "question": "找出数据库中电影数量最多的演员。",
        "query": "MATCH (a:Actor)-[:ACTED_IN]->(m:Movie) RETURN a.name, COUNT(m) AS movieCount ORDER BY movieCount DESC LIMIT 1",
    },
]
```

```python
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate
example_prompt = PromptTemplate.from_template(
    "用户输入: {question}\nCypher 查询: {query}"
)
prompt = FewShotPromptTemplate(
    examples=examples[:5],
    example_prompt=example_prompt,
    prefix="您是 Neo4j 专家。给定一个输入问题，创建一个语法正确的 Cypher 查询来运行。\n\n这是模式信息\n{schema}。\n\n以下是一些问题及其相应的 Cypher 查询示例。",
    suffix="用户输入: {question}\nCypher 查询: ",
    input_variables=["question", "schema"],
)
```

```python
print(prompt.format(question="有多少艺术家？", schema="foo"))
```

```output
您是 Neo4j 专家。给定一个输入问题，创建一个语法正确的 Cypher 查询来运行。
这是模式信息
foo。
以下是一些问题及其相应的 Cypher 查询示例。
用户输入: 有多少艺术家？
Cypher 查询: MATCH (a:Person)-[:ACTED_IN]->(:Movie) RETURN count(DISTINCT a)
用户输入: 电影《赌场》中有哪些演员？
Cypher 查询: MATCH (m:Movie {title: 'Casino'})<-[:ACTED_IN]-(a) RETURN a.name
用户输入: 汤姆·汉克斯出演了多少部电影？
Cypher 查询: MATCH (a:Person {name: 'Tom Hanks'})-[:ACTED_IN]->(m:Movie) RETURN count(m)
用户输入: 电影《辛德勒的名单》的所有流派有哪些？
Cypher 查询: MATCH (m:Movie {title: 'Schindler\'s List'})-[:IN_GENRE]->(g:Genre) RETURN g.name
用户输入: 哪些演员曾在喜剧和动作类型的电影中工作过？
Cypher 查询: MATCH (a:Person)-[:ACTED_IN]->(:Movie)-[:IN_GENRE]->(g1:Genre), (a)-[:ACTED_IN]->(:Movie)-[:IN_GENRE]->(g2:Genre) WHERE g1.name = 'Comedy' AND g2.name = 'Action' RETURN DISTINCT a.name
用户输入: 有多少艺术家？
Cypher 查询: 
```

## 动态 few-shot 示例

如果我们有足够的示例，可能希望仅在提示中包含最相关的示例，要么是因为它们不适合模型的上下文窗口，要么是因为示例的长尾会分散模型的注意力。具体来说，对于任何输入，我们希望包含与该输入最相关的示例。

我们可以使用 ExampleSelector 来实现这一点。在这种情况下，我们将使用 [SemanticSimilarityExampleSelector](https://api.python.langchain.com/en/latest/example_selectors/langchain_core.example_selectors.semantic_similarity.SemanticSimilarityExampleSelector.html)，它将把示例存储在我们选择的向量数据库中。在运行时，它将在输入和我们的示例之间执行语义相似性搜索，并返回最语义上相似的示例：

```python
from langchain_community.vectorstores import Neo4jVector
from langchain_core.example_selectors import SemanticSimilarityExampleSelector
from langchain_openai import OpenAIEmbeddings
example_selector = SemanticSimilarityExampleSelector.from_examples(
    examples,
    OpenAIEmbeddings(),
    Neo4jVector,
    k=5,
    input_keys=["question"],
)
```

```python
example_selector.select_examples({"question": "有多少艺术家？"})
```

```output
[{'query': 'MATCH (a:Person)-[:ACTED_IN]->(:Movie) RETURN count(DISTINCT a)',
  'question': '有多少艺术家？'},
 {'query': "MATCH (a:Person {{name: 'Tom Hanks'}})-[:ACTED_IN]->(m:Movie) RETURN count(m)",
  'question': '汤姆·汉克斯出演了多少部电影？'},
 {'query': "MATCH (a:Person)-[:ACTED_IN]->(:Movie)-[:IN_GENRE]->(g1:Genre), (a)-[:ACTED_IN]->(:Movie)-[:IN_GENRE]->(g2:Genre) WHERE g1.name = 'Comedy' AND g2.name = 'Action' RETURN DISTINCT a.name",
  'question': '哪些演员曾在喜剧和动作类型的电影中工作过？'},
 {'query': "MATCH (d:Person)-[:DIRECTED]->(m:Movie)<-[:ACTED_IN]-(a:Person) WHERE a.name STARTS WITH 'John' WITH d, COUNT(DISTINCT a) AS JohnsCount WHERE JohnsCount >= 3 RETURN d.name",
  'question': "哪些导演制作了至少三位名为'约翰'的不同演员的电影？"},
 {'query': 'MATCH (a:Actor)-[:ACTED_IN]->(m:Movie) RETURN a.name, COUNT(m) AS movieCount ORDER BY movieCount DESC LIMIT 1',
  'question': '找出数据库中电影数量最多的演员。'}]
```

要使用它，我们可以直接将 ExampleSelector 传递给我们的 FewShotPromptTemplate：

```python
prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="您是 Neo4j 专家。给定一个输入问题，创建一个语法正确的 Cypher 查询来运行。\n\n这是模式信息\n{schema}。\n\n以下是一些问题及其相应的 Cypher 查询示例。",
    suffix="用户输入: {question}\nCypher 查询: ",
    input_variables=["question", "schema"],
)
```

```python
print(prompt.format(question="有多少艺术家？", schema="foo"))
```

你是一位 Neo4j 专家。根据输入的问题，创建一个语法正确的 Cypher 查询来运行。

以下是模式信息：

foo.

下面是一些问题及其相应的 Cypher 查询的示例。

用户输入：有多少位艺术家？

Cypher 查询：MATCH (a:Person)-[:ACTED_IN]->(:Movie) RETURN count(DISTINCT a)

用户输入：汤姆·汉克斯演过多少部电影？

Cypher 查询：MATCH (a:Person {name: 'Tom Hanks'})-[:ACTED_IN]->(m:Movie) RETURN count(m)

用户输入：哪些演员曾在喜剧和动作类型的电影中工作过？

Cypher 查询：MATCH (a:Person)-[:ACTED_IN]->(:Movie)-[:IN_GENRE]->(g1:Genre), (a)-[:ACTED_IN]->(:Movie)-[:IN_GENRE]->(g2:Genre) WHERE g1.name = 'Comedy' AND g2.name = 'Action' RETURN DISTINCT a.name

用户输入：哪些导演曾与至少三位名为“约翰”的演员合作过？

Cypher 查询：MATCH (d:Person)-[:DIRECTED]->(m:Movie)<-[:ACTED_IN]-(a:Person) WHERE a.name STARTS WITH 'John' WITH d, COUNT(DISTINCT a) AS JohnsCount WHERE JohnsCount >= 3 RETURN d.name

用户输入：在数据库中，哪位演员拍过最多的电影？

Cypher 查询：MATCH (a:Actor)-[:ACTED_IN]->(m:Movie) RETURN a.name, COUNT(m) AS movieCount ORDER BY movieCount DESC LIMIT 1

用户输入：有多少位艺术家？

Cypher 查询：

```python
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
chain = GraphCypherQAChain.from_llm(
    graph=graph, llm=llm, cypher_prompt=prompt, verbose=True
)
```

```python
chain.invoke("How many actors are in the graph?")
```

```output
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher 查询:
MATCH (a:Person)-[:ACTED_IN]->(:Movie) RETURN count(DISTINCT a)
完整上下文:
[{'count(DISTINCT a)': 967}]
> 链结束.
```

```output
{'query': 'How many actors are in the graph?', 'result': 'There are 967 actors in the graph.'}
```