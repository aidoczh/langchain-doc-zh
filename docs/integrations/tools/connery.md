# Connery 动作工具

使用此工具，您可以将单个 Connery 动作集成到您的 LangChain 代理中。

如果您想在代理中使用多个 Connery 动作，请查看 [Connery Toolkit](/docs/integrations/toolkits/connery) 文档。

## 什么是 Connery?

Connery 是一个面向人工智能的开源插件基础架构。

使用 Connery，您可以轻松创建一个自定义插件，其中包含一组动作，并将其无缝集成到您的 LangChain 代理中。

Connery 将负责关键方面，如运行时、授权、秘钥管理、访问管理、审计日志和其他重要功能。

此外，Connery 得到社区的支持，提供了丰富的开源插件集合，可供您方便使用。

了解更多关于 Connery 的信息：

- GitHub: https://github.com/connery-io/connery

- 文档: https://docs.connery.io

## 先决条件

要在您的 LangChain 代理中使用 Connery 动作，您需要进行一些准备工作：

1. 使用 [快速入门](https://docs.connery.io/docs/runner/quick-start/) 指南设置 Connery 运行器。

2. 安装您想在代理中使用的所有带有动作的插件。

3. 设置环境变量 `CONNERY_RUNNER_URL` 和 `CONNERY_RUNNER_API_KEY`，以便工具包可以与 Connery 运行器进行通信。

## 使用 Connery 动作工具的示例

在下面的示例中，我们通过其 ID 从 Connery 运行器中获取动作，然后使用指定的参数调用它。

在这里，我们使用来自 [Gmail](https://github.com/connery-io/gmail) 插件的 **发送电子邮件** 动作的 ID。

```python
import os
from langchain.agents import AgentType, initialize_agent
from langchain_community.tools.connery import ConneryService
from langchain_openai import ChatOpenAI
# 指定您的 Connery 运行器凭据。
os.environ["CONNERY_RUNNER_URL"] = ""
os.environ["CONNERY_RUNNER_API_KEY"] = ""
# 指定 OpenAI API 密钥。
os.environ["OPENAI_API_KEY"] = ""
# 指定您的电子邮件地址，以接收下面示例中的电子邮件。
recepient_email = "test@example.com"
# 通过 ID 从 Connery 运行器获取发送电子邮件动作。
connery_service = ConneryService()
send_email_action = connery_service.get_action("CABC80BB79C15067CA983495324AE709")
```

手动运行动作。

```python
manual_run_result = send_email_action.run(
    {
        "recipient": recepient_email,
        "subject": "Test email",
        "body": "This is a test email sent from Connery.",
    }
)
print(manual_run_result)
```

使用 OpenAI Functions 代理运行动作。

您可以在此处查看此示例的 LangSmith 追踪 [链接](https://smith.langchain.com/public/a37d216f-c121-46da-a428-0e09dc19b1dc/r)。

```python
llm = ChatOpenAI(temperature=0)
agent = initialize_agent(
    [send_email_action], llm, AgentType.OPENAI_FUNCTIONS, verbose=True
)
agent_run_result = agent.run(
    f"Send an email to the {recepient_email} and say that I will be late for the meeting."
)
print(agent_run_result)
```

```output
> 进入新的 AgentExecutor 链...
调用: `CABC80BB79C15067CA983495324AE709`，参数为 `{'recipient': 'test@example.com', 'subject': 'Late for Meeting', 'body': 'Dear Team,\n\nI wanted to inform you that I will be late for the meeting today. I apologize for any inconvenience caused. Please proceed with the meeting without me and I will join as soon as I can.\n\nBest regards,\n[Your Name]'}`
{'messageId': '<d34a694d-50e0-3988-25da-e86b4c51d7a7@gmail.com>'}我已向 test@example.com 发送了一封电子邮件，告知您将会迟到参加会议。
> 链结束。
我已向 test@example.com 发送了一封电子邮件，告知您将会迟到参加会议。
```

注意：Connery 动作是一个结构化工具，因此只能在支持结构化工具的代理中使用。