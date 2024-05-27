# ChatTongyi

Tongyi Qwen 是阿里巴巴达摩院开发的一款大型语言模型。它能够通过自然语言理解和语义分析来理解用户意图，基于用户的自然语言输入。它为用户在不同领域和任务中提供服务和帮助。通过提供清晰详细的指令，您可以获得与您期望更符合的结果。

在这个笔记本中，我们将介绍如何在 `Chat` 中主要使用 [Tongyi](https://www.aliyun.com/product/dashscope) 来使用 langchain。

```python
# 安装包
%pip install --upgrade --quiet  dashscope
```

```output
注意：您可能需要重新启动内核以使用更新的软件包。
```

```python
# 获取新的令牌：https://help.aliyun.com/document_detail/611472.html?spm=a2c4g.2399481.0.0
from getpass import getpass
DASHSCOPE_API_KEY = getpass()
```

```python
import os
os.environ["DASHSCOPE_API_KEY"] = DASHSCOPE_API_KEY
```

```python
from langchain_community.chat_models.tongyi import ChatTongyi
from langchain_core.messages import HumanMessage
chatLLM = ChatTongyi(streaming=True)
res = chatLLM.stream([HumanMessage(content="hi")], streaming=True)
for r in res:
    print("chat resp:", r)
```

```output
chat resp: content='Hello' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content='!' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content=' How' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content=' can I assist you today' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content='?' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content='' response_metadata={'finish_reason': 'stop', 'request_id': '921db2c5-4d53-9a89-8e87-e4ad6a671237', 'token_usage': {'input_tokens': 20, 'output_tokens': 9, 'total_tokens': 29}} id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
```

```python
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(
        content="You are a helpful assistant that translates English to French."
    ),
    HumanMessage(
        content="Translate this sentence from English to French. I love programming."
    ),
]
chatLLM(messages)
```

```output
/Users/cheese/PARA/Projects/langchain-contribution/langchain/libs/core/langchain_core/_api/deprecation.py:119: LangChainDeprecationWarning: The method `BaseChatModel.__call__` was deprecated in langchain-core 0.1.7 and will be removed in 0.2.0. Use invoke instead.
  warn_deprecated(
```

```output
AIMessage(content="J'adore programmer.", response_metadata={'model_name': 'qwen-turbo', 'finish_reason': 'stop', 'request_id': 'ae725086-0ffa-9728-8c72-b204c7bc7eeb', 'token_usage': {'input_tokens': 36, 'output_tokens': 6, 'total_tokens': 42}}, id='run-060cc103-ef5f-4c8a-af40-792ac7f40c26-0')
```

## 工具调用

ChatTongyi 支持工具调用 API，可以描述工具及其参数，并让模型返回一个 JSON 对象，其中包含要调用的工具和该工具的输入。

### 使用 `bind_tools`

```python
from langchain_community.chat_models.tongyi import ChatTongyi
from langchain_core.tools import tool
@tool
def multiply(first_int: int, second_int: int) -> int:
    """Multiply two integers together."""
    return first_int * second_int
llm = ChatTongyi(model="qwen-turbo")
llm_with_tools = llm.bind_tools([multiply])
msg = llm_with_tools.invoke("What's 5 times forty two")
print(msg)
```

```output
content='' additional_kwargs={'tool_calls': [{'function': {'name': 'multiply', 'arguments': '{"first_int": 5, "second_int": 42}'}, 'id': '', 'type': 'function'}]} response_metadata={'model_name': 'qwen-turbo', 'finish_reason': 'tool_calls', 'request_id': '4acf0e36-44af-987a-a0c0-8b5c5eaa1a8b', 'token_usage': {'input_tokens': 200, 'output_tokens': 25, 'total_tokens': 225}} id='run-0ecd0f09-1d20-4e55-a4f3-f14d1f710ae7-0' tool_calls=[{'name': 'multiply', 'args': {'first_int': 5, 'second_int': 42}, 'id': ''}]
```

### 手动构造参数

```python
from langchain_community.chat_models.tongyi import ChatTongyi
from langchain_core.messages import HumanMessage, SystemMessage
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_time",
            "description": "当你想知道现在的时间时非常有用。",
            "parameters": {},
        },
    },
    {
        "type": "function",
        "function": {
            "name": "get_current_weather",
            "description": "当你想查询指定城市的天气时非常有用。",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "城市或县区，比如北京市、杭州市、余杭区等。",
                    }
                },
            },
            "required": ["location"],
        },
    },
]
messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="What is the weather like in San Francisco?"),
]
chatLLM = ChatTongyi()
llm_kwargs = {"tools": tools, "result_format": "message"}
ai_message = chatLLM.bind(**llm_kwargs).invoke(messages)
ai_message
```

## Tongyi With Vision

Qwen-VL（qwen-vl-plus/qwen-vl-max）是可以处理图像的模型。

```python
from langchain_community.chat_models import ChatTongyi
from langchain_core.messages import HumanMessage
chatLLM = ChatTongyi(model_name="qwen-vl-max")
image_message = {
    "image": "https://lilianweng.github.io/posts/2023-06-23-agent/agent-overview.png",
}
text_message = {
    "text": "summarize this picture",
}
message = HumanMessage(content=[text_message, image_message])
chatLLM.invoke([message])
```

AIMessage(content=[{'text': '这幅图片展示了一个人工智能系统的流程图。该系统分为两个主要组件：短期记忆和长期记忆，它们与“Memory”框相连。\n\n从“Memory”框出发，有三个分支通向不同的功能：\n\n1. “Tools” - 该分支代表人工智能系统可以利用的各种工具，包括“Calendar()”、“Calculator()”、“CodeInterpreter()”、“Search()”等未明确列出的其他工具。\n\n2. “Action” - 该分支代表人工智能系统根据其信息处理所采取的行动。它与“Tools”和“Agent”框均相连。\n\n3. “Planning” - 该分支代表人工智能系统的规划过程，涉及反思、自我批评、思维链、子目标分解等未显示的其他过程。\n\n系统的中心组件是“Agent”框，似乎协调着不同组件之间的信息流动。 “Agent”与“Tools”和“Memory”框交互，表明它在人工智能的决策过程中起着至关重要的作用。\n\n总体而言，该图片描绘了一个复杂而相互连接的人工智能系统，不同组件共同工作以处理信息、做出决策和采取行动。'}], response_metadata={'model_name': 'qwen-vl-max', 'finish_reason': 'stop', 'request_id': '6a2b9e90-7c3b-960d-8a10-6a0cf9991ae5', 'token_usage': {'input_tokens': 1262, 'output_tokens': 260, 'image_tokens': 1232}}, id='run-fd030661-c734-4580-b977-b77d42680742-0')