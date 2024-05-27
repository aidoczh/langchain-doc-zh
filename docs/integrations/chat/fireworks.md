---

sidebar_label: 烟火

---

# 聊天烟火

>[烟火](https://app.fireworks.ai/)通过创建创新的AI实验和生产平台，加速生成式AI的产品开发。

这个示例演示了如何使用LangChain与`ChatFireworks`模型进行交互。

%pip install langchain-fireworks

```python
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_fireworks import ChatFireworks
```

# 设置

1. 确保`langchain-fireworks`包已安装在您的环境中。

2. 登录[Fireworks AI](http://fireworks.ai)获取API密钥以访问我们的模型，并确保将其设置为`FIREWORKS_API_KEY`环境变量。

3. 使用模型ID设置您的模型。如果未设置模型，则默认模型为fireworks-llama-v2-7b-chat。在[app.fireworks.ai](https://app.fireworks.ai)上查看完整、最新的模型列表。

```python
import getpass
import os
if "FIREWORKS_API_KEY" not in os.environ:
    os.environ["FIREWORKS_API_KEY"] = getpass.getpass("Fireworks API Key:")
# 初始化一个烟火聊天模型
chat = ChatFireworks(model="accounts/fireworks/models/mixtral-8x7b-instruct")
```

# 直接调用模型

您可以直接调用模型，输入系统消息和用户消息以获取答案。

```python
# 聊天烟火包装器
system_message = SystemMessage(content="You are to chat with the user.")
human_message = HumanMessage(content="Who are you?")
chat.invoke([system_message, human_message])
```

```output
AIMessage(content="Hello! I'm an AI language model, a helpful assistant designed to chat and assist you with any questions or information you might need. I'm here to make your experience as smooth and enjoyable as possible. How can I assist you today?")
```

```python
# 设置额外参数：温度、最大标记、top_p
chat = ChatFireworks(
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    temperature=1,
    max_tokens=20,
)
system_message = SystemMessage(content="You are to chat with the user.")
human_message = HumanMessage(content="How's the weather today?")
chat.invoke([system_message, human_message])
```

```output
AIMessage(content="I'm an AI and do not have the ability to experience the weather firsthand. However,")
```

# 工具调用

烟火提供了[`FireFunction-v1`工具调用模型](https://fireworks.ai/blog/firefunction-v1-gpt-4-level-function-calling)。您可以将其用于结构化输出和函数调用用例：

```python
from pprint import pprint
from langchain_core.pydantic_v1 import BaseModel
class ExtractFields(BaseModel):
    name: str
    age: int
chat = ChatFireworks(
    model="accounts/fireworks/models/firefunction-v1",
).bind_tools([ExtractFields])
result = chat.invoke("I am a 27 year old named Erick")
pprint(result.additional_kwargs["tool_calls"][0])
```

```output
{'function': {'arguments': '{"name": "Erick", "age": 27}',
              'name': 'ExtractFields'},
 'id': 'call_J0WYP2TLenaFw3UeVU0UnWqx',
 'index': 0,
 'type': 'function'}
```