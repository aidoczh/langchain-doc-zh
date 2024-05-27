# neo4j_cypher

这个模板允许您使用 OpenAI LLM 以自然语言与 Neo4j 图数据库进行交互。

它将自然语言问题转换为 Cypher 查询（用于从 Neo4j 数据库中获取数据），执行查询，并根据查询结果提供自然语言响应。

[](https://medium.com/neo4j/langchain-cypher-search-tips-tricks-f7c9e9abca4d)

## 环境设置

定义以下环境变量：

```
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

## Neo4j 数据库设置

有多种设置 Neo4j 数据库的方法。

### Neo4j Aura

Neo4j AuraDB 是一个完全托管的云图数据库服务。

在 [Neo4j Aura](https://neo4j.com/cloud/platform/aura-graph-database?utm_source=langchain&utm_content=langserve) 上创建一个免费实例。

当您初始化一个免费数据库实例时，您将收到访问数据库的凭据。

## 数据填充

如果您想用一些示例数据填充数据库，可以运行 `python ingest.py`。

这个脚本将使用示例电影数据填充数据库。

## 用法

要使用这个包，您应该首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，您可以执行：

```shell
langchain app new my-app --package neo4j-cypher
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add neo4j-cypher
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from neo4j_cypher import chain as neo4j_cypher_chain
add_routes(app, neo4j_cypher_chain, path="/neo4j-cypher")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，那么您可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行在

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上看到所有模板

我们可以在 [http://127.0.0.1:8000/neo4j_cypher/playground](http://127.0.0.1:8000/neo4j_cypher/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/neo4j-cypher")
```