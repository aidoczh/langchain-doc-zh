# neo4j-vector-memory

这个模板允许你将一个 LLM（Language Model）与基于向量的检索系统集成，使用 Neo4j 作为向量存储。此外，它利用 Neo4j 数据库的图形能力来存储和检索特定用户会话的对话历史。将对话历史存储为图形可以实现无缝的对话流程，同时还可以通过图形分析来分析用户行为和文本块的检索。

## 环境设置

你需要定义以下环境变量：

```
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

## 数据填充

如果你想用一些示例数据填充数据库，可以运行 `python ingest.py`。该脚本会处理并将文本文件 `dune.txt` 的部分内容存储到 Neo4j 图形数据库中。此外，还会为这些嵌入创建一个名为 `dune` 的向量索引，以便进行高效的查询。

## 使用方法

要使用这个包，你首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，可以执行以下操作：

```shell
langchain app new my-app --package neo4j-vector-memory
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add neo4j-vector-memory
```

然后在你的 `server.py` 文件中添加以下代码：

```python
from neo4j_vector_memory import chain as neo4j_vector_memory_chain
add_routes(app, neo4j_vector_memory_chain, path="/neo4j-vector-memory")
```

（可选）现在让我们配置 LangSmith。LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。你可以在[这里](https://smith.langchain.com/)注册 LangSmith。如果你没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果你在此目录中，可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将在本地启动一个 FastAPI 应用程序，服务器运行在 [http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上查看所有模板。

我们可以在 [http://127.0.0.1:8000/neo4j-vector-memory/playground](http://127.0.0.1:8000/neo4j-vector-memory/playground) 上访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/neo4j-vector-memory")
```