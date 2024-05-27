# 研究助理

该模板实现了一个版本的[GPT Researcher](https://github.com/assafelovic/gpt-researcher)，您可以将其用作研究代理的起点。

## 环境设置

默认模板依赖于ChatOpenAI和DuckDuckGo，因此您需要以下环境变量：

- `OPENAI_API_KEY`

要使用Tavily LLM优化搜索引擎，您将需要：

- `TAVILY_API_KEY`

## 用法

要使用此软件包，您应首先安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package research-assistant
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add research-assistant
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from research_assistant import chain as research_assistant_chain
add_routes(app, research_assistant_chain, path="/research-assistant")
```

（可选）现在让我们配置LangSmith。LangSmith将帮助我们跟踪、监视和调试LangChain应用程序。您可以在[这里](https://smith.langchain.com/)注册LangSmith。如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，那么您可以直接启动LangServe实例：

```shell
langchain serve
```

这将启动一个FastAPI应用程序，服务器在本地运行，地址为[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板

我们可以在[http://127.0.0.1:8000/research-assistant/playground](http://127.0.0.1:8000/research-assistant/playground)访问playground

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/research-assistant")
```