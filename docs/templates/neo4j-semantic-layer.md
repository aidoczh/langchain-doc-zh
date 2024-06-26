# neo4j-semantic-layer

该模板旨在通过使用 OpenAI 函数调用来实现一个能够与 Neo4j 等图数据库进行交互的代理。语义层为代理提供了一套强大的工具，使其能够根据用户意图与图数据库进行交互。欲了解更多关于语义层模板的信息，请参阅[对应的博客文章](https://medium.com/towards-data-science/enhancing-interaction-between-language-models-and-graph-databases-via-a-semantic-layer-0a78ad3eba49)。

## 工具

代理利用多种工具有效地与 Neo4j 图数据库进行交互：

1. **信息工具**：

   - 获取有关电影或个人的数据，确保代理能够访问最新和最相关的信息。

2. **推荐工具**：

   - 根据用户偏好和输入提供电影推荐。

3. **记忆工具**：

   - 在知识图中存储有关用户偏好的信息，从而实现多次交互中的个性化体验。

## 环境设置

您需要定义以下环境变量：

```
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

## 数据填充

如果您想要使用示例电影数据集填充数据库，可以运行 `python ingest.py`。该脚本导入有关电影及其用户评分的信息。此外，该脚本创建了两个[全文索引](https://neo4j.com/docs/cypher-manual/current/indexes-for-full-text-search/)，用于将用户输入的信息映射到数据库中。

## 使用方法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行以下操作：

```shell
langchain app new my-app --package neo4j-semantic-layer
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add neo4j-semantic-layer
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from neo4j_semantic_layer import agent_executor as neo4j_semantic_agent
add_routes(app, neo4j_semantic_agent, path="/neo4j-semantic-layer")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[此处](https://smith.langchain.com/)注册 LangSmith。如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器将在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/neo4j-semantic-layer/playground](http://127.0.0.1:8000/neo4j-semantic-layer/playground)访问 playground。

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/neo4j-semantic-layer")
```