# rag-lancedb

这个模板使用 LanceDB 和 OpenAI 执行 RAG。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 用法

要使用这个包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，您可以执行：

```shell
langchain app new my-app --package rag-lancedb
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-lancedb
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from rag_lancedb import chain as rag_lancedb_chain
add_routes(app, rag_lancedb_chain, path="/rag-lancedb")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，那么您可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/rag-lancedb/playground](http://127.0.0.1:8000/rag-lancedb/playground)访问 playground。

我们可以通过以下代码访问代码中的模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-lancedb")
```