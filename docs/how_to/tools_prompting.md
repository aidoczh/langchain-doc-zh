---
sidebar_position: 3
---
# 如何为LLMs和聊天模型添加临时工具调用功能
:::caution
一些模型已经被微调以支持工具调用，并提供了专门的工具调用API。通常，这些模型在工具调用方面比未经微调的模型表现更好，并建议在需要工具调用的用例中使用。更多信息，请参阅[如何使用聊天模型调用工具](/docs/how_to/tool_calling/)指南。
:::
:::info 先决条件
本指南假设您熟悉以下概念：
- [LangChain工具](/docs/concepts/#tools)
- [函数/工具调用](https://python.langchain.com/v0.2/docs/concepts/#functiontool-calling)
- [聊天模型](/docs/concepts/#chat-models)
- [LLMs](/docs/concepts/#llms)
:::
在本指南中，我们将看到如何为聊天模型添加**临时**工具调用支持。这是一种替代方法，用于调用工具，如果您使用的模型不直接支持[工具调用](/docs/how_to/tool_calling/)。
我们只需编写一个提示，让模型调用适当的工具。以下是逻辑的示意图：
![chain](../../static/img/tool_chain.svg)
## 设置
我们需要安装以下软件包：
```python
%pip install --upgrade --quiet langchain langchain-community
```
如果您想使用LangSmith，请取消下面的注释：
```python
import getpass
import os
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```
您可以选择任何给定的模型进行本指南。请注意，这些模型中的大多数已经[支持原生工具调用](/docs/integrations/chat/)，因此对于这些模型，使用此处显示的提示策略是没有意义的，而应该遵循[如何使用聊天模型调用工具](/docs/how_to/tool_calling/)指南。
```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs openaiParams={`model="gpt-4"`} />
```
为了说明这个想法，我们将使用通过Ollama的`phi3`，它**不**原生支持工具调用。如果您也想使用`Ollama`，请按照[这些说明](/docs/integrations/chat/ollama/)进行操作。
```python
from langchain_community.llms import Ollama
model = Ollama(model="phi3")
```
## 创建一个工具
首先，让我们创建`add`和`multiply`工具。有关创建自定义工具的更多信息，请参阅[此指南](/docs/how_to/custom_tools)。
```python
from langchain_core.tools import tool
@tool
def multiply(x: float, y: float) -> float:
    """将两个数字相乘。"""
    return x * y
@tool
def add(x: int, y: int) -> int:
    "将两个数字相加。"
    return x + y
tools = [multiply, add]
# 让我们检查这些工具
for t in tools:
    print("--")
    print(t.name)
    print(t.description)
    print(t.args)
```
```output
--
multiply
将两个数字相乘。
{'x': {'title': 'X', 'type': 'number'}, 'y': {'title': 'Y', 'type': 'number'}}
--
add
将两个数字相加。
{'x': {'title': 'X', 'type': 'integer'}, 'y': {'title': 'Y', 'type': 'integer'}}
```
```python
multiply.invoke({"x": 4, "y": 5})
```
```output
20.0
```
## 创建我们的提示
我们将编写一个提示，指定模型可以访问的工具、这些工具的参数以及模型的期望输出格式。在这种情况下，我们将指示它输出一个形如`{"name": "...", "arguments": {...}}`的JSON块。
```python
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import render_text_description
rendered_tools = render_text_description(tools)
print(rendered_tools)
```
```output
multiply(x: float, y: float) -> float - 将两个数字相乘。
add(x: int, y: int) -> int - 将两个数字相加。
```
```python
system_prompt = f"""\
您是一个助手，可以访问以下一组工具。
以下是每个工具的名称和描述：
{rendered_tools}
根据用户输入，返回要使用的工具的名称和输入。
将您的响应作为具有'name'和'arguments'键的JSON块返回。
'arguments'应该是一个字典，其中键对应于参数名称，值对应于请求的值。
"""
prompt = ChatPromptTemplate.from_messages(
    [("system", system_prompt), ("user", "{input}")]
)
```
```python
chain = prompt | model
message = chain.invoke({"input": "what's 3 plus 1132"})
# 让我们看看模型的输出
# 如果模型是LLM（而不是聊天模型），输出将是一个字符串。
if isinstance(message, str):
    print(message)
else:  # 否则它是一个聊天模型
    print(message.content)
```
```output
{
    "name": "add",
    "arguments": {
        "x": 3,
        "y": 1132
    }
}
```
## 添加输出解析器
---
我们将使用 `JsonOutputParser` 来将我们模型的输出解析为 JSON 格式。
```python
from langchain_core.output_parsers import JsonOutputParser
chain = prompt | model | JsonOutputParser()
chain.invoke({"input": "what's thirteen times 4"})
```
```output
{'name': 'multiply', 'arguments': {'x': 13.0, 'y': 4.0}}
```
:::important
🎉 太棒了！ 🎉 现在我们已经指示我们的模型如何**请求**调用一个工具。
现在，让我们创建一些逻辑来实际运行这个工具！
:::
## 调用工具 🏃
既然模型可以请求调用一个工具，我们需要编写一个函数来实际调用这个工具。
这个函数将根据名称选择适当的工具，并传递模型选择的参数给它。
```python
from typing import Any, Dict, Optional, TypedDict
from langchain_core.runnables import RunnableConfig
class ToolCallRequest(TypedDict):
    """一个类型化字典，显示了传递给 invoke_tool 函数的输入。"""
    name: str
    arguments: Dict[str, Any]
def invoke_tool(
    tool_call_request: ToolCallRequest, config: Optional[RunnableConfig] = None
):
    """我们可以使用的执行工具调用的函数。
    参数:
        tool_call_request: 一个包含键名和参数的字典。
            名称必须与现有工具的名称匹配。
            参数是该工具的参数。
        config: 这是 LangChain 使用的包含回调、元数据等信息的配置信息。
            请参阅有关 RunnableConfig 的 LCEL 文档。
    返回:
        请求工具的输出
    """
    tool_name_to_tool = {tool.name: tool for tool in tools}
    name = tool_call_request["name"]
    requested_tool = tool_name_to_tool[name]
    return requested_tool.invoke(tool_call_request["arguments"], config=config)
```
让我们来测试一下 🧪!
```python
invoke_tool({"name": "multiply", "arguments": {"x": 3, "y": 5}})
```
```output
15.0
```
## 将其整合在一起
让我们将其整合在一起，创建一个具有加法和乘法功能的计算器链。
```python
chain = prompt | model | JsonOutputParser() | invoke_tool
chain.invoke({"input": "what's thirteen times 4.14137281"})
```
```output
53.83784653
```
## 返回工具输入
除了返回工具输出外，返回工具输入也是有帮助的。我们可以通过 LCEL 的 `RunnablePassthrough.assign` 来轻松实现这一点，将工具输出赋值给 RunnablePassrthrough 组件的输入（假定为字典），同时保留当前输入中的所有内容：
```python
from langchain_core.runnables import RunnablePassthrough
chain = (
    prompt | model | JsonOutputParser() | RunnablePassthrough.assign(output=invoke_tool)
)
chain.invoke({"input": "what's thirteen times 4.14137281"})
```
```output
{'name': 'multiply',
 'arguments': {'x': 13, 'y': 4.14137281},
 'output': 53.83784653}
```
## 接下来做什么？
这个指南展示了当模型正确输出所有所需的工具信息时的“快乐路径”。
实际上，如果您使用更复杂的工具，您将开始遇到来自模型的错误，特别是对于没有经过微调以用于调用工具和能力较弱的模型。
您需要准备好添加策略来改进模型的输出；例如，
1. 提供少量示例。
2. 添加错误处理（例如，捕获异常并将其反馈给 LLM，要求其更正先前的输出）。