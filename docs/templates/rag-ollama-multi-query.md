# rag-ollama-multi-query

该模板使用 Ollama 和 OpenAI 进行 RAG，使用了多查询检索器。

多查询检索器是一个查询转换的示例，根据用户的输入查询从不同的角度生成多个查询。

对于每个查询，它检索一组相关文档，并对所有查询的唯一并集进行答案综合。

我们使用私有的本地 LLM 来进行狭窄任务的查询生成，以避免对更大的 LLM API 进行过多调用。

查看 Ollama LLM 执行查询扩展的示例跟踪[这里](https://smith.langchain.com/public/8017d04d-2045-4089-b47f-f2d66393a999/r)。

但是，我们使用 OpenAI 来进行更具挑战性的答案综合任务（完整跟踪示例[在这里](https://smith.langchain.com/public/ec75793b-645b-498d-b855-e8d85e1f6738/r)）。

## 环境设置

要设置环境，您需要下载 Ollama。

请按照[这里](https://python.langchain.com/docs/integrations/chat/ollama)的说明进行操作。

您可以使用 Ollama 选择所需的 LLM。

此模板使用 `zephyr`，可以使用 `ollama pull zephyr` 进行访问。

还有许多其他选项可在[这里](https://ollama.ai/library)找到。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 用法

要使用此软件包，您首先应安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建新的 LangChain 项目并安装此软件包，请执行：

```shell
langchain app new my-app --package rag-ollama-multi-query
```

要将此软件包添加到现有项目中，请运行：

```shell
langchain app add rag-ollama-multi-query
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_ollama_multi_query import chain as rag_ollama_multi_query_chain
add_routes(app, rag_ollama_multi_query_chain, path="/rag-ollama-multi-query")
```

（可选）现在，让我们配置 LangSmith。LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。您可以在[这里](https://smith.langchain.com/)注册 LangSmith。如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将在本地启动一个运行服务器的 FastAPI 应用程序，地址为 [http://localhost:8000](http://localhost:8000)

您可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)查看所有模板

您可以在[http://127.0.0.1:8000/rag-ollama-multi-query/playground](http://127.0.0.1:8000/rag-ollama-multi-query/playground)访问 playground

要从代码中访问模板，请使用：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-ollama-multi-query")
```