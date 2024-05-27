# SQL-pgvector

这个模板使用户能够将 `pgvector` 与语义搜索 / RAG 结合在一起使用 PostgreSQL。

它使用 [PGVector](https://github.com/pgvector/pgvector) 扩展，如 [RAG empowered SQL cookbook](https://github.com/langchain-ai/langchain/blob/master/cookbook/retrieval_in_sql.ipynb) 中所示。

## 环境设置

如果您正在使用 `ChatOpenAI` 作为您的LLM，请确保在您的环境中设置了 `OPENAI_API_KEY`。您可以在 `chain.py` 中更改LLM和嵌入模型。

您可以配置以下环境变量供模板使用（默认值用括号表示）

- `POSTGRES_USER` (postgres)

- `POSTGRES_PASSWORD` (test)

- `POSTGRES_DB` (vectordb)

- `POSTGRES_HOST` (localhost)

- `POSTGRES_PORT` (5432)

如果您没有 postgres 实例，您可以在本地运行一个docker：

```bash
docker run \
  --name some-postgres \
  -e POSTGRES_PASSWORD=test \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=vectordb \
  -p 5432:5432 \
  postgres:16
```

以后再启动时，使用上面定义的 `--name`：

```bash
docker start some-postgres
```

### PostgreSQL数据库设置

除了启用 `pgvector` 扩展之外，您还需要在能够在SQL查询中运行语义搜索之前进行一些设置。

为了在您的 PostgreSQL 数据库上运行 RAG，您需要为您想要的特定列生成嵌入。

这个过程在 [RAG empowered SQL cookbook](https://github.com/langchain-ai/langchain/blob/master/cookbook/retrieval_in_sql.ipynb) 中有介绍，但总体方法包括：

1. 查询列中的唯一值

2. 为这些值生成嵌入

3. 将嵌入存储在单独的列或辅助表中。

## 用法

要使用这个包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，您可以执行：

```shell
langchain app new my-app --package sql-pgvector
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add sql-pgvector
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from sql_pgvector import chain as sql_pgvector_chain
add_routes(app, sql_pgvector_chain, path="/sql-pgvector")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在这个目录中，那么您可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/sql-pgvector/playground](http://127.0.0.1:8000/sql-pgvector/playground) 访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/sql-pgvector")
```