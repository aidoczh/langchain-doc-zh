# GraphQL

[GraphQL](https://graphql.org/) 是一种用于 API 的查询语言，也是用于执行这些查询的运行时。`GraphQL` 提供了对 API 中数据的完整和清晰的描述，使客户端有权仅请求他们需要的内容，而不多余的内容，更易于随时间演进 API，并且支持强大的开发者工具。

通过在提供给代理的工具列表中包含 `BaseGraphQLTool`，您可以授予代理从 GraphQL API 查询数据的能力，以满足您的任何需求。

这个 Jupyter Notebook 演示了如何使用 `GraphQLAPIWrapper` 组件与代理进行交互。

在这个例子中，我们将使用公共的 `Star Wars GraphQL API`，其终端点为：https://swapi-graphql.netlify.app/.netlify/functions/index。

首先，您需要安装 `httpx` 和 `gql` Python 包。

```python
pip install httpx gql > /dev/null
```

现在，让我们创建一个具有指定 Star Wars API 终端点的 BaseGraphQLTool 实例，并使用该工具初始化一个代理。

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
llm = OpenAI(temperature=0)
tools = load_tools(
    ["graphql"],
    graphql_endpoint="https://swapi-graphql.netlify.app/.netlify/functions/index",
)
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

现在，我们可以使用代理来针对 Star Wars GraphQL API 运行查询。让我们要求代理列出所有 Star Wars 电影及其上映日期。

```python
graphql_fields = """allFilms {
    films {
      title
      director
      releaseDate
      speciesConnection {
        species {
          name
          classification
          homeworld {
            name
          }
        }
      }
    }
  }
"""
suffix = "Search for the titles of all the stawars films stored in the graphql database that has this schema "
agent.run(suffix + graphql_fields)
```

```output
> 进入新的 AgentExecutor 链...
 我需要查询 graphql 数据库以获取所有星球大战电影的标题
操作：query_graphql
操作输入：query { allFilms { films { title } } }
观察结果："{\n  \"allFilms\": {\n    \"films\": [\n      {\n        \"title\": \"A New Hope\"\n      },\n      {\n        \"title\": \"The Empire Strikes Back\"\n      },\n      {\n        \"title\": \"Return of the Jedi\"\n      },\n      {\n        \"title\": \"The Phantom Menace\"\n      },\n      {\n        \"title\": \"Attack of the Clones\"\n      },\n      {\n        \"title\": \"Revenge of the Sith\"\n      }\n    ]\n  }\n}"
思考：我现在知道所有星球大战电影的标题
最终答案：所有星球大战电影的标题是：A New Hope, The Empire Strikes Back, Return of the Jedi, The Phantom Menace, Attack of the Clones, 和 Revenge of the Sith。
> 链结束。
```

```output
'所有星球大战电影的标题是：A New Hope, The Empire Strikes Back, Return of the Jedi, The Phantom Menace, Attack of the Clones, 和 Revenge of the Sith。'
```