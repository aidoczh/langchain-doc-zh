# Slack

本笔记本将介绍如何将 LangChain 连接到您的 `Slack` 账户。

要使用此工具包，您需要获取一个在 [Slack API 文档](https://api.slack.com/tutorials/tracks/getting-a-token) 中解释的令牌。一旦您收到了 SLACK_USER_TOKEN，您可以在下面将其输入为环境变量。

```python
%pip install --upgrade --quiet  slack_sdk > /dev/null
%pip install --upgrade --quiet  beautifulsoup4 > /dev/null # 这是可选的，但用于解析 HTML 消息
%pip install --upgrade --quiet  python-dotenv > /dev/null # 用于从 .env 文件加载环境变量
```

```output
[notice] A new release of pip is available: 23.2.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
Note: you may need to restart the kernel to use updated packages.
[notice] A new release of pip is available: 23.2.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
Note: you may need to restart the kernel to use updated packages.
[notice] A new release of pip is available: 23.2.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
Note: you may need to restart the kernel to use updated packages.
```

## 设置环境变量

工具包将读取 SLACK_USER_TOKEN 环境变量以对用户进行身份验证，因此您需要在此处设置它们。您还需要设置您的 OPENAI_API_KEY 以便稍后使用代理。

```python
# 在这里设置环境变量
# 在此示例中，您可以通过加载 .env 文件来设置环境变量。
import dotenv
dotenv.load_dotenv()
```

```output
True
```

## 创建工具包并获取工具

首先，您需要创建工具包，以便稍后可以访问其工具。

```python
from langchain_community.agent_toolkits import SlackToolkit
toolkit = SlackToolkit()
tools = toolkit.get_tools()
tools
```

```output
[SlackGetChannel(client=<slack_sdk.web.client.WebClient object at 0x11eba6a00>),
 SlackGetMessage(client=<slack_sdk.web.client.WebClient object at 0x11eba69d0>),
 SlackScheduleMessage(client=<slack_sdk.web.client.WebClient object at 0x11eba65b0>),
 SlackSendMessage(client=<slack_sdk.web.client.WebClient object at 0x11eba6790>)]
```

## 在 ReAct 代理中使用

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_openai import ChatOpenAI
```

```python
llm = ChatOpenAI(temperature=0, model="gpt-4")
prompt = hub.pull("hwchase17/openai-tools-agent")
agent = create_openai_tools_agent(
    tools=toolkit.get_tools(),
    llm=llm,
    prompt=prompt,
)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

```python
agent_executor.invoke(
    {
        "input": "在 #general 频道向我的同事发送问候。请注意，使用 `channel` 作为频道 ID 的键，使用 `message` 作为要在频道中发送的内容的键。"
    }
)
```

```python
agent_executor.invoke(
    {"input": "工作区中有多少个频道？请列出它们的名称。"}
)
```

```output
> 进入新的 AgentExecutor 链...
我需要获取工作区中频道的列表。
操作：get_channelid_name_dict
操作输入：{}[{"id": "C052SCUP4UD", "name": "general", "created": 1681297313, "num_members": 1}, {"id": "C052VBBU4M8", "name": "test-bots", "created": 1681297343, "num_members": 2}, {"id": "C053805TNUR", "name": "random", "created": 1681297313, "num_members": 2}]我现在有了频道及其名称的列表。
最终答案：工作区中有 3 个频道。它们的名称分别是 "general"、"test-bots" 和 "random"。
> 链结束。
```

```output
{'input': '工作区中有多少个频道？请列出它们的名称。',
 'output': '工作区中有 3 个频道。它们的名称分别是 "general"、"test-bots" 和 "random"。'}
```

```python
agent_executor.invoke(
    {
        "input": "告诉我过去一个月在 #introductions 频道中发送的消息数量。"
    }
)
```

```output
> 进入新的 AgentExecutor 链...
首先，我需要确定 #introductions 频道的频道 ID。
操作：get_channelid_name_dict
操作输入：None[{"id": "C052SCUP4UD", "name": "general", "created": 1681297313, "num_members": 1}, {"id": "C052VBBU4M8", "name": "test-bots", "created": 1681297343, "num_members": 2}, {"id": "C053805TNUR", "name": "random", "created": 1681297313, "num_members": 2}]#introductions 频道未列在观察到的频道中。我需要通知用户 #introductions 频道不存在或无法访问。
最终答案：#introductions 频道不存在或无法访问。
> 链结束。
```

```output
{'input': '告诉我过去一个月在 #introductions 频道中发送的消息数量。',
 'output': '#introductions 频道不存在或无法访问。'}
```