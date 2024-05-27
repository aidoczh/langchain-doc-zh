# 如何仅使用提示（而不调用工具）进行信息提取

从LLM中生成结构化输出不需要调用工具功能。只要LLM能够很好地遵循提示指令，就可以让它输出特定格式的信息。

这种方法依赖于设计良好的提示，然后解析LLM的输出以使其能够很好地提取信息。

要在没有调用工具功能的情况下提取数据：

1. 指示LLM生成按预期格式的文本（例如，具有特定模式的JSON）；

2. 使用[输出解析器](/docs/concepts#output-parsers)将模型的响应结构化为所需的Python对象。

首先我们选择一个LLM：

```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="model" />
```

:::tip

本教程旨在简单易懂，但通常应包含参考示例以提高性能！

:::

## 使用PydanticOutputParser

以下示例使用内置的`PydanticOutputParser`来解析聊天模型的输出。

```python
from typing import List, Optional
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field, validator
class Person(BaseModel):
    """关于一个人的信息。"""
    name: str = Field(..., description="人的姓名")
    height_in_meters: float = Field(
        ..., description="人的身高（以米为单位）"
    )
class People(BaseModel):
    """文本中所有人的身份信息。"""
    people: List[Person]
# 设置解析器
parser = PydanticOutputParser(pydantic_object=People)
# 提示
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "回答用户的查询。将输出用`json`标签包裹起来\n{format_instructions}",
        ),
        ("human", "{query}"),
    ]
).partial(format_instructions=parser.get_format_instructions())
```

让我们看一下发送给模型的信息：

```python
query = "Anna is 23 years old and she is 6 feet tall"
```
```python
print(prompt.format_prompt(query=query).to_string())
```
```output
System: 回答用户的查询。将输出用`json`标签包裹起来
输出应该格式化为符合以下JSON模式的JSON实例。
例如，对于模式{"properties": {"foo": {"title": "Foo", "description": "a list of strings", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}，对象{"foo": ["bar", "baz"]}是一个格式良好的模式实例。对象{"properties": {"foo": ["bar", "baz"]}}不是格式良好的。
这是输出模式：
```

{"description": "文本中所有人的身份信息。", "properties": {"people": {"title": "People", "type": "array", "items": {"$ref": "#/definitions/Person"}}}, "required": ["people"], "definitions": {"Person": {"title": "Person", "description": "关于一个人的信息。", "type": "object", "properties": {"name": {"title": "Name", "description": "人的姓名", "type": "string"}, "height_in_meters": {"title": "Height In Meters", "description": "人的身高（以米为单位）", "type": "number"}}, "required": ["name", "height_in_meters"]}}}

```
Human: Anna is 23 years old and she is 6 feet tall
```

定义了我们的提示后，我们只需将提示、模型和输出解析器链接在一起：

```python
chain = prompt | model | parser
chain.invoke({"query": query})
```
```output
People(people=[Person(name='Anna', height_in_meters=1.83)])
```

请查看相关的[Langsmith跟踪](https://smith.langchain.com/public/92ed52a3-92b9-45af-a663-0a9c00e5e396/r)。

请注意，模式出现在两个地方：

1. 在提示中，通过`parser.get_format_instructions()`；

2. 在链中，用于接收格式化的输出并将其结构化为Python对象（在本例中为Pydantic对象`People`）。

## 自定义解析

如果需要，可以使用`LangChain`和`LCEL`轻松创建自定义提示和解析器。

要创建自定义解析器，请定义一个函数，将模型的输出（通常是[AIMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html)）解析为您选择的对象。

下面是一个简单的JSON解析器的实现示例。

```python
import json
import re
from typing import List, Optional
from langchain_anthropic.chat_models import ChatAnthropic
from langchain_core.messages import AIMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field, validator
class Person(BaseModel):
    """关于一个人的信息。"""
    name: str = Field(..., description="人的姓名")
    height_in_meters: float = Field(
        ..., description="人的身高（以米为单位）"
    )
class People(BaseModel):
    """文本中所有人的身份信息。"""
    people: List[Person]
# 提示
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "回答用户的查询。将你的答案输出为符合给定模式的JSON：```json\n{schema}\n```。请确保将答案用```json和```标签包裹起来",
        ),
        ("human", "{query}"),
    ]
).partial(schema=People.schema())
# 自定义解析器
def extract_json(message: AIMessage) -> List[dict]:
    """从字符串中提取嵌入在```json和```标签之间的JSON内容。
    参数：
        text (str)：包含JSON内容的文本。
    返回：
        list：提取的JSON字符串列表。
    """
    text = message.content
    # 定义正则表达式模式以匹配JSON块
    pattern = r"```json(.*?)```"
    # 在字符串中找到模式的所有非重叠匹配项
    matches = re.findall(pattern, text, re.DOTALL)
    # 返回匹配的JSON字符串列表，去除任何前导或尾随空格
    try:
        return [json.loads(match.strip()) for match in matches]
    except Exception:
        raise ValueError(f"解析失败：{message}")
```
```python
查询 = "Anna今年23岁，身高6英尺"
print(prompt.format_prompt(query=查询).to_string())
```
```output
System: 回答用户的查询。将您的答案输出为符合给定模式的JSON：```json
{'title': 'People', 'description': '关于文本中所有人的身份信息。', 'type': 'object', 'properties': {'people': {'title': 'People', 'type': 'array', 'items': {'$ref': '#/definitions/Person'}}}, 'required': ['people'], 'definitions': {'Person': {'title': 'Person', 'description': '关于一个人的信息。', 'type': 'object', 'properties': {'name': {'title': 'Name', 'description': '人的姓名', 'type': 'string'}, 'height_in_meters': {'title': 'Height In Meters', 'description': '以米为单位表示的人的身高。', 'type': 'number'}}, 'required': ['name', 'height_in_meters']}}}
```. 确保将答案包装在 ```json 和 ``` 标签中
Human: Anna今年23岁，身高6英尺
```python
链式 = prompt | model | extract_json
链式.invoke({"query": 查询})
```
```output

[{'people': [{'name': 'Anna', 'height_in_meters': 1.83}]}]

```

## 其他库

如果您想使用解析方法提取信息，请查看[Kor](https://eyurtsev.github.io/kor/)库。它是由`LangChain`的维护者之一编写的，它可以考虑到示例，允许控制格式（例如JSON或CSV），并以TypeScript表达模式。它似乎工作得很好！