# 单人表演提示代理

该模板创建了一个代理，通过与多个人物进行多轮自我协作，将单个LLM转化为认知协同者。

认知协同者指的是与多个思维合作的智能代理，将它们各自的优势和知识结合起来，以增强复杂任务中的问题解决和整体性能。通过根据任务输入动态识别和模拟不同的人物角色，SPP释放了LLM中认知协同的潜力。

该模板将使用`DuckDuckGo`搜索API。

## 环境设置

该模板默认使用`OpenAI`。

请确保您的环境中设置了`OPENAI_API_KEY`。

## 使用方法

要使用此包，您首先应该安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其安装为唯一的包，可以执行以下操作：

```shell
langchain app new my-app --package solo-performance-prompting-agent
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add solo-performance-prompting-agent
```

并将以下代码添加到您的`server.py`文件中：

```python
from solo_performance_prompting_agent.agent import agent_executor as solo_performance_prompting_agent_chain
add_routes(app, solo_performance_prompting_agent_chain, path="/solo-performance-prompting-agent")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监视和调试LangChain应用程序。

您可以在[此处](https://smith.langchain.com/)注册LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，则可以直接启动LangServe实例：

```shell
langchain serve
```

这将在本地启动一个运行在[http://localhost:8000](http://localhost:8000)的FastAPI应用程序。

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)上查看所有模板。

我们可以在[http://127.0.0.1:8000/solo-performance-prompting-agent/playground](http://127.0.0.1:8000/solo-performance-prompting-agent/playground)上访问playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/solo-performance-prompting-agent")
```