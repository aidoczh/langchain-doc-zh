# rag-chroma-private

该模板执行 RAG，不依赖外部 API。

它利用 Ollama 作为 LLM，GPT4All 用于嵌入，以及 Chroma 用于向量存储。

向量存储在 `chain.py` 中创建，默认情况下索引 [Agents 的热门博客文章](https://lilianweng.github.io/posts/2023-06-23-agent/) 以进行问答。

## 环境设置

要设置环境，您需要下载 Ollama。

请按照 [这里](https://python.langchain.com/docs/integrations/chat/ollama) 的说明进行操作。

您可以使用 Ollama 选择所需的 LLM。

此模板使用 `llama2:7b-chat`，可通过 `ollama pull llama2:7b-chat` 访问。

还有许多其他选项可在 [这里](https://ollama.ai/library) 找到。

此软件包还使用 [GPT4All](https://python.langchain.com/docs/integrations/text_embedding/gpt4all) 嵌入。

## 用法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建新的 LangChain 项目并将其安装为唯一软件包，可以执行：

```shell
langchain app new my-app --package rag-chroma-private
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-chroma-private
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_chroma_private import chain as rag_chroma_private_chain
add_routes(app, rag_chroma_private_chain, path="/rag-chroma-private")
```

（可选）现在让我们配置 LangSmith。LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行在 

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/rag-chroma-private/playground](http://127.0.0.1:8000/rag-chroma-private/playground) 访问 playground

我们可以通过以下方式从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-chroma-private")
```

该软件包将在 `chain.py` 中创建并添加文档到向量数据库。默认情况下，它将加载一个关于 agents 的热门博客文章。但是，您可以在 [这里](https://python.langchain.com/docs/integrations/document_loaders) 选择大量的文档加载器。