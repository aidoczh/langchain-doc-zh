# 海盗语言配置

这个模板将用户输入转换为海盗语言。它展示了如何在 Runnable 中允许 `configurable_alternatives`，让您可以在 playground（或通过 API）中选择 OpenAI、Anthropic 或 Cohere 作为您的 LLM 提供商。

## 环境设置

设置以下环境变量以访问所有 3 个可配置的替代模型提供商：

- `OPENAI_API_KEY`

- `ANTHROPIC_API_KEY`

- `COHERE_API_KEY`

## 使用方法

要使用这个包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package pirate-speak-configurable
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add pirate-speak-configurable
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from pirate_speak_configurable import chain as pirate_speak_configurable_chain
add_routes(app, pirate_speak_configurable_chain, path="/pirate-speak-configurable")
```

（可选）现在让我们配置 LangSmith。LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。您可以在[这里](https://smith.langchain.com/)注册 LangSmith。如果您没有访问权限，可以跳过此部分。

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

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/pirate-speak-configurable/playground](http://127.0.0.1:8000/pirate-speak-configurable/playground) 访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/pirate-speak-configurable")
```