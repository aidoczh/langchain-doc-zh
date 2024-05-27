# rag-pinecone

该模板使用 Pinecone 和 OpenAI 执行 RAG。

## 环境设置

此模板使用 Pinecone 作为向量存储，并要求设置 `PINECONE_API_KEY`、`PINECONE_ENVIRONMENT` 和 `PINECONE_INDEX`。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 用法

要使用此软件包，您首先应安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建新的 LangChain 项目并将其安装为唯一软件包，您可以执行以下操作：

```shell
langchain app new my-app --package rag-pinecone
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-pinecone
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from rag_pinecone import chain as rag_pinecone_chain
add_routes(app, rag_pinecone_chain, path="/rag-pinecone")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

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

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/rag-pinecone/playground](http://127.0.0.1:8000/rag-pinecone/playground) 访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-pinecone")
```