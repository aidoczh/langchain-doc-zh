# OpenAI 函数代理

这个模板创建了一个代理，它使用 OpenAI 函数调用来传达其决策，以确定采取哪些行动。

这个示例创建了一个代理，可以选择使用 Tavily 的搜索引擎在互联网上查找信息。

## 环境设置

需要设置以下环境变量：

将 `OPENAI_API_KEY` 环境变量设置为访问 OpenAI 模型。

将 `TAVILY_API_KEY` 环境变量设置为访问 Tavily。

## 用法

要使用这个包，首先应安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，可以执行：

```shell
langchain app new my-app --package openai-functions-agent
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add openai-functions-agent
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from openai_functions_agent import agent_executor as openai_functions_agent_chain
add_routes(app, openai_functions_agent_chain, path="/openai-functions-agent")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

你可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果你没有访问权限，可以跳过这一部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果你在这个目录中，那么你可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/openai-functions-agent/playground](http://127.0.1:8000/openai-functions-agent/playground)访问 playground。

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/openai-functions-agent")
```