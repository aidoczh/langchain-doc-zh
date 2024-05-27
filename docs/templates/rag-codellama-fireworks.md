

# rag-codellama-fireworks

这个模板对代码库执行 RAG。

它使用 Fireworks 的 codellama-34b 托管的 [LLM 推理 API](https://blog.fireworks.ai/accelerating-code-completion-with-fireworks-fast-llm-inference-f4e8b5ec534a)。

## 环境设置

设置 `FIREWORKS_API_KEY` 环境变量以访问 Fireworks 模型。

您可以从 [这里](https://app.fireworks.ai/login?callbackURL=https://app.fireworks.ai) 获取它。

## 使用方法

要使用这个包，您应该首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，您可以执行：

```shell
langchain app new my-app --package rag-codellama-fireworks
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-codellama-fireworks
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_codellama_fireworks import chain as rag_codellama_fireworks_chain
add_routes(app, rag_codellama_fireworks_chain, path="/rag-codellama-fireworks")
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

如果您在此目录中，那么您可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/rag-codellama-fireworks/playground](http://127.0.0.1:8000/rag-codellama-fireworks/playground) 访问 playground

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-codellama-fireworks")
```