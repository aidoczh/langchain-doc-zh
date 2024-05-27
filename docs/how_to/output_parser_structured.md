# 如何使用输出解析器将LLM的响应解析为结构化格式

语言模型输出的是文本。但有时候，您可能希望获得比纯文本更结构化的信息。虽然一些模型提供商支持[内置的返回结构化输出的方法](/docs/how_to/structured_output)，但并非所有模型都支持。

输出解析器是帮助结构化语言模型响应的类。输出解析器必须实现两种主要方法：

- "获取格式指令"：返回一个字符串，其中包含语言模型输出的格式指令。

- "解析"：接受一个字符串（假设为语言模型的响应）并将其解析为某种结构。

还有一个可选的方法：

- "带提示解析"：接受一个字符串（假设为语言模型的响应）和一个提示（假设为生成此响应的提示）并将其解析为某种结构。提示主要是为了在OutputParser希望以某种方式重试或修复输出时提供信息，并且需要从提示中获取信息。

## 入门

下面我们将介绍主要类型的输出解析器，即`PydanticOutputParser`。

```python
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field, validator
from langchain_openai import OpenAI
model = OpenAI(model_name="gpt-3.5-turbo-instruct", temperature=0.0)
# 定义所需的数据结构。
class Joke(BaseModel):
    setup: str = Field(description="设置一个笑话的问题")
    punchline: str = Field(description="解答笑话的答案")
    # 您可以使用Pydantic轻松添加自定义验证逻辑。
    @validator("setup")
    def question_ends_with_question_mark(cls, field):
        if field[-1] != "?":
            raise ValueError("问题格式错误！")
        return field
# 设置解析器并将指令注入到提示模板中。
parser = PydanticOutputParser(pydantic_object=Joke)
prompt = PromptTemplate(
    template="回答用户的问题。\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
# 创建一个查询，用于提示语言模型填充数据结构。
prompt_and_model = prompt | model
output = prompt_and_model.invoke({"query": "给我讲个笑话。"})
parser.invoke(output)
```

```output
Joke(setup='为什么小鸡要过马路？', punchline='为了到对面！')
```

## LCEL

输出解析器实现了[Runnable接口](/docs/concepts#interface)，这是[LangChain表达式语言（LCEL）](/docs/concepts#langchain-expression-language)的基本构建块。这意味着它们支持`invoke`、`ainvoke`、`stream`、`astream`、`batch`、`abatch`、`astream_log`调用。

输出解析器接受字符串或`BaseMessage`作为输入，并可以返回任意类型。

```python
parser.invoke(output)
```

```output
Joke(setup='为什么小鸡要过马路？', punchline='为了到对面！')
```

除了手动调用解析器外，我们还可以将其添加到我们的`Runnable`序列中：

```python
chain = prompt | model | parser
chain.invoke({"query": "给我讲个笑话。"})
```

```output
Joke(setup='为什么小鸡要过马路？', punchline='为了到对面！')
```

虽然所有解析器都支持流式接口，但只有某些解析器可以通过部分解析的对象进行流式处理，因为这在很大程度上取决于输出类型。无法构建部分对象的解析器将简单地生成完全解析的输出。

例如，`SimpleJsonOutputParser`可以通过部分输出进行流式处理：

```python
from langchain.output_parsers.json import SimpleJsonOutputParser
json_prompt = PromptTemplate.from_template(
    "返回一个带有`answer`键的JSON对象，回答以下问题：{question}"
)
json_parser = SimpleJsonOutputParser()
json_chain = json_prompt | model | json_parser
```

```python
list(json_chain.stream({"question": "谁发明了显微镜？"}))
```

```output
[{},
 {'answer': ''},
 {'answer': 'Ant'},
 {'answer': 'Anton'},
 {'answer': 'Antonie'},
 {'answer': 'Antonie van'},
 {'answer': 'Antonie van Lee'},
 {'answer': 'Antonie van Leeu'},
 {'answer': 'Antonie van Leeuwen'},
 {'answer': 'Antonie van Leeuwenho'},
 {'answer': 'Antonie van Leeuwenhoek'}]
```

而`PydanticOutputParser`则不行：

```python
list(chain.stream({"query": "给我讲个笑话。"}))
```

```output
[Joke(setup='为什么小鸡要过马路？', punchline='为了到对面！')]
```