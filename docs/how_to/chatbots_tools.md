# 如何为聊天机器人添加工具

本节将介绍如何创建对话代理：可以使用工具与其他系统和 API 进行交互的聊天机器人。

在阅读本指南之前，我们建议您先阅读本节中的[聊天机器人快速入门](/docs/tutorials/chatbot)，并熟悉[有关代理的文档](/docs/tutorials/agents)。

## 设置

在本指南中，我们将使用一个带有用于搜索网络的单个工具的[OpenAI 工具代理](/docs/how_to/agent_executor)。默认情况下，它将由[Tavily](/docs/integrations/tools/tavily_search)提供支持，但您也可以将其替换为任何类似的工具。本节的其余部分将假定您正在使用 Tavily。

您需要在[Tavily 网站](https://tavily.com/)上注册账户，并安装以下软件包：

```python
%pip install --upgrade --quiet langchain-openai tavily-python
```

```output
警告：您正在使用 pip 版本 22.0.4；然而，版本 23.3.2 可用。
您应该考虑通过 '/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip' 命令进行升级。
注意：您可能需要重新启动内核以使用更新后的软件包。
```

```output
True
```

您还需要将您的 OpenAI 密钥设置为 `OPENAI_API_KEY`，并将您的 Tavily API 密钥设置为 `TAVILY_API_KEY`。

## 创建代理

我们的最终目标是创建一个代理，它可以在需要时以对话方式回答用户的问题并查找信息。

首先，让我们初始化 Tavily 和一个能够调用工具的 OpenAI 聊天模型：

```python
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_openai import ChatOpenAI
tools = [TavilySearchResults(max_results=1)]
# 选择将驱动代理的 LLM
# 只有某些模型支持这一点
chat = ChatOpenAI(model="gpt-3.5-turbo-1106", temperature=0)
```

为了使我们的代理具有对话功能，我们还必须选择一个带有聊天历史占位符的提示。以下是一个示例：

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
# 改编自 https://smith.langchain.com/hub/hwchase17/openai-tools-agent
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. You may not need to use tools for every query - the user may just want to chat!",
        ),
        MessagesPlaceholder(variable_name="messages"),
        MessagesPlaceholder(variable_name="agent_scratchpad"),
    ]
)
```

太棒了！现在让我们组装我们的代理：

```python
from langchain.agents import AgentExecutor, create_openai_tools_agent
agent = create_openai_tools_agent(chat, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

## 运行代理

现在我们已经设置好我们的代理，让我们尝试与它进行交互！它可以处理既不需要查找的琐碎查询：

```python
from langchain_core.messages import HumanMessage
agent_executor.invoke({"messages": [HumanMessage(content="I'm Nemo!")]})
```

```output
> 进入新的 AgentExecutor 链...
你好，尼莫！很高兴见到你。我今天能帮你什么忙吗？
> 完成链。
```

```output
{'messages': [HumanMessage(content="I'm Nemo!")],
 'output': "你好，尼莫！很高兴见到你。我今天能帮你什么忙吗？"}
```

或者，它可以使用传递的搜索工具来获取最新信息（如果需要）：

```python
agent_executor.invoke(
    {
        "messages": [
            HumanMessage(
                content="What is the current conservation status of the Great Barrier Reef?"
            )
        ],
    }
)
```

```output
> 进入新的 AgentExecutor 链...
调用：`tavily_search_results_json`，参数为 `{'query': 'current conservation status of the Great Barrier Reef'}`
[{'url': 'https://www.barrierreef.org/news/blog/this-is-the-critical-decade-for-coral-reef-survival', 'content': "global coral reef conservation.  © 2024 Great Barrier Reef Foundation. Website by bigfish.tv  #Related News · 29 January 2024 290m more baby corals to help restore and protect the Great Barrier Reef  Great Barrier Reef Foundation Managing Director Anna Marsden says it’s not too late if we act now.The Status of Coral Reefs of the World: 2020 report is the largest analysis of global coral reef health ever undertaken. It found that 14 per cent of the world's coral has been lost since 2009. The report also noted, however, that some of these corals recovered during the 10 years to 2019."}]大堡礁的当前保育状况是一个重要的问题。根据大堡礁基金会的《世界珊瑚礁状况：2020年报告》，自2009年以来，全球14%的珊瑚已经消失。然而，报告还指出，其中一些珊瑚在2019年前的10年中已经恢复。欲了解更多信息，您可以访问以下链接：[大堡礁基金会 - 保育状况](https://www.barrierreef.org/news/blog/this-is-the-critical-decade-for-coral-reef-survival)
> 完成链。
```

当前大堡礁的保护状况非常令人担忧。根据大堡礁基金会的《世界珊瑚礁状况：2020年报告》，自2009年以来，全球14%的珊瑚已经消失。然而，报告还指出，其中一些珊瑚在2019年前的10年内已经恢复。欲了解更多信息，请访问以下链接：[大堡礁基金会 - 保护状况](https://www.barrierreef.org/news/blog/this-is-the-critical-decade-for-coral-reef-survival)