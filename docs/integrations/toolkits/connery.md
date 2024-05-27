# 康尼工具包

使用这个工具包，你可以将康尼动作集成到你的 LangChain 代理中。

如果你只想在你的代理中使用特定的康尼动作，查看[康尼动作工具](/docs/integrations/tools/connery)文档。

## 什么是康尼？

康尼是一个用于人工智能的开源插件基础架构。

通过康尼，你可以轻松地创建一个自定义插件，其中包含一组动作，并将它们无缝地集成到你的 LangChain 代理中。

康尼将处理关键方面，如运行时、授权、秘钥管理、访问管理、审计日志和其他重要功能。

此外，得益于我们的社区支持，康尼提供了一个多样化的开源插件集合，以增加便利性。

了解更多关于康尼的信息：

- GitHub: https://github.com/connery-io/connery

- 文档: https://docs.connery.io

## 先决条件

要在你的 LangChain 代理中使用康尼动作，你需要做一些准备工作：

1. 使用[快速入门](https://docs.connery.io/docs/runner/quick-start/)指南设置康尼运行器。

2. 安装所有你想在代理中使用的插件及其动作。

3. 设置环境变量 `CONNERY_RUNNER_URL` 和 `CONNERY_RUNNER_API_KEY`，以便工具包可以与康尼运行器进行通信。

## 使用康尼工具包的示例

在下面的示例中，我们创建了一个代理，使用两个康尼动作来总结一个公共网页，并通过电子邮件发送摘要：

1. 来自[总结](https://github.com/connery-io/summarization-plugin)插件的**总结公共网页**动作。

2. 来自[Gmail](https://github.com/connery-io/gmail)插件的**发送电子邮件**动作。

你可以在这里查看此示例的 LangSmith 跟踪：[链接](https://smith.langchain.com/public/4af5385a-afe9-46f6-8a53-57fe2d63c5bc/r)。

```python
import os
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.connery import ConneryToolkit
from langchain_community.tools.connery import ConneryService
from langchain_openai import ChatOpenAI
# 指定你的康尼运行器凭据。
os.environ["CONNERY_RUNNER_URL"] = ""
os.environ["CONNERY_RUNNER_API_KEY"] = ""
# 指定 OpenAI API 密钥。
os.environ["OPENAI_API_KEY"] = ""
# 指定你的电子邮件地址，以接收来自下面示例的摘要电子邮件。
recepient_email = "test@example.com"
# 创建一个包含来自康尼运行器的所有可用动作的康尼工具包。
connery_service = ConneryService()
connery_toolkit = ConneryToolkit.create_instance(connery_service)
# 使用 OpenAI 函数代理执行提示，使用康尼工具包中的动作。
llm = ChatOpenAI(temperature=0)
agent = initialize_agent(connery_toolkit.get_tools(), llm, AgentType.OPENAI_FUNCTIONS, verbose=True)
result = agent.run(
    f"""Make a short summary of the webpage http://www.paulgraham.com/vb.html in three sentences
and send it to {recepient_email}. Include the link to the webpage into the body of the email."""
)
print(result)
```

```output
> 进入新的 AgentExecutor 链...
调用: `CA72DFB0AB4DF6C830B43E14B0782F70`，参数为 `{'publicWebpageUrl': 'http://www.paulgraham.com/vb.html'}`
{'summary': '作者反思了生命短暂的概念，以及如何有了孩子让他们意识到生命的真正短暂性。他们讨论了时间如何可以转化为离散的数量，以及某些经历的有限性。作者强调了在生活中优先考虑和消除不必要的事物的重要性，以及积极追求有意义的经历。他们还讨论了陷入在线争论的负面影响，以及需要意识到时间如何被消耗。作者建议修剪不必要的活动，不要等待去做重要的事情，并珍惜自己拥有的时间。'}
调用: `CABC80BB79C15067CA983495324AE709`，参数为 `{'recipient': 'test@example.com', 'subject': 'Summary of the webpage', 'body': 'Here is a short summary of the webpage http://www.paulgraham.com/vb.html:\n\nThe author reflects on the concept of life being short and how having children made them realize the true brevity of life. They discuss how time can be converted into discrete quantities and how limited certain experiences are. The author emphasizes the importance of prioritizing and eliminating unnecessary things in life, as well as actively pursuing meaningful experiences. They also discuss the negative impact of getting caught up in online arguments and the need to be aware of how time is being spent. The author suggests pruning unnecessary activities, not waiting to do things that matter, and savoring the time one has.\n\nYou can find the full webpage [here](http://www.paulgraham.com/vb.html).'}`
{'messageId': '<2f04b00e-122d-c7de-c91e-e78e0c3276d6@gmail.com>'}我已经向 test@example.com 发送了包含网页摘要的电子邮件。请检查你的收件箱。
> 链结束。
我已经向 test@example.com 发送了包含网页摘要的电子邮件。请检查你的收件箱。
```

注意：Connery Action 是一种结构化工具，因此只能在支持结构化工具的代理程序中使用。