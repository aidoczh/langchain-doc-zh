

# rag-astradb

这个模板将使用 Astra DB 进行 RAG（`AstraDB` 向量存储类）。

## 环境设置

需要一个 [Astra DB](https://astra.datastax.com) 数据库；免费套餐也可以。

- 您需要数据库的 **API 端点**（例如 `https://0123...-us-east1.apps.astra.datastax.com`）...

- ... 以及一个 **token**（`AstraCS:...`）。

此外，还需要一个 **OpenAI API Key**。_请注意，除非您修改代码，否则此演示仅支持 OpenAI。_

通过环境变量提供连接参数和密钥。请参考 `.env.template` 获取变量名称。

## 用法

要使用这个包，您首先应该安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将其安装为唯一包，可以执行：

```shell
langchain app new my-app --package rag-astradb
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-astradb
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from astradb_entomology_rag import chain as astradb_entomology_rag_chain
add_routes(app, astradb_entomology_rag_chain, path="/rag-astradb")
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

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/rag-astradb/playground](http://127.0.0.1:8000/rag-astradb/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-astradb")
```

## 参考

包含 LangServe chain 的独立存储库：[这里](https://github.com/hemidactylus/langserve_astradb_entomology_rag)。