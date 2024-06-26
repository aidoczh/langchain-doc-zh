# cassandra-synonym-caching

这个模板提供了一个简单的链式模板，展示了使用基于 Apache Cassandra® 或 Astra DB 的 LLM 缓存的用法。

## 环境设置

要设置您的环境，您需要以下内容：

- 一个 [Astra](https://astra.datastax.com) Vector 数据库（免费版即可！）。**您需要一个 [数据库管理员令牌](https://awesome-astra.github.io/docs/pages/astra/create-token/#c-procedure)**，特别是以 `AstraCS:...` 开头的字符串；

- 同样地，准备好您的 [数据库 ID](https://awesome-astra.github.io/docs/pages/astra/faq/#where-should-i-find-a-database-identifier)，您将需要在下面输入它；

- 一个 **OpenAI API 密钥**。（更多信息请参见[这里](https://cassio.org/start_here/#llm-access)，请注意，除非您修改代码，否则此演示默认支持 OpenAI。）

_注意：_您也可以选择使用常规的 Cassandra 集群：为此，请确保按照 `.env.template` 中所示提供 `USE_CASSANDRA_CLUSTER` 条目以及后续的环境变量，以指定如何连接到它。

## 使用方法

要使用此包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，可以执行以下操作：

```shell
langchain app new my-app --package cassandra-synonym-caching
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add cassandra-synonym-caching
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from cassandra_synonym_caching import chain as cassandra_synonym_caching_chain
add_routes(app, cassandra_synonym_caching_chain, path="/cassandra-synonym-caching")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将在本地启动一个运行在 [http://localhost:8000](http://localhost:8000) 的 FastAPI 应用程序服务器。

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上查看所有模板。

我们可以在 [http://127.0.0.1:8000/cassandra-synonym-caching/playground](http://127.0.0.1:8000/cassandra-synonym-caching/playground) 上访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/cassandra-synonym-caching")
```

## 参考

独立的 LangServe 模板仓库：[这里](https://github.com/hemidactylus/langserve_cassandra_synonym_caching)。