

# rag-matching-engine

这个模板使用 Google Cloud Platform 的 Vertex AI 来执行 RAG，并配备了匹配引擎。

它将利用先前创建的索引来根据用户提供的问题检索相关文档或上下文。

## 环境设置

在运行代码之前应创建一个索引。

创建此索引的过程可以在[这里](https://github.com/GoogleCloudPlatform/generative-ai/blob/main/language/use-cases/document-qa/question_answering_documents_langchain_matching_engine.ipynb)找到。

应设置用于 Vertex 的环境变量：

```
PROJECT_ID
ME_REGION
GCS_BUCKET
ME_INDEX_ID
ME_ENDPOINT_ID
```

## 用法

要使用此包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包安装为唯一包，您可以执行：

```shell
langchain app new my-app --package rag-matching-engine
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-matching-engine
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_matching_engine import chain as rag_matching_engine_chain
add_routes(app, rag_matching_engine_chain, path="/rag-matching-engine")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/rag-matching-engine/playground](http://127.0.0.1:8000/rag-matching-engine/playground)访问 playground。

我们可以通过以下方式从代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-matching-engine")
```

有关如何连接到模板的更多详细信息，请参考 Jupyter 笔记本 `rag_matching_engine`。