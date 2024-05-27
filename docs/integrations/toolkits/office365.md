# Office365

[Microsoft 365](https://www.office.com/) 是由微软公司拥有的一系列办公软件、协作和基于云的服务产品。

注：`Office 365` 已更名为 `Microsoft 365`。

本手册介绍了如何将 LangChain 连接到 `Office365` 电子邮件和日历。

要使用此工具包，您需要设置您的凭据，详细说明请参阅[Microsoft Graph 认证和授权概述](https://learn.microsoft.com/en-us/graph/auth/)。一旦您收到了 CLIENT_ID 和 CLIENT_SECRET，您可以在下面的环境变量中输入它们。

您还可以在[此处查看认证说明](https://o365.github.io/python-o365/latest/getting_started.html#oauth-setup-pre-requisite)。

```python
%pip install --upgrade --quiet  O365
%pip install --upgrade --quiet  beautifulsoup4  # 这是可选的，但用于解析 HTML 消息很有用
```

## 分配环境变量

工具包将读取 `CLIENT_ID` 和 `CLIENT_SECRET` 环境变量以对用户进行身份验证，因此您需要在此处设置它们。您还需要设置您的 `OPENAI_API_KEY` 以便稍后使用代理。

```python
# 在这里设置环境变量
```

## 创建工具包并获取工具

首先，您需要创建工具包，以便以后可以访问其工具。

```python
from langchain_community.agent_toolkits import O365Toolkit
toolkit = O365Toolkit()
tools = toolkit.get_tools()
tools
```

```output
[O365SearchEvents(name='events_search', description=" 使用此工具搜索用户日历事件。输入必须是搜索查询的开始和结束日期时间。输出是用户日历中在开始和结束时间之间的所有事件的 JSON 列表。您可以假设用户不能安排任何会议，而且在会议期间用户都很忙。没有事件的时间对用户来说是空闲的。 ", args_schema=<class 'langchain_community.tools.office365.events_search.SearchEventsInput'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302),
 O365CreateDraftMessage(name='create_email_draft', description='使用此工具使用提供的消息字段创建草稿电子邮件。', args_schema=<class 'langchain_community.tools.office365.create_draft_message.CreateDraftMessageSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302),
 O365SearchEmails(name='messages_search', description='使用此工具搜索电子邮件消息。输入必须是有效的 Microsoft Graph v1.0 $search 查询。输出是所请求资源的 JSON 列表。', args_schema=<class 'langchain_community.tools.office365.messages_search.SearchEmailsInput'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302),
 O365SendEvent(name='send_event', description='使用此工具使用提供的事件字段创建并发送事件。', args_schema=<class 'langchain_community.tools.office365.send_event.SendEventSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302),
 O365SendMessage(name='send_email', description='使用此工具使用提供的消息字段发送电子邮件。', args_schema=<class 'langchain_community.tools.office365.send_message.SendMessageSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302)]
```

## 在代理中使用

```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import OpenAI
```

```python
llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=toolkit.get_tools(),
    llm=llm,
    verbose=False,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
)
```

```python
agent.run(
    "Create an email draft for me to edit of a letter from the perspective of a sentient parrot"
    " who is looking to collaborate on some research with her"
    " estranged friend, a cat. Under no circumstances may you send the message, however."
)
```

```output
'草稿电子邮件已正确创建。'
```

```python
agent.run(
    "Could you search in my drafts folder and let me know if any of them are about collaboration?"
)
```

```output
"我在您的草稿文件夹中找到一份关于合作的草稿。它是在 2023-06-16T18:22:17+0000 发送的，主题是'合作请求'。"
```

```python
agent.run(
    "Can you schedule a 30 minute meeting with a sentient parrot to discuss research collaborations on October 3, 2023 at 2 pm Easter Time?"
)
```

```output
/home/vscode/langchain-py-env/lib/python3.11/site-packages/O365/utils/windows_tz.py:639: PytzUsageWarning: zone 属性是特定于 pytz 接口的；请迁移到新的时区提供程序。有关如何迁移的详细信息，请参见 https://pytz-deprecation-shim.readthedocs.io/en/latest/migration.html
  iana_tz.zone if isinstance(iana_tz, tzinfo) else iana_tz)
/home/vscode/langchain-py-env/lib/python3.11/site-packages/O365/utils/utils.py:463: PytzUsageWarning: zone 属性是特定于 pytz 接口的；请迁移到新的时区提供程序。有关如何迁移的详细信息，请参见 https://pytz-deprecation-shim.readthedocs.io/en/latest/migration.html
  timezone = date_time.tzinfo.zone if date_time.tzinfo is not None else None
```

```output
'我已经安排了与一只有思维的鹦鹉在2023年10月3日下午2点进行研究合作的会议。请告诉我是否需要进行任何更改。'
```

```python
agent.run(
    "你能告诉我2023年10月3日在东部时间是否有任何事件，如果有的话，告诉我是否有任何事件是与一只有思维的鹦鹉相关的？"
)
```

```output
"是的，你在2023年10月3日与一只有思维的鹦鹉有一个事件。该事件标题为'与有思维的鹦鹉会面'，计划从下午6:00到6:30。"
```