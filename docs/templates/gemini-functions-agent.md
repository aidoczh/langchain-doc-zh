# gemini-functions-agent

这个模板创建了一个代理，使用Google Gemini函数调用来传达其决策以及采取的行动。

这个示例创建了一个代理，可以选择使用Tavily的搜索引擎在互联网上查找信息。

[点击此处查看一个LangSmith跟踪示例](https://smith.langchain.com/public/0ebf1bd6-b048-4019-b4de-25efe8d3d18c/r)

## 环境设置

需要设置以下环境变量：

将`TAVILY_API_KEY`环境变量设置为访问Tavily的API密钥。

将`GOOGLE_API_KEY`环境变量设置为访问Google Gemini API的密钥。

## 使用方法

要使用这个包，首先需要安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其安装为唯一的包，可以执行以下操作：

```shell
langchain app new my-app --package gemini-functions-agent
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add gemini-functions-agent
```

然后将以下代码添加到你的`server.py`文件中：

```python
from gemini_functions_agent import agent_executor as gemini_functions_agent_chain
add_routes(app, gemini_functions_agent_chain, path="/openai-functions-agent")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监控和调试LangChain应用程序。

你可以在[这里](https://smith.langchain.com/)注册LangSmith。

如果你没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果你在这个目录下，你可以直接启动一个LangServe实例：

```shell
langchain serve
```

这将在本地启动一个FastAPI应用程序，服务器正在运行在[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)上看到所有模板。

我们可以在[http://127.0.0.1:8000/gemini-functions-agent/playground](http://127.0.0.1:8000/gemini-functions-agent/playground)上访问playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/gemini-functions-agent")
```