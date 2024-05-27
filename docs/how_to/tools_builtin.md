---

sidebar_position: 4

sidebar_class_name: hidden

---

# 如何使用内置工具和工具包

:::info 先决条件

本指南假定您熟悉以下概念：

- [LangChain 工具](/docs/concepts/#tools)

- [LangChain 工具包](/docs/concepts/#tools)

:::

## 工具

LangChain 拥有大量第三方工具。请访问[工具集成](/docs/integrations/tools/)查看可用工具列表。

:::important

在使用第三方工具时，请确保您了解工具的工作原理、权限情况。请阅读其文档，并检查是否需要从安全角度考虑任何事项。请查看我们的[安全](https://python.langchain.com/v0.1/docs/security/)指南获取更多信息。

:::

让我们尝试一下[维基百科集成](/docs/integrations/tools/wikipedia/)。

```python
!pip install -qU wikipedia
```

```python
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
api_wrapper = WikipediaAPIWrapper(top_k_results=1, doc_content_chars_max=100)
tool = WikipediaQueryRun(api_wrapper=api_wrapper)
print(tool.invoke({"query": "langchain"}))
```

```output
Page: LangChain
Summary: LangChain is a framework designed to simplify the creation of applications
```

该工具具有以下默认关联项：

```python
print(f"Name: {tool.name}")
print(f"Description: {tool.description}")
print(f"args schema: {tool.args}")
print(f"returns directly?: {tool.return_direct}")
```

```output
Name: wiki-tool
Description: look up things in wikipedia
args schema: {'query': {'title': 'Query', 'description': 'query to look up in Wikipedia, should be 3 or less words', 'type': 'string'}}
returns directly?: True
```

## 自定义默认工具

我们还可以修改内置工具的名称、描述和参数的 JSON 模式。

在定义参数的 JSON 模式时，重要的是输入保持与函数相同，因此您不应更改它。但您可以轻松为每个输入定义自定义描述。

```python
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
from langchain_core.pydantic_v1 import BaseModel, Field
class WikiInputs(BaseModel):
    """维基百科工具的输入。"""
    query: str = Field(
        description="query to look up in Wikipedia, should be 3 or less words"
    )
tool = WikipediaQueryRun(
    name="wiki-tool",
    description="look up things in wikipedia",
    args_schema=WikiInputs,
    api_wrapper=api_wrapper,
    return_direct=True,
)
print(tool.run("langchain"))
```

```output
Page: LangChain
Summary: LangChain is a framework designed to simplify the creation of applications
```

```python
print(f"Name: {tool.name}")
print(f"Description: {tool.description}")
print(f"args schema: {tool.args}")
print(f"returns directly?: {tool.return_direct}")
```

```output
Name: wiki-tool
Description: look up things in wikipedia
args schema: {'query': {'title': 'Query', 'description': 'query to look up in Wikipedia, should be 3 or less words', 'type': 'string'}}
returns directly?: True
```

## 如何使用内置工具包

工具包是一组旨在一起使用以执行特定任务的工具。它们具有便捷的加载方法。

要获取可用的现成工具包完整列表，请访问[集成](/docs/integrations/toolkits/)。

所有工具包都公开了一个 `get_tools` 方法，该方法返回一个工具列表。

通常您应该这样使用它们：

```python
# 初始化一个工具包
toolkit = ExampleTookit(...)
# 获取工具列表
tools = toolkit.get_tools()
```