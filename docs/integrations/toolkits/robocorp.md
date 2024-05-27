# Robocorp

本笔记介绍如何开始使用 [Robocorp Action Server](https://github.com/robocorp/robocorp) 动作工具包和 LangChain。

Robocorp 是扩展 AI 代理、助手和副驾驶能力的最简单方式，可以使用自定义动作。

## 安装

首先，请参阅 [Robocorp Quickstart](https://github.com/robocorp/robocorp#quickstart) 了解如何设置 `Action Server` 并创建您的动作。

在您的 LangChain 应用程序中，安装 `langchain-robocorp` 包：

```python
# 安装包
%pip install --upgrade --quiet langchain-robocorp
```

当您按照上述快速入门创建新的 `Action Server` 时，它将创建一个包括 `action.py` 在内的文件目录。

我们可以像 [这里](https://github.com/robocorp/robocorp/tree/master/actions#describe-your-action) 所示的那样将 Python 函数添加为动作。

让我们向 `action.py` 添加一个虚拟函数。

```
@action
def get_weather_forecast(city: str, days: int, scale: str = "celsius") -> str:
    """
    返回给定城市的天气条件预报。
    Args:
        city (str): 要获取天气条件的目标城市
        days: 要返回的天数预报
        scale (str): 要使用的温度刻度，应为 "celsius" 或 "fahrenheit" 之一
    Returns:
        str: 请求的天气条件预报
    """
    return "75F and sunny :)"
```

然后启动服务器：

```
action-server start
```

我们可以看到：

```
找到新动作: get_weather_forecast
```

通过转到运行在 `http://localhost:8080` 的服务器并使用 UI 来运行该函数，可以在本地进行测试。

## 环境设置

可选择设置以下环境变量：

- `LANGCHAIN_TRACING_V2=true`：启用 LangSmith 日志运行跟踪，也可以绑定到相应的 Action Server 动作运行日志。有关更多信息，请参阅 [LangSmith 文档](https://docs.smith.langchain.com/tracing#log-runs)。

## 用法

我们在上面启动了本地动作服务器，运行在 `http://localhost:8080`。

```python
from langchain.agents import AgentExecutor, OpenAIFunctionsAgent
from langchain_core.messages import SystemMessage
from langchain_openai import ChatOpenAI
from langchain_robocorp import ActionServerToolkit
# 初始化 LLM 聊天模型
llm = ChatOpenAI(model="gpt-4", temperature=0)
# 初始化 Action Server Toolkit
toolkit = ActionServerToolkit(url="http://localhost:8080", report_trace=True)
tools = toolkit.get_tools()
# 初始化代理
system_message = SystemMessage(content="You are a helpful assistant")
prompt = OpenAIFunctionsAgent.create_prompt(system_message)
agent = OpenAIFunctionsAgent(llm=llm, prompt=prompt, tools=tools)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
executor.invoke("What is the current weather today in San Francisco in fahrenheit?")
```

```output
> 进入新的 AgentExecutor 链...
调用: `robocorp_action_server_get_weather_forecast`，参数为 `{'city': 'San Francisco', 'days': 1, 'scale': 'fahrenheit'}`
"75F and sunny :)"今天旧金山的当前天气是 75F 晴朗。
> 链结束。
```

```output
{'input': 'What is the current weather today in San Francisco in fahrenheit?',
 'output': '今天旧金山的当前天气是 75F 晴朗。'}
```

### 单输入工具

默认情况下，`toolkit.get_tools()` 将返回结构化工具作为动作。

要返回单输入工具，请传递一个用于处理输入的聊天模型。

```python
# 初始化单输入 Action Server Toolkit
toolkit = ActionServerToolkit(url="http://localhost:8080")
tools = toolkit.get_tools(llm=llm)
```