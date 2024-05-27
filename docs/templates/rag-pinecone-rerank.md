# rag-pinecone-rerank

该模板使用 Pinecone 和 OpenAI 进行 RAG，并利用 [Cohere 进行重新排序](https://txt.cohere.com/rerank/) 返回的文档。

重新排序提供了一种使用指定的过滤器或标准对检索到的文档进行排序的方法。

## 环境设置

该模板使用 Pinecone 作为向量存储，并需要设置 `PINECONE_API_KEY`、`PINECONE_ENVIRONMENT` 和 `PINECONE_INDEX`。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

设置 `COHERE_API_KEY` 环境变量以访问 Cohere ReRank。

## 用法

要使用此软件包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行以下操作：

```shell
langchain app new my-app --package rag-pinecone-rerank
```

如果要将此软件包添加到现有项目中，只需运行：

```shell
langchain app add rag-pinecone-rerank
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_pinecone_rerank import chain as rag_pinecone_rerank_chain
add_routes(app, rag_pinecone_rerank_chain, path="/rag-pinecone-rerank")
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

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/rag-pinecone-rerank/playground](http://127.0.0.1:8000/rag-pinecone-rerank/playground) 访问 playground

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-pinecone-rerank")
```