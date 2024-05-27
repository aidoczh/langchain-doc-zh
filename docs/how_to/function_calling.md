

---

sidebar_position: 2

---

# 如何进行工具/函数调用

:::info

我们将术语“工具调用”与“函数调用”互换使用。尽管“函数调用”有时指的是对单个函数的调用，但我们将所有模型都视为能够在每条消息中返回多个工具或函数调用。

:::

工具调用允许模型通过生成符合用户定义模式的输出来响应给定提示。虽然名称暗示模型正在执行某些操作，但实际情况并非如此！模型正在为工具提供参数，并且实际运行工具（或不运行）取决于用户 - 例如，如果您想要从非结构化文本中提取与某个模式匹配的输出，您可以为模型提供一个“提取”工具，该工具接受与所需模式匹配的参数，然后将生成的输出视为最终结果。

工具调用包括名称、参数字典和可选标识符。参数字典结构为`{参数名称: 参数值}`。

许多LLM提供商，包括[Anthropic](https://www.anthropic.com/)、[Cohere](https://cohere.com/)、[Google](https://cloud.google.com/vertex-ai)、[Mistral](https://mistral.ai/)、[OpenAI](https://openai.com/)等，支持工具调用功能的各种变体。这些功能通常允许向LLM发送请求时包含可用工具及其模式，并且响应中包含对这些工具的调用。例如，给定一个搜索引擎工具，LLM可能通过首先发出对搜索引擎的调用来处理查询。调用LLM的系统可以接收工具调用，执行它，并将输出返回给LLM以指导其响应。LangChain包括一套[内置工具](/docs/integrations/tools/)，并支持几种定义您自己的[自定义工具](/docs/how_to/custom_tools)的方法。工具调用对于构建[使用工具的链和代理](/docs/how_to#tools)以及从模型中获取结构化输出非常有用。

不同提供商采用不同的格式约定来格式化工具模式和工具调用。例如，Anthropic将工具调用作为解析结构返回到较大内容块中：

```python
[
  {
    "text": "<thinking>\nI should use a tool.\n</thinking>",
    "type": "text"
  },
  {
    "id": "id_value",
    "input": {"arg_name": "arg_value"},
    "name": "tool_name",
    "type": "tool_use"
  }
]
```

而OpenAI将工具调用分离为一个独立参数，参数为JSON字符串：

```python
{
  "tool_calls": [
    {
      "id": "id_value",
      "function": {
        "arguments": '{"arg_name": "arg_value"}',
        "name": "tool_name"
      },
      "type": "function"
    }
  ]
}
```

LangChain实现了用于定义工具、将其传递给LLM以及表示工具调用的标准接口。

## 将工具传递给LLM

支持工具调用功能的聊天模型实现了`.bind_tools`方法，该方法接收一个LangChain的[工具对象列表](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.BaseTool.html#langchain_core.tools.BaseTool)，并将它们绑定到聊天模型中的预期格式。对聊天模型的后续调用将在其对LLM的调用中包含工具模式。

例如，我们可以使用Python函数上的`@tool`装饰器定义自定义工具的模式：

```python
from langchain_core.tools import tool
@tool
def add(a: int, b: int) -> int:
    """将a和b相加。"""
    return a + b
@tool
def multiply(a: int, b: int) -> int:
    """将a和b相乘。"""
    return a * b
tools = [add, multiply]
```

或者，我们可以使用Pydantic来定义模式：

```python
from langchain_core.pydantic_v1 import BaseModel, Field
# 请注意，这里的文档字符串至关重要，因为它们将与类名一起传递给模型。
class Add(BaseModel):
    """将两个整数相加。"""
    a: int = Field(..., description="第一个整数")
    b: int = Field(..., description="第二个整数")
class Multiply(BaseModel):
    """将两个整数相乘。"""
    a: int = Field(..., description="第一个整数")
    b: int = Field(..., description="第二个整数")
tools = [Add, Multiply]
```

我们可以将它们绑定到聊天模型中：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="llm"
  fireworksParams={`model="accounts/fireworks/models/firefunction-v1", temperature=0`}
/>

我们可以使用`bind_tools()`方法来处理将`Multiply`转换为“工具”并将其绑定到模型（即在每次调用模型时传递它）。

```python
llm_with_tools = llm.bind_tools(tools)
```

## 工具调用

如果工具调用包含在LLM的响应中，它们将附加到相应的

[消息](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html#langchain_core.messages.ai.AIMessage) 或 [消息块](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessageChunk.html#langchain_core.messages.ai.AIMessageChunk) 

作为 `.tool_calls` 属性中的 [工具调用](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.ToolCall.html#langchain_core.messages.tool.ToolCall) 

对象列表。`ToolCall` 是一个类型化字典，包括工具名称、参数值字典和（可选的）标识符。没有工具调用的消息默认将此属性设为空列表。

示例：

```python
query = "3 * 12 是多少？另外，11 + 49 是多少？"
llm_with_tools.invoke(query).tool_calls
```

```output
[{'name': 'Multiply',
  'args': {'a': 3, 'b': 12},
  'id': 'call_1Tdp5wUXbYQzpkBoagGXqUTo'},
 {'name': 'Add',
  'args': {'a': 11, 'b': 49},
  'id': 'call_k9v09vYioS3X0Qg35zESuUKI'}]
```

`.tool_calls` 属性应包含有效的工具调用。请注意，偶尔模型提供者可能会输出格式不正确的工具调用（例如，不是有效 JSON 的参数）。在这些情况下，解析失败时，会在 `.invalid_tool_calls` 属性中填充 [InvalidToolCall](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.InvalidToolCall.html#langchain_core.messages.tool.InvalidToolCall) 的实例。`InvalidToolCall` 可以包含名称、字符串参数、标识符和错误消息。

如果需要，[输出解析器](/docs/how_to#output-parsers) 可以进一步处理输出。例如，我们可以转换回原始的 Pydantic 类：

```python
from langchain_core.output_parsers.openai_tools import PydanticToolsParser
chain = llm_with_tools | PydanticToolsParser(tools=[Multiply, Add])
chain.invoke(query)
```

```output
[Multiply(a=3, b=12), Add(a=11, b=49)]
```

### 流式处理

当在流式处理上下文中调用工具时， 

[消息块](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessageChunk.html#langchain_core.messages.ai.AIMessageChunk) 

将通过 `.tool_call_chunks` 属性以列表形式填充 [工具调用块](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.ToolCallChunk.html#langchain_core.messages.tool.ToolCallChunk) 

对象。`ToolCallChunk` 包括工具 `名称`、`参数` 和 `id` 的可选字符串字段，并包括一个可选的整数字段 `index`，可用于连接块。字段是可选的，因为工具调用的部分可能跨越不同的块进行流式处理（例如，包含参数子字符串的块可能对于工具名称和 id 具有空值）。

由于消息块继承自其父消息类，带有工具调用块的 

[AIMessageChunk](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessageChunk.html#langchain_core.messages.ai.AIMessageChunk) 

还将包括 `.tool_calls` 和 `.invalid_tool_calls` 字段。这些字段是从消息的工具调用块中尽力解析而来的。

请注意，并非所有提供者目前都支持工具调用的流式处理。

示例：

```python
async for chunk in llm_with_tools.astream(query):
    print(chunk.tool_call_chunks)
```

```output
[]
[{'name': 'Multiply', 'args': '', 'id': 'call_d39MsxKM5cmeGJOoYKd
"Whats 119 times 8 minus 20. Don't do any math yourself, only use tools for math. Respect order of operations"
),
    AIMessage(
        content='119 * 8 is 952',
        tool_calls=[{'name': 'Multiply', 'args': {'a': 119, 'b': 8}, 'id': 'call_Dl3FXRVkQCFW4sUNYOe4rFr7'}]
    ),
    AIMessage(
        content='952 - 20 is 932',
        tool_calls=[{'name': 'Add', 'args': {'a': 952, 'b': -20}, 'id': 'call_n03l4hmka7VZTCiP387Wud2C'}]
    )
]
```

```python
prompt = ChatPromptTemplate(
    messages=examples,
    runnable=RunnablePassthrough(),
    model=llm_with_tools
)
prompt.invoke("Whats 119 times 8 minus 20. Don't do any math yourself, only use tools for math. Respect order of operations")
```

```
"317253乘以128472再加四的结果是多少"，姓名="example_user"
    ),
```

```python
    AIMessage(
        "",
        name="example_assistant",
        tool_calls=[
            {"name": "Multiply", "args": {"x": 317253, "y": 128472}, "id": "1"}
        ],
    ),
    ToolMessage("16505054784", tool_call_id="1"),
    AIMessage(
        "",
        name="example_assistant",
        tool_calls=[{"name": "Add", "args": {"x": 16505054784, "y": 4}, "id": "2"}],
    ),
    ToolMessage("16505054788", tool_call_id="2"),
    AIMessage(
        "317253乘以128472再加四的结果是16505054788",
        name="example_assistant",
    ),
]
system = """你不擅长数学，但是擅长使用计算器。
使用过去的工具使用作为正确使用工具的示例。"""
few_shot_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        *examples,
        ("human", "{query}"),
    ]
)
chain = {"query": RunnablePassthrough()} | few_shot_prompt | llm_with_tools
chain.invoke("119乘以8减去20").tool_calls
```

```output
[{'name': 'Multiply',
  'args': {'a': 119, 'b': 8},
  'id': 'call_MoSgwzIhPxhclfygkYaKIsGZ'}]
```

这次似乎得到了正确的输出。

这是[LangSmith跟踪](https://smith.langchain.com/public/f70550a1-585f-4c9d-a643-13148ab1616f/r)的样子。

## 下一步

- **输出解析**：查看[OpenAI工具输出解析器](/docs/how_to/output_parser_structured)

    以了解如何将调用API的函数响应提取为各种格式。

- **结构化输出链**：[一些模型具有构造函数](/docs/how_to/structured_output)，

    可以帮助您创建结构化输出链。

- **工具使用**：查看如何构建调用

    工具的链和代理的[这些

    指南](/docs/how_to#tools)。