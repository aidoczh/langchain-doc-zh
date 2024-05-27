# 如何将值映射到图数据库

在本指南中，我们将介绍通过将用户输入的值映射到数据库来改进图数据库查询生成的策略。

当使用内置的图链时，LLM知道图模式，但不知道存储在数据库中的属性值。

因此，我们可以在图数据库QA系统中引入一个新的步骤来准确地映射值。

## 设置

首先，获取所需的软件包并设置环境变量：

```python
%pip install --upgrade --quiet  langchain langchain-community langchain-openai neo4j
```

在本指南中，默认使用OpenAI模型，但您可以将其替换为您选择的模型提供商。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# Uncomment the below to use LangSmith. Not required.
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

接下来，我们需要定义Neo4j凭据。

按照[这些安装步骤](https://neo4j.com/docs/operations-manual/current/installation/)设置Neo4j数据库。

```python
os.environ["NEO4J_URI"] = "bolt://localhost:7687"
os.environ["NEO4J_USERNAME"] = "neo4j"
os.environ["NEO4J_PASSWORD"] = "password"
```

下面的示例将创建与Neo4j数据库的连接，并将其填充有关电影及其演员的示例数据。

```python
from langchain_community.graphs import Neo4jGraph
graph = Neo4jGraph()
# Import movie information
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

## 检测用户输入中的实体

我们需要提取我们想要映射到图数据库的实体/值的类型。在本示例中，我们处理的是一个电影图，因此我们可以将电影和人物映射到数据库。

```python
from typing import List, Optional
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
class Entities(BaseModel):
    """Identifying information about entities."""
    names: List[str] = Field(
        ...,
        description="All the person or movies appearing in the text",
    )
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are extracting person and movies from the text.",
        ),
        (
            "human",
            "Use the given format to extract information from the following "
            "input: {question}",
        ),
    ]
)
entity_chain = prompt | llm.with_structured_output(Entities)
```

我们可以测试实体提取链。

```python
entities = entity_chain.invoke({"question": "Who played in Casino movie?"})
entities
```

我们将利用一个简单的`CONTAINS`子句将实体与数据库进行匹配。在实际应用中，您可能希望使用模糊搜索或全文索引来允许轻微的拼写错误。

```python
match_query = """MATCH (p:Person|Movie)
WHERE p.name CONTAINS $value OR p.title CONTAINS $value
RETURN coalesce(p.name, p.title) AS result, labels(p)[0] AS type
LIMIT 1
"""
def map_to_database(entities: Entities) -> Optional[str]:
    result = ""
    for entity in entities.names:
        response = graph.query(match_query, {"value": entity})
        try:
            result += f"{entity} maps to {response[0]['result']} {response[0]['type']} in database\n"
        except IndexError:
            pass
    return result
map_to_database(entities)
```

## 自定义Cypher生成链

我们需要定义一个自定义的Cypher提示，该提示将实体映射信息与模式和用户问题一起传递给构建Cypher语句。

我们将使用LangChain表达式语言来实现。

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
# Generate Cypher statement based on natural language input
cypher_template = """Based on the Neo4j graph schema below, write a Cypher query that would answer the user's question:
{schema}
Entities in the question map to the following database values:
{entities_list}
Question: {question}
Cypher query:"""  # noqa: E501
cypher_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "Given an input question, convert it to a Cypher query. No pre-amble.",
        ),
        ("human", cypher_template),
    ]
)
cypher_response = (
    RunnablePassthrough.assign(names=entity_chain)
    | RunnablePassthrough.assign(
        entities_list=lambda x: map_to_database(x["names"]),
        schema=lambda _: graph.get_schema,
    )
    | cypher_prompt
    | llm.bind(stop=["\nCypherResult:"])
    | StrOutputParser()
)
```

```python
cypher = cypher_response.invoke({"question": "谁出演了《赌城风云》这部电影?"})
cypher
```

```output
'MATCH (:Movie {title: "Casino"})<-[:ACTED_IN]-(actor)\nRETURN actor.name'
```

## 根据数据库结果生成答案

现在我们有了一个生成 Cypher 语句的链条，我们需要执行这个 Cypher 语句并将数据库结果发送回一个 LLM，以生成最终答案。

同样，我们将使用 LCEL。

```python
from langchain.chains.graph_qa.cypher_utils import CypherQueryCorrector, Schema
# 用于关系方向的 Cypher 验证工具
corrector_schema = [
    Schema(el["start"], el["type"], el["end"])
    for el in graph.structured_schema.get("relationships")
]
cypher_validation = CypherQueryCorrector(corrector_schema)
# 根据数据库结果生成自然语言响应
response_template = """根据问题、Cypher 查询和 Cypher 响应，写出一个自然语言响应：
问题: {question}
Cypher 查询: {query}
Cypher 响应: {response}"""  # noqa: E501
response_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "根据输入的问题和 Cypher 响应，将其转换为自然语言答案。不需要废话。",
        ),
        ("human", response_template),
    ]
)
chain = (
    RunnablePassthrough.assign(query=cypher_response)
    | RunnablePassthrough.assign(
        response=lambda x: graph.query(cypher_validation(x["query"])),
    )
    | response_prompt
    | llm
    | StrOutputParser()
)
```

```python
chain.invoke({"question": "谁出演了《赌城风云》这部电影?"})
```

```output
'罗伯特·德尼罗、詹姆斯·伍兹、乔·佩西和莎朗·斯通出演了电影《赌城风云》。'
```

