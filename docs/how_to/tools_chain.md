---

sidebar_position: 0

---

# 如何使用链式工具

在本指南中，我们将介绍创建链式工具和代理调用工具的基本方法。工具可以是任何东西 - API、函数、数据库等。工具使我们能够扩展模型的能力，不仅仅是输出文本/消息。使用工具与模型的关键在于正确提示模型并解析其响应，以便选择正确的工具并为其提供正确的输入。

## 设置

我们需要安装以下软件包才能完成本指南：

```python
%pip install --upgrade --quiet langchain
```

如果您想在 [LangSmith](https://docs.smith.langchain.com/) 中跟踪运行情况，请取消注释并设置以下环境变量：

```python
import getpass
import os
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 创建工具

首先，我们需要创建一个要调用的工具。在本示例中，我们将从一个函数创建一个自定义工具。有关创建自定义工具的更多信息，请参阅[此指南](/docs/how_to/custom_tools)。

```python
from langchain_core.tools import tool
@tool
def multiply(first_int: int, second_int: int) -> int:
    """Multiply two integers together."""
    return first_int * second_int
```

```python
print(multiply.name)
print(multiply.description)
print(multiply.args)
```

```output
multiply
multiply(first_int: int, second_int: int) -> int - Multiply two integers together.
{'first_int': {'title': 'First Int', 'type': 'integer'}, 'second_int': {'title': 'Second Int', 'type': 'integer'}}
```

```python
multiply.invoke({"first_int": 4, "second_int": 5})
```

```output
20
```

## 链式工具

如果我们知道我们只需要使用一个工具固定次数，我们可以创建一个链式工具来实现。让我们创建一个简单的链式工具，只是将用户指定的数字相乘。

![chain](../../static/img/tool_chain.svg)

### 调用工具/函数

使用工具调用 API（有时也称为函数调用）是使用 LLMs 与工具一起使用的最可靠的方法之一。这仅适用于明确支持工具调用的模型。您可以在[此处](/docs/integrations/chat/)查看支持工具调用的模型，并在[此指南](/docs/how_to/function_calling)中了解有关如何使用工具调用的更多信息。

首先，我们将定义我们的模型和工具。我们将从一个单一工具 `multiply` 开始。

```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="llm"/>
```

我们将使用 `bind_tools` 将工具的定义作为每次调用模型的一部分传递进去，以便在适当的时候模型可以调用工具。

```python
llm_with_tools = llm.bind_tools([multiply])
```

当模型调用工具时，这将显示在输出的 `AIMessage.tool_calls` 属性中。

```python
msg = llm_with_tools.invoke("whats 5 times forty two")
msg.tool_calls
```

```output
[{'name': 'multiply',
  'args': {'first_int': 5, 'second_int': 42},
  'id': 'call_cCP9oA3tRz7HDrjFn1FdmDaG'}]
```

请查看[此处的 LangSmith 跟踪](https://smith.langchain.com/public/81ff0cbd-e05b-4720-bf61-2c9807edb708/r)。

### 调用工具

太棒了！我们能够生成工具调用。但是，如果我们想要实际调用工具怎么办？为了做到这一点，我们需要将生成的工具参数传递给我们的工具。作为一个简单的例子，我们将只提取第一个 tool_call 的参数。

```python
from operator import itemgetter
chain = llm_with_tools | (lambda x: x.tool_calls[0]["args"]) | multiply
chain.invoke("What's four times 23")
```

```output
92
```

请查看[此处的 LangSmith 跟踪](https://smith.langchain.com/public/16bbabb9-fc9b-41e5-a33d-487c42df4f85/r)。

## 代理

当我们知道任何用户输入所需的工具使用特定顺序时，链式工具非常有用。但对于某些用例，我们使用工具的次数取决于输入。在这些情况下，我们希望让模型自己决定使用工具的次数和顺序。[代理](/docs/tutorials/agents)正是让我们做到这一点的。

LangChain 提供了许多内置代理，针对不同的用例进行了优化。在[此处](/docs/concepts#agents)阅读有关所有代理类型的信息。

我们将使用[工具调用代理](https://api.python.langchain.com/en/latest/agents/langchain.agents.tool_calling_agent.base.create_tool_calling_agent.html)，这通常是最可靠的类型，也是大多数用例的推荐类型。

![agent](../../static/img/tool_agent.svg)

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_tool_calling_agent
```

```python
# 获取要使用的提示 - 可以替换为包含变量 "agent_scratchpad" 和 "input" 的任何提示！
prompt = hub.pull("hwchase17/openai-tools-agent")
prompt.pretty_print()
```

```output
================================ System Message ================================
You are a helpful assistant
============================= Messages Placeholder =============================
{chat_history}
================================ Human Message =================================
{input}
============================= Messages Placeholder =============================
{agent_scratchpad}
```

代理也很棒，因为它们使得使用多个工具变得容易。

```python
@tool
def add(first_int: int, second_int: int) -> int:
    "Add two integers."
    return first_int + second_int
@tool
def exponentiate(base: int, exponent: int) -> int:
    "Exponentiate the base to the exponent power."
    return base**exponent
tools = [multiply, add, exponentiate]
```

```python
# 构建工具调用代理
agent = create_tool_calling_agent(llm, tools, prompt)
```

```python
# 通过传入代理和工具创建代理执行器
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

有了代理，我们可以提出需要任意多次使用工具的问题：

```python
agent_executor.invoke(
    {
        "input": "Take 3 to the fifth power and multiply that by the sum of twelve and three, then square the whole result"
    }
)
```

```output
> 进入新的 AgentExecutor 链...
调用：`exponentiate`，参数为 `{'base': 3, 'exponent': 5}`
243
调用：`add`，参数为 `{'first_int': 12, 'second_int': 3}`
15
调用：`multiply`，参数为 `{'first_int': 243, 'second_int': 15}`
3645
调用：`exponentiate`，参数为 `{'base': 405, 'exponent': 2}`
164025将3的五次方乘以12和3的和，然后对整个结果进行平方的结果是243。
12和3的和是15。
将243乘以15的结果是3645。
最后，将3645平方的结果是164025。
> 链结束。
```

```output
{'input': 'Take 3 to the fifth power and multiply that by the sum of twelve and three, then square the whole result',
 'output': '将3的五次方乘以12和3的和，然后对整个结果进行平方的结果是243。\n\n12和3的和是15。\n\n将243乘以15的结果是3645。\n\n最后，将3645平方的结果是164025。'}
```

请查看[LangSmith 的跟踪结果](https://smith.langchain.com/public/eeeb27a4-a2f8-4f06-a3af-9c983f76146c/r)。