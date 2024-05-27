# 如何解析 YAML 输出

:::info 先决条件

本指南假定读者熟悉以下概念：

- [聊天模型](/docs/concepts/#chat-models)

- [输出解析器](/docs/concepts/#output-parsers)

- [提示模板](/docs/concepts/#prompt-templates)

- [结构化输出](/docs/how_to/structured_output)

- [链接可运行实例](/docs/how_to/sequence/)

:::

来自不同提供商的大型语言模型（LLMs）通常根据它们训练的具体数据具有不同的优势。这也意味着有些模型在生成 JSON 以外的格式输出方面可能更“优秀”和可靠。

这个输出解析器允许用户指定任意模式，并查询符合该模式的 LLMS 输出，使用 YAML 格式化他们的响应。

:::note

请记住，大型语言模型是有泄漏的抽象！您必须使用具有足够容量的 LLM 来生成格式良好的 YAML。

:::

```python
%pip install -qU langchain langchain-openai
import os
from getpass import getpass
os.environ["OPENAI_API_KEY"] = getpass()
```

我们使用 [Pydantic](https://docs.pydantic.dev) 与 [`YamlOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain.output_parsers.yaml.YamlOutputParser.html#langchain.output_parsers.yaml.YamlOutputParser) 来声明我们的数据模型，并为模型提供更多关于应生成何种类型 YAML 的上下文信息：

```python
from langchain.output_parsers import YamlOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI
# 定义您期望的数据结构。
class Joke(BaseModel):
    setup: str = Field(description="设置笑话的问题")
    punchline: str = Field(description="解答笑话的答案")
model = ChatOpenAI(temperature=0)
# 创建一个查询，旨在提示语言模型填充数据结构。
joke_query = "告诉我一个笑话。"
# 设置一个解析器 + 将指令注入到提示模板中。
parser = YamlOutputParser(pydantic_object=Joke)
prompt = PromptTemplate(
    template="回答用户的查询。\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
chain = prompt | model | parser
chain.invoke({"query": joke_query})
```

```output
Joke(setup="自行车为什么找不到回家的路？", punchline='因为它迷失了方向！')
```

解析器将自动解析输出的 YAML，并创建一个带有数据的 Pydantic 模型。我们可以看到解析器的 `format_instructions`，这些指令被添加到提示中：

```python
parser.get_format_instructions()
```

```output
'输出应格式化为符合以下 JSON 模式的 YAML 实例。\n\n# 示例\n## 模式\n```\n{"title": "Players", "description": "A list of players", "type": "array", "items": {"$ref": "#/definitions/Player"}, "definitions": {"Player": {"title": "Player", "type": "object", "properties": {"name": {"title": "Name", "description": "Player name", "type": "string"}, "avg": {"title": "Avg", "description": "Batting average", "type": "number"}}, "required": ["name", "avg"]}}}\n```\n## 格式良好的实例\n```\n- name: John Doe\n  avg: 0.3\n- name: Jane Maxfield\n  avg: 1.4\n```\n\n## 模式\n```\n{"properties": {"habit": { "description": "A common daily habit", "type": "string" }, "sustainable_alternative": { "description": "An environmentally friendly alternative to the habit", "type": "string"}}, "required": ["habit", "sustainable_alternative"]}\n```\n## 格式良好的实例\n```\nhabit: Using disposable water bottles for daily hydration.\nsustainable_alternative: Switch to a reusable water bottle to reduce plastic waste and decrease your environmental footprint.\n``` \n\n请遵循标准的 YAML 格式约定，缩进为 2 个空格，并确保数据类型严格遵循以下 JSON 模式：\n```\n{"properties": {"setup": {"title": "Setup", "description": "question to set up a joke", "type": "string"}, "punchline": {"title": "Punchline", "description": "answer to resolve the joke", "type": "string"}}, "required": ["setup", "punchline"]}\n```\n\n请始终使用三个反引号（```）将 YAML 输出括起来。请不要添加任何无效的 YAML 输出！'

```

您可以并且应该尝试在提示的其他部分中添加自己的格式提示，以增强或替换默认指令。

## 下一步

您已经学会如何提示模型返回 XML。接下来，请查看[关于获取结构化输出的更广泛指南](/docs/how_to/structured_output)，了解其他相关技术。