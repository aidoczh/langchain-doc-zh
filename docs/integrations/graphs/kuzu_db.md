# Kuzu

[Kùzu](https://kuzudb.com) 是一款可嵌入的属性图数据库管理系统，旨在提高查询速度和可扩展性。

Kùzu采用宽松的（MIT）开源许可证，并实现了[Cypher](https://en.wikipedia.org/wiki/Cypher_(query_language))，这是一种声明式图查询语言，可在属性图中实现富有表现力和高效的数据查询。

它采用列存储，并且其查询处理器包含新颖的连接算法，使其能够扩展到非常大的图，而不会牺牲查询性能。

这个笔记本展示了如何使用LLMs为[Kùzu](https://kuzudb.com)数据库提供自然语言接口，并使用Cypher进行查询。

## 设置

Kùzu是一个嵌入式数据库（在进程中运行），因此无需管理服务器。

只需通过其Python包进行安装：

```bash
pip install kuzu
```

在本地机器上创建一个数据库并连接到它：

```python
import kuzu
db = kuzu.Database("test_db")
conn = kuzu.Connection(db)
```

首先，我们为一个简单的电影数据库创建架构：

```python
conn.execute("CREATE NODE TABLE Movie (name STRING, PRIMARY KEY(name))")
conn.execute("CREATE NODE TABLE Person (name STRING, birthDate STRING, PRIMARY KEY(name))")
conn.execute("CREATE REL TABLE ActedIn (FROM Person TO Movie)")
```

然后我们可以插入一些数据：

```python
conn.execute("CREATE (:Person {name: 'Al Pacino', birthDate: '1940-04-25'})")
conn.execute("CREATE (:Person {name: 'Robert De Niro', birthDate: '1943-08-17'})")
conn.execute("CREATE (:Movie {name: 'The Godfather'})")
conn.execute("CREATE (:Movie {name: 'The Godfather: Part II'})")
conn.execute("CREATE (:Movie {name: 'The Godfather Coda: The Death of Michael Corleone'})")
conn.execute("MATCH (p:Person), (m:Movie) WHERE p.name = 'Al Pacino' AND m.name = 'The Godfather' CREATE (p)-[:ActedIn]->(m)")
conn.execute("MATCH (p:Person), (m:Movie) WHERE p.name = 'Al Pacino' AND m.name = 'The Godfather: Part II' CREATE (p)-[:ActedIn]->(m)")
conn.execute("MATCH (p:Person), (m:Movie) WHERE p.name = 'Al Pacino' AND m.name = 'The Godfather Coda: The Death of Michael Corleone' CREATE (p)-[:ActedIn]->(m)")
conn.execute("MATCH (p:Person), (m:Movie) WHERE p.name = 'Robert De Niro' AND m.name = 'The Godfather: Part II' CREATE (p)-[:ActedIn]->(m)")
```

## 创建 `KuzuQAChain`

现在我们可以创建 `KuzuGraph` 和 `KuzuQAChain`。要创建 `KuzuGraph`，我们只需要将数据库对象传递给 `KuzuGraph` 构造函数：

```python
from langchain.chains import KuzuQAChain
from langchain_community.graphs import KuzuGraph
from langchain_openai import ChatOpenAI
```

```python
graph = KuzuGraph(db)
```

```python
chain = KuzuQAChain.from_llm(
    llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo-16k"),
    graph=graph,
    verbose=True,
)
```

## 刷新图架构信息

如果数据库的架构发生变化，可以刷新生成Cypher语句所需的架构信息。

您还可以显示Kùzu图的架构，如下所示：

```python
# graph.refresh_schema()
```

```python
print(graph.get_schema)
```

```output
Node properties: [{'properties': [('name', 'STRING')], 'label': 'Movie'}, {'properties': [('name', 'STRING'), ('birthDate', 'STRING')], 'label': 'Person'}]
Relationships properties: [{'properties': [], 'label': 'ActedIn'}]
Relationships: ['(:Person)-[:ActedIn]->(:Movie)']
```

## 查询图

现在我们可以使用 `KuzuQAChain` 来向图询问问题：

```python
chain.invoke("Who acted in The Godfather: Part II?")
```

```output
{'query': 'Who acted in The Godfather: Part II?',
 'result': 'Al Pacino, Robert De Niro acted in The Godfather: Part II.'}
```

```python
chain.invoke("Robert De Niro played in which movies?")
```

```output
{'query': 'Robert De Niro played in which movies?',
 'result': 'Robert De Niro played in The Godfather: Part II.'}
```

```python
chain.invoke("How many actors played in the Godfather: Part II?")
```

```output
{'query': 'How many actors played in the Godfather: Part II?',
 'result': '0'}
```

## 使用单独的LLM进行Cypher和答案生成

您可以分别指定`cypher_llm`和`qa_llm`来使用不同的LLM进行Cypher生成和答案生成。

```python
chain = KuzuQAChain.from_llm(
    cypher_llm=ChatOpenAI(temperature=0, model="gpt-3.5-turbo-16k"),
    qa_llm=ChatOpenAI(temperature=0, model="gpt-4"),
    graph=graph,
    verbose=True,
)
```

```output
/Users/prrao/code/langchain/.venv/lib/python3.11/site-packages/langchain_core/_api/deprecation.py:119: LangChainDeprecationWarning: The class `LLMChain` was deprecated in LangChain 0.1.17 and will be removed in 0.3.0. Use RunnableSequence, e.g., `prompt | llm` instead.
  warn_deprecated(
```

```python
chain.invoke("How many actors played in The Godfather: Part II?")
```

```output
> 进入新的KuzuQAChain链...
/Users/prrao/code/langchain/.venv/lib/python3.11/site-packages/langchain_core/_api/deprecation.py:119: LangChainDeprecationWarning: The method `Chain.run` was deprecated in langchain 0.1.0 and will be removed in 0.2.0. Use invoke instead.
  warn_deprecated(
```

```output
生成的Cypher:
MATCH (:Person)-[:ActedIn]->(:Movie {name: 'The Godfather: Part II'})
RETURN count(*)
完整上下文:
[{'COUNT_STAR()': 2}]
```

```output
/Users/prrao/code/langchain/.venv/lib/python3.11/site-packages/langchain_core/_api/deprecation.py:119: LangChainDeprecationWarning: The method `Chain.__call__` was deprecated in langchain 0.1.0 and will be removed in 0.2.0. Use invoke instead.
  warn_deprecated(
```

```output
> 完成链。
```

```output
{'query': 'The Godfather: Part II有多少位演员参演?',
 'result': 'The Godfather: Part II有两位演员参演。'}
```