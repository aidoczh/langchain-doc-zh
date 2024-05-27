# 【已废弃】实验性 Anthropi 工具包包装器

::: {.callout-warning}

Anthropic API 现已正式支持调用工具，因此不再需要此解决方法。请使用 [ChatAnthropic](/docs/integrations/chat/anthropic)，并使用 `langchain-anthropic>=0.1.5`。

:::

本笔记本展示了如何使用围绕 Anthropi 的实验性包装器，使其具有工具调用和结构化输出功能。它遵循 Anthropi 的指南 [这里](https://docs.anthropic.com/claude/docs/functions-external-tools)。

该包装器可从 `langchain-anthropic` 包中获取，并且还需要可选依赖项 `defusedxml` 用于解析 llm 的 XML 输出。

注意：这是一个将被 Anthropi 正式实现的工具调用功能取代的 beta 功能，但在此期间进行测试和实验非常有用。

```python
%pip install -qU langchain-anthropic defusedxml
from langchain_anthropic.experimental import ChatAnthropicTools
```

## 工具绑定

`ChatAnthropicTools` 提供了一个 `bind_tools` 方法，允许您传递 Pydantic 模型或 BaseTools 到 llm。

```python
from langchain_core.pydantic_v1 import BaseModel
class Person(BaseModel):
    name: str
    age: int
model = ChatAnthropicTools(model="claude-3-opus-20240229").bind_tools(tools=[Person])
model.invoke("I am a 27 year old named Erick")
```

```output
AIMessage(content='', additional_kwargs={'tool_calls': [{'function': {'name': 'Person', 'arguments': '{"name": "Erick", "age": "27"}'}, 'type': 'function'}]})
```

## 结构化输出

`ChatAnthropicTools` 还实现了用于提取值的 [`with_structured_output` 规范](/docs/how_to/structured_output)。注意：这可能不像显式提供工具调用的模型那样稳定。

```python
chain = ChatAnthropicTools(model="claude-3-opus-20240229").with_structured_output(
    Person
)
chain.invoke("I am a 27 year old named Erick")
```

```output
Person(name='Erick', age=27)
```