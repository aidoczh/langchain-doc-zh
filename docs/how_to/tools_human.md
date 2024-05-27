# 如何为工具添加人工审批环节

对于某些工具，我们不相信模型能够独立执行。在这种情况下，我们可以要求在调用工具之前进行人工审批。

:::info

本指南展示了在 Jupyter Notebook 或终端中为代码添加人工审批环节的简单方法。

要构建一个生产应用程序，您需要做更多的工作来适当地跟踪应用程序状态。

我们推荐使用 `langgraph` 来提供这种能力。更多详情，请参阅[指南](https://langchain-ai.github.io/langgraph/how-tos/human-in-the-loop/)。

:::

## 设置

我们需要安装以下软件包：

```python
%pip install --upgrade --quiet langchain
```

并设置以下环境变量：

```python
import getpass
import os
# 如果您想使用 LangSmith，请取消下面的注释：
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 链

让我们创建一些简单的（虚拟的）工具和一个调用工具的链：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm"/>

```python
from typing import Dict, List
from langchain_core.messages import AIMessage
from langchain_core.runnables import Runnable, RunnablePassthrough
from langchain_core.tools import tool
@tool
def count_emails(last_n_days: int) -> int:
    """将两个整数相乘。"""
    return last_n_days * 2
@tool
def send_email(message: str, recipient: str) -> str:
    "将两个整数相加。"
    return f"成功发送邮件给 {recipient}。"
tools = [count_emails, send_email]
llm_with_tools = llm.bind_tools(tools)
def call_tools(msg: AIMessage) -> List[Dict]:
    """简单的顺序工具调用辅助函数。"""
    tool_map = {tool.name: tool for tool in tools}
    tool_calls = msg.tool_calls.copy()
    for tool_call in tool_calls:
        tool_call["output"] = tool_map[tool_call["name"]].invoke(tool_call["args"])
    return tool_calls
chain = llm_with_tools | call_tools
chain.invoke("最近5天我收到了多少封电子邮件？")
```

```output
[{'name': 'count_emails',
  'args': {'last_n_days': 5},
  'id': 'toolu_01QYZdJ4yPiqsdeENWHqioFW',
  'output': 10}]
```

## 添加人工审批

让我们在链中添加一个步骤，该步骤将要求一个人批准或拒绝调用工具的请求。

在拒绝的情况下，该步骤将引发一个异常，停止执行链的其余部分。

```python
import json
class NotApproved(Exception):
    """自定义异常。"""
def human_approval(msg: AIMessage) -> AIMessage:
    """负责传递其输入或引发异常。
    Args:
        msg: 来自聊天模型的输出
    Returns:
        msg: 来自 msg 的原始输出
    """
    tool_strs = "\n\n".join(
        json.dumps(tool_call, indent=2) for tool_call in msg.tool_calls
    )
    input_msg = (
        f"您是否批准以下工具调用\n\n{tool_strs}\n\n"
        "除了 'Y'/'Yes'（不区分大小写）之外的任何回答都将被视为拒绝。\n >>>"
    )
    resp = input(input_msg)
    if resp.lower() not in ("yes", "y"):
        raise NotApproved(f"工具调用未获批准:\n\n{tool_strs}")
    return msg
```

```python
chain = llm_with_tools | human_approval | call_tools
chain.invoke("最近5天我收到了多少封电子邮件？")
```

```output
您是否批准以下工具调用
{
  "name": "count_emails",
  "args": {
    "last_n_days": 5
  },
  "id": "toolu_01WbD8XeMoQaRFtsZezfsHor"
}
除了 'Y'/'Yes'（不区分大小写）之外的任何回答都将被视为拒绝。
 >>> yes
```

```output
[{'name': 'count_emails',
  'args': {'last_n_days': 5},
  'id': 'toolu_01WbD8XeMoQaRFtsZezfsHor',
  'output': 10}]
```

```python
try:
    chain.invoke("给 sally@gmail.com 发送一封邮件，内容是 '最近怎么样？'")
except NotApproved as e:
    print()
    print(e)
```

```output
您是否批准以下工具调用
{
  "name": "send_email",
  "args": {
    "recipient": "sally@gmail.com",
    "message": "最近怎么样？"
  },
  "id": "toolu_014XccHFzBiVcc9GV1harV9U"
}
除了 'Y'/'Yes'（不区分大小写）之外的任何回答都将被视为拒绝。
 >>> no
``````output

工具调用未获批准:

{

  "name": "send_email",

  "args": {

    "recipient": "sally@gmail.com",

    "message": "最近怎么样？"

  },

  "id": "toolu_014XccHFzBiVcc9GV1harV9U"

}