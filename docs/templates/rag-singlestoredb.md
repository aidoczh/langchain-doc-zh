# rag-singlestoredb

这个模板使用 SingleStoreDB 和 OpenAI 执行 RAG。

## 环境设置

该模板将 SingleStoreDB 用作向量存储，并需要设置 `SINGLESTOREDB_URL`。它应该采用以下形式：`admin:password@svc-xxx.svc.singlestore.com:port/db_name`

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用方法

要使用此软件包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行以下操作：

```shell
langchain app new my-app --package rag-singlestoredb
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-singlestoredb
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from rag_singlestoredb import chain as rag_singlestoredb_chain
add_routes(app, rag_singlestoredb_chain, path="/rag-singlestoredb")
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

我们可以在 [http://127.0.0.1:8000/rag-singlestoredb/playground](http://127.0.0.1:8000/rag-singlestoredb/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-singlestoredb")
```