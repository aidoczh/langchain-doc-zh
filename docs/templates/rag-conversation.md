

# rag-conversation

这个模板用于[对话式](https://python.langchain.com/docs/expression_language/cookbook/retrieval#conversational-retrieval-chain) [检索](https://python.langchain.com/docs/use_cases/question_answering/)，这是最流行的 LLM 应用之一。

它将对话历史和检索到的文档传递给 LLM 进行综合。

## 环境设置

此模板使用 Pinecone 作为向量存储，并要求设置 `PINECONE_API_KEY`、`PINECONE_ENVIRONMENT` 和 `PINECONE_INDEX`。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 用法

要使用此包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，您可以执行：

```shell
langchain app new my-app --package rag-conversation
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-conversation
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_conversation import chain as rag_conversation_chain
add_routes(app, rag_conversation_chain, path="/rag-conversation")
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

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板

我们可以在[http://127.0.0.1:8000/rag-conversation/playground](http://127.0.0.1:8000/rag-conversation/playground)访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-conversation")
```