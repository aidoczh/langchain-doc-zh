# FalkorDB

>[FalkorDB](https://www.falkordb.com/) 是一个低延迟的图数据库，为 GenAI 提供知识。

本文展示了如何使用 LLM（语言模型）为 `FalkorDB` 数据库提供自然语言接口。

## 设置

您可以在本地运行 `falkordb` Docker 容器：

```bash
docker run -p 6379:6379 -it --rm falkordb/falkordb
```

启动后，您可以在本地机器上创建一个数据库并连接到它。

```python
from langchain.chains import FalkorDBQAChain
from langchain_community.graphs import FalkorDBGraph
from langchain_openai import ChatOpenAI
```

## 创建图连接并插入演示数据

```python
graph = FalkorDBGraph(database="movies")
```

```python
graph.query(
    """
    CREATE 
        (al:Person {name: 'Al Pacino', birthDate: '1940-04-25'}),
        (robert:Person {name: 'Robert De Niro', birthDate: '1943-08-17'}),
        (tom:Person {name: 'Tom Cruise', birthDate: '1962-07-3'}),
        (val:Person {name: 'Val Kilmer', birthDate: '1959-12-31'}),
        (anthony:Person {name: 'Anthony Edwards', birthDate: '1962-7-19'}),
        (meg:Person {name: 'Meg Ryan', birthDate: '1961-11-19'}),
        (god1:Movie {title: 'The Godfather'}),
        (god2:Movie {title: 'The Godfather: Part II'}),
        (god3:Movie {title: 'The Godfather Coda: The Death of Michael Corleone'}),
        (top:Movie {title: 'Top Gun'}),
        (al)-[:ACTED_IN]->(god1),
        (al)-[:ACTED_IN]->(god2),
        (al)-[:ACTED_IN]->(god3),
        (robert)-[:ACTED_IN]->(god2),
        (tom)-[:ACTED_IN]->(top),
        (val)-[:ACTED_IN]->(top),
        (anthony)-[:ACTED_IN]->(top),
        (meg)-[:ACTED_IN]->(top)
"""
)
```

```output
[]
```

## 创建 FalkorDBQAChain

```python
graph.refresh_schema()
print(graph.schema)
import os
os.environ["OPENAI_API_KEY"] = "API_KEY_HERE"
```

```output
Node properties: [[OrderedDict([('label', None), ('properties', ['name', 'birthDate', 'title'])])]]
Relationships properties: [[OrderedDict([('type', None), ('properties', [])])]]
Relationships: [['(:Person)-[:ACTED_IN]->(:Movie)']]
```

```python
chain = FalkorDBQAChain.from_llm(ChatOpenAI(temperature=0), graph=graph, verbose=True)
```

## 查询图数据库

```python
chain.run("Who played in Top Gun?")
```

```output
> 进入新的 FalkorDBQAChain 链...
生成的 Cypher 查询语句:
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.title = 'Top Gun'
RETURN p.name
完整上下文:
[['Tom Cruise'], ['Val Kilmer'], ['Anthony Edwards'], ['Meg Ryan'], ['Tom Cruise'], ['Val Kilmer'], ['Anthony Edwards'], ['Meg Ryan']]
> 链结束。
```

```output
'在《Top Gun》中，Tom Cruise、Val Kilmer、Anthony Edwards 和 Meg Ryan 都有演出。'
```

```python
chain.run("Who is the oldest actor who played in The Godfather: Part II?")
```

```output
> 进入新的 FalkorDBQAChain 链...
生成的 Cypher 查询语句:
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
WHERE m.title = 'The Godfather: Part II'
RETURN p.name
ORDER BY p.birthDate ASC
LIMIT 1
完整上下文:
[['Al Pacino']]
> 链结束。
```

```output
'在《The Godfather: Part II》中，最年长的演员是 Al Pacino。'
```

```python
chain.run("Robert De Niro played in which movies?")
```

```output
> 进入新的 FalkorDBQAChain 链...
生成的 Cypher 查询语句:
MATCH (p:Person {name: 'Robert De Niro'})-[:ACTED_IN]->(m:Movie)
RETURN m.title
完整上下文:
[['The Godfather: Part II'], ['The Godfather: Part II']]
> 链结束。
```

```output
'Robert De Niro 出演了《The Godfather: Part II》。'
```