# rag-azure-search

这个模板使用 [Azure AI Search](https://learn.microsoft.com/azure/search/search-what-is-azure-search) 作为向量存储库，并使用 Azure OpenAI 的聊天和嵌入模型执行 RAG。

有关在 Azure AI Search 中使用 RAG 的更多详细信息，请参考[此笔记本](https://github.com/langchain-ai/langchain/blob/master/docs/docs/integrations/vectorstores/azuresearch.ipynb)。

## 环境设置

***先决条件：*** 需要已有 [Azure AI Search](https://learn.microsoft.com/azure/search/search-what-is-azure-search) 和 [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/overview) 资源。

***环境变量:***

要运行这个模板，您需要设置以下环境变量：

***必需的:***

- AZURE_SEARCH_ENDPOINT - Azure AI Search 服务的终结点。

- AZURE_SEARCH_KEY - Azure AI Search 服务的 API 密钥。

- AZURE_OPENAI_ENDPOINT - Azure OpenAI 服务的终结点。

- AZURE_OPENAI_API_KEY - Azure OpenAI 服务的 API 密钥。

- AZURE_EMBEDDINGS_DEPLOYMENT - 用于嵌入的 Azure OpenAI 部署的名称。

- AZURE_CHAT_DEPLOYMENT - 用于聊天的 Azure OpenAI 部署的名称。

***可选的:***

- AZURE_SEARCH_INDEX_NAME - 要使用的现有 Azure AI Search 索引的名称。如果未提供，将创建一个名称为 "rag-azure-search" 的索引。

- OPENAI_API_VERSION - 要使用的 Azure OpenAI API 版本。默认为 "2023-05-15"。

## 用法

要使用这个包，您应该首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，您可以执行：

```shell
langchain app new my-app --package rag-azure-search
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-azure-search
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_azure_search import chain as rag_azure_search_chain
add_routes(app, rag_azure_search_chain, path="/rag-azure-search")
```

(可选) 现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，那么您可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动带有服务器的 FastAPI 应用程序，本地运行的服务器位于

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/rag-azure-search/playground](http://127.0.0.1:8000/rag-azure-search/playground) 访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-azure-search")
```