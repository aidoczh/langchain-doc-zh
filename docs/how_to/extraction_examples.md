# 如何在提取过程中使用参考示例

通过提供参考示例给语言模型可以提高提取的质量。

数据提取旨在生成文本和其他非结构化或半结构化格式中的信息的结构化表示。在这种情况下，通常会使用[工具调用](/docs/concepts#functiontool-calling)语言模型的功能。本指南演示了如何构建少样本工具调用示例来帮助引导提取和类似应用的行为。

:::tip

虽然本指南侧重于如何在工具调用模型中使用示例，但这种技术通常适用，并且也适用于JSON等更多基于提示的技术。

:::

LangChain在从包含工具调用的语言模型消息中实现了[tool-call属性](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html#langchain_core.messages.ai.AIMessage.tool_calls)。有关更多详细信息，请参阅我们的[工具调用指南](/docs/how_to/tool_calling)。为了构建数据提取的参考示例，我们构建了一个包含以下序列的聊天历史记录：

- 包含示例输入的[HumanMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.human.HumanMessage.html)

- 包含示例工具调用的[AIMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html)

- 包含示例工具输出的[ToolMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.ToolMessage.html)

LangChain采用了这种将工具调用结构化为LLM模型提供者之间对话的约定。

首先，我们构建一个包含这些消息占位符的提示模板：

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
# 定义一个自定义提示，提供说明和任何其他上下文。
# 1) 您可以将示例添加到提示模板中以提高提取质量
# 2) 引入其他参数以考虑上下文（例如，包括从中提取文本的文档的元数据）
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are an expert extraction algorithm. "
            "Only extract relevant information from the text. "
            "If you do not know the value of an attribute asked "
            "to extract, return null for the attribute's value.",
        ),
        # ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        MessagesPlaceholder("examples"),  # <-- EXAMPLES!
        # ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        ("human", "{text}"),
    ]
)
```

测试一下这个模板：

```python
from langchain_core.messages import (
    HumanMessage,
)
prompt.invoke(
    {"text": "this is some text", "examples": [HumanMessage(content="testing 1 2 3")]}
)
```

```output
ChatPromptValue(messages=[SystemMessage(content="You are an expert extraction algorithm. Only extract relevant information from the text. If you do not know the value of an attribute asked to extract, return null for the attribute's value."), HumanMessage(content='testing 1 2 3'), HumanMessage(content='this is some text')])
```

## 定义模式

让我们重用[提取教程](/docs/tutorials/extraction)中的人物模式。

```python
from typing import List, Optional
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI
class Person(BaseModel):
    """关于一个人的信息。"""
    # ^ Person实体的文档字符串。
    # 这个文档字符串作为Person模式的描述发送给LLM，它可以帮助提高提取结果。
    # 注意：
    # 1. 每个字段都是`optional`的，这允许模型拒绝提取它！
    # 2. 每个字段都有一个`description`，这个描述被LLM使用。
    # 有一个好的描述可以帮助提高提取结果。
    name: Optional[str] = Field(..., description="人的姓名")
    hair_color: Optional[str] = Field(
        ..., description="如果已知，人的眼睛颜色"
    )
    height_in_meters: Optional[str] = Field(..., description="以米为单位的身高")
class Data(BaseModel):
    """关于人的提取数据。"""
    # 创建一个模型，以便我们可以提取多个实体。
    people: List[Person]
```

## 定义参考示例

示例可以定义为输入-输出对的列表。

每个示例包含一个示例`input`文本和一个示例`output`，显示应从文本中提取的内容。

:::important

这有点复杂，所以可以跳过。

示例的格式需要与使用的API匹配（例如，工具调用或JSON模式等）。

在这里，格式化的示例将与工具调用API期望的格式匹配，因为我们正在使用它。

:::

```python
import uuid
from typing import Dict, List, TypedDict
from langchain_core.messages import (
    AIMessage,
    BaseMessage,
    HumanMessage,
    SystemMessage,
    ToolMessage,
)
from langchain_core.pydantic_v1 import BaseModel, Field
class Example(TypedDict):
    """一个由文本输入和预期工具调用组成的示例表示。
    对于提取，工具调用被表示为 pydantic 模型的实例。
    """
    input: str  # 这是示例文本
    tool_calls: List[BaseModel]  # 应该被提取的 pydantic 模型实例
def tool_example_to_messages(example: Example) -> List[BaseMessage]:
    """将示例转换为可以输入到 LLM 中的消息列表。
    这段代码是一个适配器，将我们的示例转换为可以输入到聊天模型中的消息列表。
    每个示例的消息列表对应于：
    1) HumanMessage: 包含应从中提取内容的内容。
    2) AIMessage: 包含模型中提取的信息
    3) ToolMessage: 包含对模型的确认，模型正确请求了一个工具。
    需要 ToolMessage 是因为一些聊天模型是针对代理而不是提取用例进行了超优化。
    """
    messages: List[BaseMessage] = [HumanMessage(content=example["input"])]
    tool_calls = []
    for tool_call in example["tool_calls"]:
        tool_calls.append(
            {
                "id": str(uuid.uuid4()),
                "args": tool_call.dict(),
                # 函数名目前对应于 pydantic 模型的名称
                # 这在 API 中是隐含的，
                # 会随着时间的推移而改进。
                "name": tool_call.__class__.__name__,
            },
        )
    messages.append(AIMessage(content="", tool_calls=tool_calls))
    tool_outputs = example.get("tool_outputs") or [
        "You have correctly called this tool."
    ] * len(tool_calls)
    for output, tool_call in zip(tool_outputs, tool_calls):
        messages.append(ToolMessage(content=output, tool_call_id=tool_call["id"]))
    return messages
examples = [
    (
        "The ocean is vast and blue. It's more than 20,000 feet deep. There are many fish in it.",
        Person(name=None, height_in_meters=None, hair_color=None),
    ),
    (
        "Fiona traveled far from France to Spain.",
        Person(name="Fiona", height_in_meters=None, hair_color=None),
    ),
]
messages = []
for text, tool_call in examples:
    messages.extend(
        tool_example_to_messages({"input": text, "tool_calls": [tool_call]})
    )
```

让我们测试一下提示

```python
example_prompt = prompt.invoke({"text": "this is some text", "examples": messages})
for message in example_prompt.messages:
    print(f"{message.type}: {message}")
```

```output
system: content="You are an expert extraction algorithm. Only extract relevant information from the text. If you do not know the value of an attribute asked to extract, return null for the attribute's value."
human: content="The ocean is vast and blue. It's more than 20,000 feet deep. There are many fish in it."
ai: content='' tool_calls=[{'name': 'Person', 'args': {'name': None, 'hair_color': None, 'height_in_meters': None}, 'id': 'b843ba77-4c9c-48ef-92a4-54e534f24521'}]
tool: content='You have correctly called this tool.' tool_call_id='b843ba77-4c9c-48ef-92a4-54e534f24521'
human: content='Fiona traveled far from France to Spain.'
ai: content='' tool_calls=[{'name': 'Person', 'args': {'name': 'Fiona', 'hair_color': None, 'height_in_meters': None}, 'id': '46f00d6b-50e5-4482-9406-b07bb10340f6'}]
tool: content='You have correctly called this tool.' tool_call_id='46f00d6b-50e5-4482-9406-b07bb10340f6'
human: content='this is some text'
```

## 创建一个提取器

让我们选择一个 LLM。因为我们正在使用工具调用，所以我们需要一个支持工具调用功能的模型。查看[此表](/docs/integrations/chat)以获取可用的 LLM。

import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs
  customVarName="llm"
  openaiParams={`model="gpt-4-0125-preview", temperature=0`}
/>

根据所需的模式，使用 `.with_structured_output` 方法对模型输出进行结构化，参考[提取教程](/docs/tutorials/extraction)：

```python
runnable = prompt | llm.with_structured_output(
    schema=Data,
    method="function_calling",
    include_raw=False,
)
```

## 没有示例 😿

请注意，即使是功能强大的模型也可能在**非常简单**的测试用例中失败！

```python
for _ in range(5):
    text = "The solar system is large, but earth has only 1 moon."
    print(runnable.invoke({"text": text, "examples": []}))
```

## 带有示例的情况 😻

参考示例有助于修复失败的情况！

```python
for _ in range(5):
    text = "太阳系很大，但地球只有一个月亮。"
    print(runnable.invoke({"text": text, "examples": messages}))
```

```output
people=[]
people=[]
people=[]
people=[]
people=[]
```

请注意，我们可以将few-shot示例视为[Langsmith跟踪](https://smith.langchain.com/public/4c436bc2-a1ce-440b-82f5-093947542e40/r)中的工具调用。

我们在正样本上保持了性能：

```python
runnable.invoke(
    {
        "text": "我的名字是Harrison。我的头发是黑色的。",
        "examples": messages,
    }
)
```

```output
Data(people=[Person(name='Harrison', hair_color='black', height_in_meters=None)])
```