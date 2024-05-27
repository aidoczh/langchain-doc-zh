

# Memgraph

>[Memgraph](https://github.com/memgraph/memgraph) 是一款开源图数据库，兼容 `Neo4j`。

>该数据库使用 `Cypher` 图查询语言。

>[Cypher](https://en.wikipedia.org/wiki/Cypher_(query_language)) 是一种声明式图查询语言，允许在属性图中进行富有表现力和高效的数据查询。

这篇笔记展示了如何使用LLMs为 [Memgraph](https://github.com/memgraph/memgraph) 数据库提供自然语言接口。

## 设置

要完成本教程，您需要安装 [Docker](https://www.docker.com/get-started/) 和 [Python 3.x](https://www.python.org/)。

确保您有一个正在运行的 Memgraph 实例。要快速第一次运行 Memgraph 平台（Memgraph 数据库 + MAGE 库 + Memgraph Lab），请执行以下操作：

在 Linux/MacOS 上：

```
curl https://install.memgraph.com | sh
```

在 Windows 上：

```
iwr https://windows.memgraph.com | iex
```

这两个命令运行一个脚本，该脚本会将一个 Docker Compose 文件下载到您的系统中，在两个单独的容器中构建和启动 `memgraph-mage` 和 `memgraph-lab` Docker 服务。

阅读更多关于安装过程的信息，请查看[Memgraph 文档](https://memgraph.com/docs/getting-started/install-memgraph)。

现在您可以开始使用 `Memgraph` 了！

首先安装并导入所有必要的软件包。我们将使用名为 [pip](https://pip.pypa.io/en/stable/installation/) 的软件包管理器，以及 `--user` 标志，以确保适当的权限。如果您安装了 Python 3.4 或更高版本，pip 默认已包含在内。您可以使用以下命令安装所有必需的软件包：

```python
pip install langchain langchain-openai neo4j gqlalchemy --user
```

您可以在此笔记本中运行提供的代码块，也可以使用单独的 Python 文件来尝试 Memgraph 和 LangChain。

```python
import os
from gqlalchemy import Memgraph
from langchain.chains import GraphCypherQAChain
from langchain_community.graphs import MemgraphGraph
from langchain_core.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
```

我们使用 Python 库 [GQLAlchemy](https://github.com/memgraph/gqlalchemy) 来建立我们的 Memgraph 数据库和 Python 脚本之间的连接。您也可以使用兼容 Memgraph 的 Neo4j 驱动程序建立到运行中 Memgraph 实例的连接。要使用 GQLAlchemy 执行查询，我们可以如下设置 Memgraph 实例：

```python
memgraph = Memgraph(host="127.0.0.1", port=7687)
```

## 填充数据库

您可以轻松使用 Cypher 查询语言填充您的新数据库。如果您暂时不理解每一行代码，可以从[这里](https://memgraph.com/docs/cypher-manual/)的文档中学习 Cypher。运行以下脚本将在数据库上执行一个种子查询，为我们提供有关视频游戏的数据，包括发布者、可用平台和流派等详细信息。这些数据将作为我们工作的基础。

```python
# 创建并执行种子查询
query = """
    MERGE (g:Game {name: "Baldur's Gate 3"})
    WITH g, ["PlayStation 5", "Mac OS", "Windows", "Xbox Series X/S"] AS platforms,
            ["Adventure", "Role-Playing Game", "Strategy"] AS genres
    FOREACH (platform IN platforms |
        MERGE (p:Platform {name: platform})
        MERGE (g)-[:AVAILABLE_ON]->(p)
    )
    FOREACH (genre IN genres |
        MERGE (gn:Genre {name: genre})
        MERGE (g)-[:HAS_GENRE]->(gn)
    )
    MERGE (p:Publisher {name: "Larian Studios"})
    MERGE (g)-[:PUBLISHED_BY]->(p);
"""
memgraph.execute(query)
```

## 刷新图模式

您可以通过以下脚本实例化 Memgraph-LangChain 图。这个接口将允许我们使用 LangChain 查询我们的数据库，自动创建生成 Cypher 查询所需的图模式。

```python
graph = MemgraphGraph(url="bolt://localhost:7687", username="", password="")
```

如果需要，您可以手动刷新图模式，如下所示。

```python
graph.refresh_schema()
```

为了熟悉数据并验证更新后的图模式，您可以使用以下语句打印它。

```python
print(graph.schema)
```

```
节点属性如下：
节点名称：'Game'，节点属性：[{'property': 'name', 'type': 'str'}]
节点名称：'Platform'，节点属性：[{'property': 'name', 'type': 'str'}]
节点名称：'Genre'，节点属性：[{'property': 'name', 'type': 'str'}]
节点名称：'Publisher'，节点属性：[{'property': 'name', 'type': 'str'}]
关系属性如下：
关系如下：
['(:Game)-[:AVAILABLE_ON]->(:Platform)']
['(:Game)-[:HAS_GENRE]->(:Genre)']
['(:Game)-[:PUBLISHED_BY]->(:Publisher)']
```

## 查询数据库

要与 OpenAI API 交互，您必须使用 Python 的 [os](https://docs.python.org/3/library/os.html) 包将您的 API 密钥配置为环境变量。这样可以确保您的请求得到适当的授权。您可以在[这里](https://help.openai.com/en/articles/4936850-where-do-i-find-my-secret-api-key)找到有关获取 API 密钥的更多信息。

```python
os.environ["OPENAI_API_KEY"] = "your-key-here"
```

您应该使用以下脚本创建图链，该图链将根据您的图数据在问答过程中使用。虽然默认为 GPT-3.5-turbo，但您也可以考虑尝试其他模型，如[GPT-4](https://help.openai.com/en/articles/7102672-how-can-i-access-gpt-4)，以获得明显改进的 Cypher 查询和结果。我们将使用您之前配置的 OpenAI 聊天密钥。我们将将温度设置为零，以确保回答是可预测且一致的。此外，我们将使用我们的 Memgraph-LangChain 图，并将默认为 False 的 verbose 参数设置为 True，以接收有关查询生成的更详细消息。

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, model_name="gpt-3.5-turbo"
)
```

现在您可以开始提问！

```python
response = chain.run("Baldur's Gate 3 在哪些平台上可用？")
print(response)
```

```
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher:
MATCH (g:Game {name: 'Baldur\'s Gate 3'})-[:AVAILABLE_ON]->(p:Platform)
RETURN p.name
完整上下文:
[{'p.name': 'PlayStation 5'}, {'p.name': 'Mac OS'}, {'p.name': 'Windows'}, {'p.name': 'Xbox Series X/S'}]
> 完成链。
Baldur's Gate 3 可在 PlayStation 5、Mac OS、Windows 和 Xbox Series X/S 上使用。
```

```python
response = chain.run("Baldur's Gate 3 在 Windows 上可用吗？")
print(response)
```

```
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher:
MATCH (:Game {name: 'Baldur\'s Gate 3'})-[:AVAILABLE_ON]->(:Platform {name: 'Windows'})
RETURN true
完整上下文:
[{'true': True}]
> 完成链。
是的，Baldur's Gate 3 可在 Windows 上使用。
```

## 链修饰符

要修改链的行为并获得更多上下文或额外信息，您可以修改链的参数。

#### 返回直接查询结果

`return_direct` 修饰符指定是否返回执行的 Cypher 查询的直接结果还是处理过的自然语言响应。

```python
# 返回直接查询图的结果
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, return_direct=True
)
```

```python
response = chain.run("Baldur's Gate 3 由哪个工作室发行？")
print(response)
```

```
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher:
MATCH (:Game {name: 'Baldur\'s Gate 3'})-[:PUBLISHED_BY]->(p:Publisher)
RETURN p.name
> 完成链。
[{'p.name': 'Larian Studios'}]
```

#### 返回查询中间步骤

`return_intermediate_steps` 链修饰符通过包含查询的中间步骤来增强返回的响应，除了初始查询结果。

```python
# 返回查询执行的所有中间步骤
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, return_intermediate_steps=True
)
```

```python
response = chain("Baldur's Gate 3 是一款冒险游戏吗？")
print(f"中间步骤: {response['intermediate_steps']}")
print(f"最终响应: {response['result']}")
```

```
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher:
MATCH (g:Game {name: 'Baldur\'s Gate 3'})-[:HAS_GENRE]->(genre:Genre {name: 'Adventure'})
RETURN g, genre
完整上下文:
[{'g': {'name': "Baldur's Gate 3"}, 'genre': {'name': 'Adventure'}}]
> 完成链。
中间步骤: [{'query': "MATCH (g:Game {name: 'Baldur\\'s Gate 3'})-[:HAS_GENRE]->(genre:Genre {name: 'Adventure'})\nRETURN g, genre"}, {'context': [{'g': {'name': "Baldur's Gate 3"}, 'genre': {'name': 'Adventure'}}]}]
最终响应: 是的，Baldur's Gate 3 是一款冒险游戏。
```

#### 限制查询结果数量

当您希望限制查询结果的最大数量时，可以使用 `top_k` 修饰符。

```python
# 限制查询返回的结果最大数量
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, top_k=2
)
```

```python
response = chain.run("Baldur's Gate 3 与哪些类型的游戏相关联？")
print(response)
```

```
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher:
MATCH (:Game {name: 'Baldur\'s Gate 3'})-[:HAS_GENRE]->(g:Genre)
RETURN g.name
完整上下文:
[{'g.name': 'Adventure'}, {'g.name': 'Role-Playing Game'}]
> 完成链。
Baldur's Gate 3 与冒险和角色扮演游戏相关联。
```

# 高级查询

随着解决方案的复杂性增加，您可能会遇到需要仔细处理的不同用例。确保应用程序的可扩展性对于保持用户流畅体验至关重要，不应出现任何故障。

让我们再次实例化我们的链，并尝试提出一些用户可能会问的问题。

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True, model_name="gpt-3.5-turbo"
)
```

```python
response = chain.run("Baldur's Gate 3 在 PS5 上有吗?")
print(response)
```

```
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher 查询语句:
MATCH (g:Game {name: 'Baldur\'s Gate 3'})-[:AVAILABLE_ON]->(p:Platform {name: 'PS5'})
返回 g.name, p.name
完整上下文:
[]
> 链结束。
抱歉，我没有信息来回答您的问题。
```

生成的 Cypher 查询看起来不错，但我们没有收到任何信息作为回应。这展示了在使用LLM时常见的挑战之一——用户提出查询的方式与数据存储方式之间的不匹配。在这种情况下，用户感知与实际数据存储之间的差异可能导致不匹配。提示细化是一种有效的解决方案，通过这一过程，模型可以更好地理解这些差异，从而生成准确和相关的查询，成功检索所需数据。

### 提示细化

为了解决这个问题，我们可以调整QA链的初始Cypher提示。这涉及向LLM添加关于用户如何引用特定平台的指导，例如在我们的例子中的PS5。我们使用 LangChain [PromptTemplate](/docs/how_to#prompt-templates) 来实现这一点，创建一个修改后的初始提示。然后将这个修改后的提示作为参数提供给我们精细化的 Memgraph-LangChain 实例。

```python
CYPHER_GENERATION_TEMPLATE = """
任务: 生成Cypher语句以查询图数据库。
说明:
仅使用模式中提供的关系类型和属性。
不要使用未提供的任何其他关系类型或属性。
模式:
{schema}
注意: 在您的回答中不要包括任何解释或道歉。
不要回答任何问题，除非要求您构建Cypher语句。
在用户询问有关PS5、Play Station 5或PS 5时，这个平台称为PlayStation 5。
问题是:
{question}
"""
CYPHER_GENERATION_PROMPT = PromptTemplate(
    input_variables=["schema", "question"], template=CYPHER_GENERATION_TEMPLATE
)
```

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0),
    cypher_prompt=CYPHER_GENERATION_PROMPT,
    graph=graph,
    verbose=True,
    model_name="gpt-3.5-turbo",
)
```

```python
response = chain.run("Baldur's Gate 3 在 PS5 上有吗?")
print(response)
```

```
> 进入新的 GraphCypherQAChain 链...
生成的 Cypher:
MATCH (g:Game {name: 'Baldur\'s Gate 3'})-[:AVAILABLE_ON]->(p:Platform {name: 'PlayStation 5'})
返回 g.name, p.name
完整上下文:
[{'g.name': "Baldur's Gate 3", 'p.name': 'PlayStation 5'}]
> 链结束。
是的，Baldur's Gate 3 在 PlayStation 5 上有。
```

现在，通过包含平台命名指导的修改后初始Cypher提示，我们获得了更准确和相关的结果，与用户查询更加接近。

这种方法允许进一步改进您的QA链。您可以轻松地将额外的提示细化数据集成到您的链中，从而提升应用程序的整体用户体验。

```