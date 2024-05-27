

# 如何使用模型调用工具

:::info 先决条件

本指南假定您熟悉以下概念：

- [聊天模型](/docs/concepts/#chat-models)

- [LangChain 工具](/docs/concepts/#tools)

:::

:::info

我们将术语“工具调用”与“函数调用”互换使用。尽管

有时“函数调用”指的是对单个函数的调用，

但我们将所有模型视为可以在

每条消息中返回多个工具或函数调用。

:::

工具调用允许聊天模型通过“调用工具”来响应给定提示。

虽然名称暗示模型正在执行

某些操作，但实际情况并非如此！模型生成

工具的参数，实际运行工具（或不运行）取决于用户。

例如，如果您想要从非结构化文本中提取与某个模式匹配的输出，

您可以给模型一个“提取”工具，该工具接受

与所需模式匹配的参数，然后将生成的输出视为最终结果。

然而，工具调用不仅限于[结构化输出](/docs/how_to/structured_output/)

因为您可以将调用工具的响应传递回模型，以创建更长的交互。

例如，给定一个搜索引擎工具，LLM 可能通过

首先使用带有参数的搜索引擎调用来处理查询。调用 LLM 的系统可以

接收工具调用，执行它，并将输出返回给 LLM 以通知其

响应。LangChain 包括一套[内置工具](/docs/integrations/tools/)

并支持几种方法来定义您自己的[自定义工具](/docs/how_to/custom_tools)。

工具调用并非普遍存在，但许多流行的 LLM 提供商，包括[Anthropic](https://www.anthropic.com/),

[Cohere](https://cohere.com/), [Google](https://cloud.google.com/vertex-ai),

[Mistral](https://mistral.ai/), [OpenAI](https://openai.com/) 等，

支持工具调用功能的变体。

LangChain 实现了用于定义工具、将其传递给 LLM

以及表示工具调用的标准接口。本指南将向您展示如何使用它们。

## 将工具传递给聊天模型

支持工具调用功能的聊天模型实现了`.bind_tools`方法，该方法

接收 LangChain 的工具对象列表[tool objects](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.BaseTool.html#langchain_core.tools.BaseTool)

并以其预期格式将它们绑定到聊天模型。对聊天模型的后续调用将在其对 LLM 的调用中包含工具模式。

例如，我们可以使用 Python 函数上的`@tool`装饰器定义自定义工具的模式：

```python
from langchain_core.tools import tool
@tool
def add(a: int, b: int) -> int:
    """将 a 和 b 相加。"""
    return a + b
@tool
def multiply(a: int, b: int) -> int:
    """将 a 和 b 相乘。"""
    return a * b
tools = [add, multiply]
```

或者，我们可以使用[Pydantic](https://docs.pydantic.dev)来定义模式：

```python
from langchain_core.pydantic_v1 import BaseModel, Field
# 请注意，这里的文档字符串至关重要，因为它们将与
# 类名一起传递给模型。
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

我们可以将它们绑定到聊天模型如下：

import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs
  customVarName="llm"
  fireworksParams={`model="accounts/fireworks/models/firefunction-v1", temperature=0`}
/>

我们将使用`.bind_tools()`方法来处理将`Multiply`转换为模型的正确格式，然后绑定它（即

每次调用模型时传递它）。

```python
llm_with_tools = llm.bind_tools(tools)
```

## 工具调用

如果工具调用包含在 LLM 响应中，则它们将附加到相应的

[消息](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html#langchain_core.messages.ai.AIMessage)

或[消息块](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessageChunk.html#langchain_core.messages.ai.AIMessageChunk)

作为[tool call](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.ToolCall.html#langchain_core.messages.tool.ToolCall)对象列表，存储在`.tool_calls`属性中。

请注意，聊天模型可以同时调用多个工具。

`ToolCall`是一个类型化字典，包括

工具名称、参数值字典和（可选）标识符。没有

工具调用的消息默认将此属性设置为空列表。

```python
query = "3 * 12 是多少？另外，11 + 49 是多少？"
llm_with_tools.invoke(query).tool_calls```
```
```output

[{'name': 'Multiply',

  'args': {'a': 3, 'b': 12},

  'id': 'call_KquHA7mSbgtAkpkmRPaFnJKa'},

 {'name': 'Add',

  'args': {'a': 11, 'b': 49},

  'id': 'call_Fl0hQi4IBTzlpaJYlM5kPQhE'}]

```
`.tool_calls` 属性应包含有效的工具调用。请注意，偶尔，模型提供者可能会输出格式不正确的工具调用（例如，不是有效的 JSON 的参数）。在这些情况下，当解析失败时，[InvalidToolCall](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.InvalidToolCall.html#langchain_core.messages.tool.InvalidToolCall) 的实例会出现在 `.invalid_tool_calls` 属性中。`InvalidToolCall` 可以有名称、字符串参数、标识符和错误消息。
如果需要的话，[输出解析器](/docs/how_to#output-parsers) 可以进一步处理输出。例如，我们可以将其转换回原始的 Pydantic 类：
```python
from langchain_core.output_parsers.openai_tools import PydanticToolsParser
chain = llm_with_tools | PydanticToolsParser(tools=[Multiply, Add])
chain.invoke(query)
```
```

[Multiply(a=3, b=12), Add(a=11, b=49)]

```
### 流式处理
当在流式处理上下文中调用工具时，[消息块](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessageChunk.html#langchain_core.messages.ai.AIMessageChunk) 将通过 `.tool_call_chunks` 属性中的工具调用块对象的列表进行填充。`ToolCallChunk` 包括工具的可选字符串字段 `name`、`args` 和 `id`，并包括一个可选的整数字段 `index`，可用于将块连接在一起。字段是可选的，因为工具调用的部分可能会在不同的块中进行流式处理（例如，包含参数子字符串的块可能对于工具名称和标识符具有空值）。
因为消息块从其父消息类继承，具有工具调用块的 [AIMessageChunk](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessageChunk.html#langchain_core.messages.ai.AIMessageChunk) 也将包括 `.tool_calls` 和 `.invalid_tool_calls` 字段。这些字段是从消息的工具调用块中尽力解析出来的。
请注意，目前并非所有提供者都支持工具调用的流式处理：
```python
async for chunk in llm_with_tools.astream(query):
    print(chunk.tool_call_chunks)
```
```

[]

[{'name': 'Multiply', 'args': '', 'id': 'call_3aQwTP9CYlFxwOvQZPHDu6wL', 'index': 0}]

[{'name': None, 'args': '{"a"', 'id': None, 'index': 0}]

[{'name': None, 'args': ': 3, ', 'id': None, 'index': 0}]

[{'name': None, 'args': '"b": 1', 'id': None, 'index': 0}]

[{'name': None, 'args': '2}', 'id': None, 'index': 0}]

[{'name': 'Add', 'args': '', 'id': 'call_SQUoSsJz2p9Kx2x73GOgN1ja', 'index': 1}]

[{'name': None, 'args': '{"a"', 'id': None, 'index': 1}]

[{'name': None, 'args': ': 11,', 'id': None, 'index': 1}]

[{'name': None, 'args': ' "b": ', 'id': None, 'index': 1}]

[{'name': None, 'args': '49}', 'id': None, 'index': 1}]

[]

```
请注意，添加消息块将合并它们对应的工具调用块。这是 LangChain 的各种[工具输出解析器](/docs/how_to/output_parser_structured)支持流式处理的原理。
例如，下面我们累积工具调用块：
```python
first = True
async for chunk in llm_with_tools.astream(query):
    if first:
        gathered = chunk
        first = False
    else:
        gathered = gathered + chunk
    print(gathered.tool_call_chunks)
```
```

[]

[{'name': 'Multiply', 'args': '', 'id': 'call_AkL3dVeCjjiqvjv8ckLxL3gP', 'index': 0}]

[{'name': 'Multiply', 'args': '{"a"', 'id': 'call_AkL3dVeCjjiqvjv8ckLxL3gP', 'index': 0}]

[{'name': 'Multiply', 'args': '{"a": 3, ', 'id': 'call_AkL3dVeCjjiqvjv8ckLxL3gP', 'index': 0}]

[{'name': 'Multiply', 'args': '{"a": 3, "b": 1', 'id': 'call_AkL3dVeCjjiqvjv8ckLxL3gP', 'index': 0}]

[{'name': 'Multiply', 'args': '{"a": 3, "b": 12}', 'id': 'call_AkL3dVeCjjiqvjv8ckLxL3gP', 'index': 0}]

[{'name': 'Multiply', 'args': '{"a": 3, "b": 12}', 'id': 'call_AkL3dVeCjjiqvjv8ckLxL3gP', 'index': 0}, {'name': 'Add', 'args': '', 'id': 'call_b4iMiB3chGNGqbt5SjqqD2Wh', 'index': 1}]

[{'name': 'Multiply', 'args': '{"a": 3, "b": 12}', 'id': 'call_AkL3dVeCjjiqvjv8ckLxL3gP', 'index': 0}, {'name': 'Add', 'args': '{"a"', 'id': 'call_b4iMiB3chGNGqbt5SjqqD2Wh', 'index': 1}]

[{'name': 'Multiply', 'args': '{"a": 3, "b": 12}', 'id': 'call_AkL3dVeCjjiqvjv8ckLxL3gP', 'index': 0}, {'name': 'Add', 'args': '{"a": 11,', 'id': 'call_b4iMiB3chGNGqbt5SjqqD2Wh', 'index': 1}]

```
```python
print(type(gathered.tool_calls[0]["args"]))
```
```output

<class 'dict'>

```
## 工具调用累积示例
以下是累积工具调用的示例，以展示部分解析过程：
```python
print(type(gathered.tool_call_chunks[0]["args"]))
```
```output

<class 'str'>

```
```python
first = True
async for chunk in llm_with_tools.astream(query):
    if first:
        gathered = chunk
        first = False
    else:
        gathered = gathered + chunk
    print(gathered.tool_calls)
```
```output

[]

[]

[{'name': 'Multiply', 'args': {}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}]

[{'name': 'Multiply', 'args': {'a': 3}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}]

[{'name': 'Multiply', 'args': {'a': 3, 'b': 1}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}]

[{'name': 'Multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}]

[{'name': 'Multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}]

[{'name': 'Multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}, {'name': 'Add', 'args': {}, 'id': 'call_54Hx3DGjZitFlEjgMe1DYonh'}]

[{'name': 'Multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}, {'name': 'Add', 'args': {'a': 11}, 'id': 'call_54Hx3DGjZitFlEjgMe1DYonh'}]

[{'name': 'Multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}, {'name': 'Add', 'args': {'a': 11}, 'id': 'call_54Hx3DGjZitFlEjgMe1DYonh'}]

[{'name': 'Multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}, {'name': 'Add', 'args': {'a': 11, 'b': 49}, 'id': 'call_54Hx3DGjZitFlEjgMe1DYonh'}]

[{'name': 'Multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_4p0D4tHVXSiae9Mu0e8jlI1m'}, {'name': 'Add', 'args': {'a': 11, 'b': 49}, 'id': 'call_54Hx3DGjZitFlEjgMe1DYonh'}]

```
```python
print(type(gathered.tool_calls[0]["args"]))
```
```output

<class 'dict'>

```
## 将工具输出传递给模型
如果我们使用模型生成的工具调用来实际调用工具，并希望将工具结果传递回模型，我们可以使用 `ToolMessage` 进行操作。
```python
from langchain_core.messages import HumanMessage, ToolMessage
messages = [HumanMessage(query)]
ai_msg = llm_with_tools.invoke(messages)
messages.append(ai_msg)
for tool_call in ai_msg.tool_calls:
    selected_tool = {"add": add, "multiply": multiply}[tool_call["name"].lower()]
    tool_output = selected_tool.invoke(tool_call["args"])
    messages.append(ToolMessage(tool_output, tool_call_id=tool_call["id"]))
messages
```
```output

[HumanMessage(content='What is 3 * 12? Also, what is 11 + 49?'),

 AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_svc2GLSxNFALbaCAbSjMI9J8', 'function': {'arguments': '{"a": 3, "b": 12}', 'name': 'Multiply'}, 'type': 'function'}, {'id': 'call_r8jxte3zW6h3MEGV3zH2qzFh', 'function': {'arguments': '{"a": 11, "b": 49}', 'name': 'Add'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 50, 'prompt_tokens': 105, 'total_tokens': 155}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_d9767fc5b9', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-a79ad1dd-95f1-4a46-b688-4c83f327a7b3-0', tool_calls=[{'name': 'Multiply', 'args': {'a': 3, 'b': 12}, 'id': 'call_svc2GLSxNFALbaCAbSjMI9J8'}, {'name': 'Add', 'args': {'a': 11, 'b': 49}, 'id': 'call_r8jxte3zW6h3MEGV3zH2qzFh'}]),

 ToolMessage(content='36', tool_call_id='call_svc2GLSxNFALbaCAbSjMI9J8'),

 ToolMessage(content='60', tool_call_id='call_r8jxte3zW6h3MEGV3zH2qzFh')]

```
```python
llm_with_tools.invoke(messages)
```
```output

AIMessage(content='3 * 12 is 36 and 11 + 49 is 60.', response_metadata={'token_usage': {'completion_tokens': 18, 'prompt_tokens': 171, 'total_tokens': 189}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_d9767fc5b9', 'finish_reason': 'stop', 'logprobs': None}, id='run-20b52149-e00d-48ea-97cf-f8de7a255f8c-0')

```
请注意，在 `ToolMessage` 中我们传回与模型接收的相同的 `id`，以帮助模型将工具响应与工具调用匹配。
## 少量示例提示
对于更复杂的工具使用，通过在提示中添加少量示例非常有用。我们可以通过在提示中添加带有 `ToolCall` 和相应 `ToolMessage` 的 `AIMessage` 来实现这一点。
例如，即使有一些特殊说明，我们的模型也可能因操作顺序而感到困惑：
```python
llm_with_tools.invoke(
    "Whats 119 times 8 minus 20. Don't do any math yourself, only use tools for math. Respect order of operations"
).tool_calls
```
```output

[{'name': 'Multiply',

  'args': {'a': 119, 'b': 8},

  'id': 'call_T88XN6ECucTgbXXkyDeC2CQj'},

 {'name': 'Add',

  'args': {'a': 952, 'b': -20},

  'id': 'call_licdlmGsRqzup8rhqJSb1yZ4'}]

```
模型不应该尝试添加任何东西，因为它在技术上还不知道 119 * 8 的结果。
通过添加一些示例提示，我们可以纠正这种行为：
```python
from langchain_core.messages import AIMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
examples = [
    HumanMessage(
        "What's the product of 317253 and 128472 plus four", name="example_user"
    ),
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
        "The product of 317253 and 128472 plus four is 16505054788",
        name="example_assistant",
    ),
]
system = """You are bad at math but are an expert at using a calculator. 
Use past tool usage as an example of how to correctly use the tools."""
few_shot_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        *examples,
        ("human", "{query}"),
    ]
)
chain = {"query": RunnablePassthrough()} | few_shot_prompt | llm_with_tools
chain.invoke("Whats 119 times 8 minus 20").tool_calls
```
```output

[{'name': 'Multiply',

  'args': {'a': 119, 'b': 8},

  'id': 'call_9MvuwQqg7dlJupJcoTWiEsDo'}]

```
这次我们得到了正确的输出。
这是 [LangSmith trace](https://smith.langchain.com/public/f70550a1-585f-4c9d-a643-13148ab1616f/r) 的样子。
## 绑定模型特定格式（高级）
提供者采用不同的约定来格式化工具模式。
例如，OpenAI 使用的格式如下：
- `type`：工具的类型。在撰写本文时，这总是 `"function"`。
- `function`：包含工具参数的对象。
- `function.name`：要输出的模式的名称。
- `function.description`：要输出的模式的高级描述。
- `function.parameters`：您想要提取的模式的嵌套细节，格式为 [JSON schema](https://json-schema.org/) 字典。
如果需要，我们也可以直接将这种模型特定格式绑定到模型上。以下是一个示例：
```python
from langchain_openai import ChatOpenAI
model = ChatOpenAI()
model_with_tools = model.bind(
    tools=[
        {
            "type": "function",
            "function": {
                "name": "multiply",
                "description": "Multiply two integers together.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "a": {"type": "number", "description": "First integer"},
                        "b": {"type": "number", "description": "Second integer"},
                    },
                    "required": ["a", "b"],
                },
            },
        }
    ]
)
model_with_tools.invoke("Whats 119 times 8?")
```
```output

AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_mn4ELw1NbuE0DFYhIeK0GrPe', 'function': {'arguments': '{"a":119,"b":8}', 'name': 'multiply'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 17, 'prompt_tokens': 62, 'total_tokens': 79}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-353e8a9a-7125-4f94-8c68-4f3da4c21120-0', tool_calls=[{'name': 'multiply', 'args': {'a': 119, 'b': 8}, 'id': 'call_mn4ELw1NbuE0DFYhIeK0GrPe'}])

```

这在功能上等同于上面的 `bind_tools()` 调用。

## 下一步

现在您已经学会了如何将工具模式绑定到聊天模型，并调用这些工具。接下来，查看一些更具体的工具调用用法：

- 构建[使用工具的链和代理](/docs/how_to#tools)

- 从模型获取[结构化输出](/docs/how_to/structured_output/)