# 检索代理

该软件包使用 Azure OpenAI 来使用代理架构进行检索。

默认情况下，这将在 Arxiv 上进行检索。

## 环境设置

由于我们在使用 Azure OpenAI，我们需要设置以下环境变量：

```shell
export AZURE_OPENAI_ENDPOINT=...
export AZURE_OPENAI_API_VERSION=...
export AZURE_OPENAI_API_KEY=...
```

## 用法

要使用此软件包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package retrieval-agent
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add retrieval-agent
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from retrieval_agent import chain as retrieval_agent_chain
add_routes(app, retrieval_agent_chain, path="/retrieval-agent")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[此处](https://smith.langchain.com/)注册 LangSmith。

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

我们可以在 [http://127.0.0.1:8000/retrieval-agent/playground](http://127.0.0.1:8000/retrieval-agent/playground) 访问 playground

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/retrieval-agent")
```