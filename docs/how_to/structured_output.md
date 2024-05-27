---

sidebar_position: 3

---

# 如何从模型中返回结构化数据

:::info 先决条件

本指南假定您熟悉以下概念：

- [聊天模型](/docs/concepts/#chat-models)

- [函数/工具调用](/docs/concepts/#functiontool-calling)

:::

让模型返回符合特定模式的输出通常很有用。一个常见的用例是从文本中提取数据以插入数据库或与其他下游系统一起使用。本指南涵盖了从模型获取结构化输出的几种策略。

## `.with_structured_output()` 方法

:::info 支持的模型

您可以在[这里找到支持此方法的模型列表](/docs/integrations/chat/)。

:::

这是获取结构化输出的最简单和最可靠的方法。`with_structured_output()` 适用于提供原生 API 以用于结构化输出的模型，比如工具/函数调用或 JSON 模式，并在幕后利用这些功能。

该方法接受一个模式作为输入，该模式指定所需输出属性的名称、类型和描述。该方法返回一个类似模型的 Runnable，但不是输出字符串或消息，而是输出与给定模式对应的对象。模式可以指定为 [JSON 模式](https://json-schema.org/) 或 Pydantic 类。如果使用 JSON 模式，则 Runnable 将返回一个字典，如果使用 Pydantic 类，则将返回 Pydantic 对象。

举个例子，让我们让一个模型生成一个笑话，并将设置部分与笑话部分分开：

import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs
  customVarName="llm"
/>

如果我们希望模型返回一个 Pydantic 对象，我们只需要传入所需的 Pydantic 类：

```python
from typing import Optional
from langchain_core.pydantic_v1 import BaseModel, Field
class Joke(BaseModel):
    """要告诉用户的笑话。"""
    setup: str = Field(description="笑话的设置部分")
    punchline: str = Field(description="笑话的结尾部分")
    rating: Optional[int] = Field(description="笑话有多有趣，从1到10分")
structured_llm = llm.with_structured_output(Joke)
structured_llm.invoke("告诉我一个关于猫的笑话")
```

```output
Joke(setup='猫为什么坐在电脑上？', punchline='为了盯着老鼠！', rating=None)
```

:::tip

除了 Pydantic 类的结构外，Pydantic 类的名称、文档字符串以及参数的名称和提供的描述也非常重要。大多数时候 `with_structured_output` 使用模型的函数/工具调用 API，您可以有效地将所有这些信息视为添加到模型提示中。

:::

我们还可以传入一个[JSON 模式](https://json-schema.org/)字典，如果您不想使用 Pydantic。在这种情况下，响应也是一个字典：

```python
json_schema = {
    "title": "joke",
    "description": "要告诉用户的笑话。",
    "type": "object",
    "properties": {
        "setup": {
            "type": "string",
            "description": "笑话的设置部分",
        },
        "punchline": {
            "type": "string",
            "description": "笑话的结尾部分",
        },
        "rating": {
            "type": "integer",
            "description": "笑话有多有趣，从1到10分",
        },
    },
    "required": ["setup", "punchline"],
}
structured_llm = llm.with_structured_output(json_schema)
structured_llm.invoke("告诉我一个关于猫的笑话")
```

```output
{'setup': '猫为什么坐在电脑上？',
 'punchline': '因为它想盯着老鼠！',
 'rating': 8}
```

### 从多个模式中选择

让模型从多个模式中选择的最简单方法是创建一个具有联合类型属性的父 Pydantic 类：

```python
from typing import Union
class ConversationalResponse(BaseModel):
    """以对话方式回应。友善而乐于助人。"""
    response: str = Field(description="对用户查询的对话回应")
class Response(BaseModel):
    output: Union[Joke, ConversationalResponse]
structured_llm = llm.with_structured_output(Response)
structured_llm.invoke("告诉我一个关于猫的笑话")
```

```output
Response(output=Joke(setup='猫为什么坐在电脑上？', punchline='为了盯着老鼠！', rating=8))
```

```python
structured_llm.invoke("你今天好吗？")
```

```output
Response(output=ConversationalResponse(response="我只是一个数字助手，所以我没有感觉，但我在这里，随时准备帮助您。我可以如何协助您？"))
```

另外，您可以使用直接调用的工具来让模型在选项之间进行选择，如果您选择的模型支持的话。这需要更多的解析和设置，但在某些情况下可以提供更好的性能，因为您不需要使用嵌套的模式。有关更多详细信息，请参阅[此操作指南](/docs/how_to/tool_calling/)。

### 流式输出

当输出类型为字典时（即，模式指定为JSON模式字典），我们可以从结构化模型中流式传输输出。

:::info

请注意，生成的是已经聚合的块，而不是增量。

:::

```python
structured_llm = llm.with_structured_output(json_schema)
for chunk in structured_llm.stream("给我讲一个关于猫的笑话"):
    print(chunk)
```

```output
{}
{'setup': ''}
{'setup': '为什么'}
{'setup': '为什么是'}
{'setup': '为什么是猫'}
{'setup': '为什么是猫坐'}
{'setup': '为什么是猫坐在'}
{'setup': '为什么是猫坐在电脑'}
{'setup': '为什么是猫坐在电脑上'}
{'setup': '为什么是猫坐在电脑上？'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': ''}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持一'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持一只'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持一只眼'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持一只眼睛'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持一只眼睛盯'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持一只眼睛盯着'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持一只眼睛盯着鼠标'}
{'setup': '为什么是猫坐在电脑上？', 'punchline': '因为它想要保持一只眼睛盯着鼠标！', 'rating': 8}
```

### 少样本提示

对于更复杂的模式，向提示中添加少样本示例非常有用。有几种方法可以实现这一点。

最简单和最通用的方法是将示例添加到提示中的系统消息中：

```python
from langchain_core.prompts import ChatPromptTemplate
system = """你是一个幽默的喜剧演员，擅长踢门笑话。\
返回一个具有设置（回答“谁在那里？”）和最终笑话（回答“<设置>谁？”）的笑话。
以下是一些笑话示例：
example_user: 给我讲一个关于飞机的笑话
example_assistant: {{"setup": "为什么飞机永远不会累？", "punchline": "因为它们有休息机翼！", "rating": 2}}
example_user: 再给我讲一个关于飞机的笑话
example_assistant: {{"setup": "货物", "punchline": "货物“呜呜”，但飞机“嗖嗖”！", "rating": 10}}
example_user: 现在讲一个关于毛毛虫的笑话
example_assistant: {{"setup": "毛毛虫", "punchline": "毛毛虫真的很慢，但看着我变成蝴蝶并独领风骚！", "rating": 5}}}"""
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", "{input}")])
few_shot_structured_llm = prompt | structured_llm
few_shot_structured_llm.invoke("关于啄木鸟有什么有趣的事情")
```

```output
{'setup': '啄木鸟',
 'punchline': "啄木鸟“敲敲”，但别担心，它们从不指望你回答门！", 'rating': 8}
```

当底层的输出结构化方法是工具调用时，我们可以将示例作为显式的工具调用传递。您可以查看您使用的模型是否在其API参考中使用了工具调用。

```python
from langchain_core.messages import AIMessage, HumanMessage, ToolMessage
examples = [
    HumanMessage("给我讲一个关于飞机的笑话", name="example_user"),
    AIMessage(
        "",
        name="example_assistant",
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "为什么飞机永远不会累？",
                    "punchline": "因为它们有休息机翼！",
                    "rating": 2,
                },
                "id": "1",
            }
        ],
    ),
    # 大多数工具调用模型希望在具有工具调用的AIMessage之后跟随ToolMessage(s)。
    ToolMessage("", tool_call_id="1"),
    # 一些模型还希望在ToolMessages之后跟随AIMessage，
    # 因此您可能需要在此处添加AIMessage。
    HumanMessage("再给我讲一个关于飞机的笑话", name="example_user"),
    AIMessage(
        "",
        name="example_assistant",
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "货物",
                    "punchline": "货物“呜呜”，但飞机“嗖嗖”！",
                    "rating": 10,
                },
                "id": "2",
            }
        ],
    ),
    ToolMessage("", tool_call_id="2"),
    HumanMessage("现在讲一个关于毛毛虫的笑话", name="example_user"),
    AIMessage(
        "",
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "毛毛虫",
                    "punchline": "毛毛虫真的很慢，但看着我变成蝴蝶并独领风骚！",
                    "rating": 5,
                },
                "id": "3",
            }
        ],
    ),
    ToolMessage("", tool_call_id="3"),
]
system = """你是一个幽默的喜剧演员。你擅长踢门笑话。\
返回一个具有设置（回答“谁在那里？”）\
和最终笑话（回答“<设置>谁？”）的笑话。"""
prompt = ChatPromptTemplate.from_messages(
    [("system", system), ("placeholder", "{examples}"), ("human", "{input}")]
)
few_shot_structured_llm = prompt | structured_llm
few_shot_structured_llm.invoke({"input": "鳄鱼", "examples": examples})
```

```output
{'setup': 'Crocodile',
 'punchline': "Crocodile 'see you later', but in a while, it becomes an alligator!",
 'rating': 7}
```

欲了解有关使用工具调用的零样本提示的更多信息，请参阅[此处](/docs/how_to/function_calling/#Few-shot-prompting)。

### （高级）指定结构化输出的方法

对于支持多种结构化输出方式的模型（即支持工具调用和 JSON 模式），您可以使用 `method=` 参数来指定要使用的方法。

:::info JSON 模式

如果使用 JSON 模式，则仍需在模型提示中指定所需的模式。您传递给 `with_structured_output` 的模式仅用于解析模型输出，不会像工具调用那样传递给模型。

要查看您正在使用的模型是否支持 JSON 模式，请查看其在 [API 参考](https://api.python.langchain.com/en/latest/langchain_api_reference.html) 中的条目。

:::

```python
structured_llm = llm.with_structured_output(Joke, method="json_mode")
structured_llm.invoke(
    "Tell me a joke about cats, respond in JSON with `setup` and `punchline` keys"
)
```

```output
Joke(setup='为什么猫坐在电脑上？', punchline='因为它想盯着老鼠！', rating=None)
```

## 直接提示和解析模型

并非所有模型都支持 `.with_structured_output()`，因为并非所有模型都支持工具调用或 JSON 模式。对于这类模型，您需要直接提示模型使用特定格式，并使用输出解析器从原始模型输出中提取结构化响应。

### 使用 `PydanticOutputParser`

以下示例使用内置的 [`PydanticOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.pydantic.PydanticOutputParser.html) 解析了提示模型以匹配给定的 Pydantic 模式的输出。请注意，我们直接从解析器的一个方法向提示中添加了 `format_instructions`：

```python
from typing import List
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
class Person(BaseModel):
    """关于一个人的信息。"""
    name: str = Field(..., description="人的姓名")
    height_in_meters: float = Field(
        ..., description="以米为单位表示的人的身高。"
    )
class People(BaseModel):
    """关于文本中所有人的身份信息。"""
    people: List[Person]
# 设置解析器
parser = PydanticOutputParser(pydantic_object=People)
# 提示
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "回答用户的查询。用 `json` 标记包裹输出\n{format_instructions}",
        ),
        ("human", "{query}"),
    ]
).partial(format_instructions=parser.get_format_instructions())
```

让我们看看发送给模型的信息：

```python
query = "Anna is 23 years old and she is 6 feet tall"
print(prompt.invoke(query).to_string())
```

```output
System: 回答用户的查询。用 `json` 标记包裹输出
输出应该以符合下面 JSON 模式的 JSON 实例格式化。
例如，对于模式 {"properties": {"foo": {"title": "Foo", "description": "a list of strings", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}，
对象 {"foo": ["bar", "baz"]} 是模式的格式良好的实例。对象 {"properties": {"foo": ["bar", "baz"]}} 不是格式良好的。
下面是输出模式：
```

{"description": "关于文本中所有人的身份信息。", "properties": {"people": {"title": "People", "type": "array", "items": {"$ref": "#/definitions/Person"}}}, "required": ["people"], "definitions": {"Person": {"title": "Person", "description": "关于一个人的信息。", "type": "object", "properties": {"name": {"title": "Name", "description": "人的姓名", "type": "string"}, "height_in_meters": {"title": "Height In Meters", "description": "以米为单位表示的人的身高。", "type": "number"}}, "required": ["name", "height_in_meters"]}}}

```
Human: Anna is 23 years old and she is 6 feet tall
```

现在让我们进行调用：

```python
chain = prompt | llm | parser
chain.invoke({"query": query})
```

```output
People(people=[Person(name='Anna', height_in_meters=1.8288)])
```

要深入了解如何使用输出解析器和提示技术进行结构化输出，请参阅[此指南](/docs/how_to/output_parser_structured)。

### 自定义解析

您还可以使用 [LangChain 表达式语言 (LCEL)](/docs/concepts/#langchain-expression-language) 创建自定义提示和解析器，使用普通函数从模型的输出中解析出结果。

```python
import json
import re
from typing import List
from langchain_core.messages import AIMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
class Person(BaseModel):
    """一个人的信息。"""
    name: str = Field(..., description="人的姓名")
    height_in_meters: float = Field(
        ..., description="人的身高（以米为单位）"
    )
class People(BaseModel):
    """文本中所有人的身份信息。"""
    people: List[Person]
# Prompt
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "回答用户的问题。将你的答案输出为符合给定模式的JSON：```json\n{schema}\n```。"
            "确保将答案包裹在```json和```标签中。",
        ),
        ("human", "{query}"),
    ]
).partial(schema=People.schema())
# Custom parser
def extract_json(message: AIMessage) -> List[dict]:
    """从字符串中提取嵌在```json和```标签之间的JSON内容。
    参数：
        text (str)：包含JSON内容的字符串。
    返回：
        list：提取的JSON字符串列表。
    """
    text = message.content
    # 定义匹配JSON块的正则表达式模式
    pattern = r"```json(.*?)```"
    # 在字符串中找到所有非重叠的模式匹配项
    matches = re.findall(pattern, text, re.DOTALL)
    # 返回匹配的JSON字符串列表，去除任何前导或尾随空格
    try:
        return [json.loads(match.strip()) for match in matches]
    except Exception:
        raise ValueError(f"解析失败：{message}")
```

这是发送给模型的提示：

```
query = "Anna今年23岁，身高6英尺"
print(prompt.format_prompt(query=query).to_string())
```

```output
System: Answer the user query. Output your answer as JSON that  matches the given schema: ```json
{'title': 'People', 'description': 'Identifying information about all people in a text.', 'type': 'object', 'properties': {'people': {'title': 'People', 'type': 'array', 'items': {'$ref': '#/definitions/Person'}}}, 'required': ['people'], 'definitions': {'Person': {'title': 'Person', 'description': 'Information about a person.', 'type': 'object', 'properties': {'name': {'title': 'Name', 'description': 'The name of the person', 'type': 'string'}, 'height_in_meters': {'title': 'Height In Meters', 'description': 'The height of the person expressed in meters.', 'type': 'number'}}, 'required': ['name', 'height_in_meters']}}}
```. Make sure to wrap the answer in ```json and ``` tags
Human: Anna is 23 years old and she is 6 feet tall
```

这是调用模型时的样子：

```
chain = prompt | llm | extract_json
chain.invoke({"query": query})
```
```output
[{'people': [{'name': 'Anna', 'height_in_meters': 1.8288}]}]
```
