

# 如何解析 JSON 输出

:::info 先决条件

本指南假定读者熟悉以下概念：

- [聊天模型](/docs/concepts/#chat-models)

- [输出解析器](/docs/concepts/#output-parsers)

- [提示模板](/docs/concepts/#prompt-templates)

- [结构化输出](/docs/how_to/structured_output)

- [链接运行器](/docs/how_to/sequence/)

:::

虽然一些模型提供商支持[内置的方法返回结构化输出](/docs/how_to/structured_output)，但并非所有都支持。我们可以使用输出解析器来帮助用户通过提示指定任意的 JSON 模式，查询符合该模式的模型输出，最后将该模式解析为 JSON。

:::note

请记住，大型语言模型是有泄漏的抽象！您必须使用具有足够容量的大型语言模型来生成格式良好的 JSON。

:::

[`JsonOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.json.JsonOutputParser.html) 是一个内置选项，用于提示并解析 JSON 输出。虽然它在功能上类似于 [`PydanticOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.pydantic.PydanticOutputParser.html)，但它还支持流式返回部分 JSON 对象。

以下是如何将其与 [Pydantic](https://docs.pydantic.dev/) 一起使用以方便地声明预期模式的示例：

```python
%pip install -qU langchain langchain-openai
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass()
```
```python
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI
model = ChatOpenAI(temperature=0)
# 定义您期望的数据结构。
class Joke(BaseModel):
    setup: str = Field(description="设置笑话的问题")
    punchline: str = Field(description="解决笑话的答案")
# 还有一个用于提示语言模型填充数据结构的查询意图。
joke_query = "告诉我一个笑话。"
# 设置解析器 + 将指令注入提示模板。
parser = JsonOutputParser(pydantic_object=Joke)
prompt = PromptTemplate(
    template="回答用户的查询。\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
chain = prompt | model | parser
chain.invoke({"query": joke_query})
```
```output
{'setup': "为什么自行车不能自己站起来？",
 'punchline': '因为它太累了！'}
```

请注意，我们将解析器中的 `format_instructions` 直接传递到提示中。您可以并且应该尝试在提示的其他部分中添加自己的格式提示，以增强或替换默认指令：

```python
parser.get_format_instructions()
```
```output
'输出应格式化为符合以下 JSON 模式的 JSON 实例。\n\n例如，对于模式 {"properties": {"foo": {"title": "Foo", "description": "字符串列表", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}，对象 {"foo": ["bar", "baz"]} 是该模式的格式良好实例。对象 {"properties": {"foo": ["bar", "baz"]}} 不是格式良好的。\n\n这是输出模式：\n```\n{"properties": {"setup": {"title": "Setup", "description": "设置笑话的问题", "type": "string"}, "punchline": {"title": "Punchline", "description": "解决笑话的答案", "type": "string"}}, "required": ["setup", "punchline"]}\n```'
```

## 流式处理

如上所述，`JsonOutputParser` 和 `PydanticOutputParser` 之间的一个关键区别是 `JsonOutputParser` 输出解析器支持流式处理部分块。以下是其示例：

```python
for s in chain.stream({"query": joke_query}):
    print(s)
```
```output
{}
{'setup': ''}
{'setup': '为什么'}
{'setup': '为什么不能'}
{'setup': '为什么不能'}
{'setup': '为什么不能'}
{'setup': '为什么不能'}
{'setup': '为什么不能'}
{'setup': '为什么不能'}
{'setup': '为什么不能'}
{'setup': '为什么不能'}
{'setup': '为什么不能'}
{'setup': '为什么不能', 'punchline': ''}
{'setup': '为什么不能', 'punchline': '因为'}
{'setup': '为什么不能', 'punchline': '因为'}
{'setup': '为什么不能', 'punchline': '因为'}
{'setup': '为什么不能', 'punchline': '因为'}
{'setup': '为什么不能', 'punchline': '因为'}
{'setup': '为什么不能', 'punchline': '因为它太累了'}
{'setup': '为什么不能', 'punchline': '因为它太累了！'}
## 没有 Pydantic
你也可以在没有 Pydantic 的情况下使用 `JsonOutputParser`。这将提示模型返回 JSON，但不提供关于模式应该是什么的具体信息。
```python
joke_query = "Tell me a joke."
parser = JsonOutputParser()
prompt = PromptTemplate(
    template="Answer the user query.\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
chain = prompt | model | parser
chain.invoke({"query": joke_query})
```
```output

{'response': "Sure! Here's a joke for you: Why couldn't the bicycle stand up by itself? Because it was two tired!"}

```

## 下一步

你已经学会了一种提示模型返回结构化 JSON 的方法。接下来，查看[更广泛的获取结构化输出指南](/docs/how_to/structured_output)以了解其他技术。