# 如何解析 XML 输出

:::info 先决条件

本指南假设您熟悉以下概念：

- [聊天模型](/docs/concepts/#chat-models)

- [输出解析器](/docs/concepts/#output-parsers)

- [提示模板](/docs/concepts/#prompt-templates)

- [结构化输出](/docs/how_to/structured_output)

- [将可运行项链接在一起](/docs/how_to/sequence/)

:::

不同提供商的LLM在特定数据上的训练不同，因此它们的优势也不同。这也意味着，与JSON以外的格式生成输出方面，一些模型可能更“好”和更可靠。

本指南将向您展示如何使用[`XMLOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.xml.XMLOutputParser.html)来提示模型生成XML输出，然后将该输出解析为可用的格式。

:::note

请记住，大型语言模型是有泄漏的抽象！您必须使用足够容量的LLM来生成格式良好的XML。

:::

在以下示例中，我们使用Anthropic的Claude-2模型（https://docs.anthropic.com/claude/docs），这是一个针对XML标签进行了优化的模型之一。

```python
%pip install -qU langchain langchain-anthropic
import os
from getpass import getpass
os.environ["ANTHROPIC_API_KEY"] = getpass()
```

让我们从对模型的简单请求开始。

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.output_parsers import XMLOutputParser
from langchain_core.prompts import PromptTemplate
model = ChatAnthropic(model="claude-2.1", max_tokens_to_sample=512, temperature=0.1)
actor_query = "生成汤姆·汉克斯的简化电影作品列表。"
output = model.invoke(
    f"""{actor_query}
请将电影放在<movie></movie>标签中"""
)
print(output.content)
```

```output
以下是汤姆·汉克斯的简化电影作品列表，电影被放在XML标签中：
<movie>Splash</movie>
<movie>Big</movie>
<movie>A League of Their Own</movie>
<movie>Sleepless in Seattle</movie>
<movie>Forrest Gump</movie>
<movie>Toy Story</movie>
<movie>Apollo 13</movie>
<movie>Saving Private Ryan</movie>
<movie>Cast Away</movie>
<movie>The Da Vinci Code</movie>
```

这实际上效果不错！但是将该XML解析为更易于使用的格式会更好。我们可以使用`XMLOutputParser`将默认的格式指令添加到提示中，并将输出的XML解析为字典：

```python
parser = XMLOutputParser()
# 我们将在下面的提示中添加这些指令
parser.get_format_instructions()
```

```output
'输出应格式化为XML文件。\n1. 输出应符合以下标签。 \n2. 如果没有给出标签，请自行添加。\n3. 记得始终打开和关闭所有标签。\n\n例如，对于标签["foo", "bar", "baz"]：\n1. 字符串"<foo>\n   <bar>\n      <baz></baz>\n   </bar>\n</foo>"是模式的格式良好的实例。 \n2. 字符串"<foo>\n   <bar>\n   </foo>"是格式不良的实例。\n3. 字符串"<foo>\n   <tag>\n   </tag>\n</foo>"是格式不良的实例。\n\n以下是输出标签：\n```\nNone\n```'
```

```python
prompt = PromptTemplate(
    template="""{query}\n{format_instructions}""",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
chain = prompt | model | parser
output = chain.invoke({"query": actor_query})
print(output)
```

```output
{'filmography': [{'movie': [{'title': 'Big'}, {'year': '1988'}]}, {'movie': [{'title': 'Forrest Gump'}, {'year': '1994'}]}, {'movie': [{'title': 'Toy Story'}, {'year': '1995'}]}, {'movie': [{'title': 'Saving Private Ryan'}, {'year': '1998'}]}, {'movie': [{'title': 'Cast Away'}, {'year': '2000'}]}]}
```

我们还可以添加一些标签以根据我们的需求定制输出。您可以在提示的其他部分中尝试添加自己的格式提示，以增强或替换默认指令：

```python
parser = XMLOutputParser(tags=["movies", "actor", "film", "name", "genre"])
# 我们将在下面的提示中添加这些指令
parser.get_format_instructions()
```

```output
'输出应格式化为XML文件。\n1. 输出应符合以下标签。 \n2. 如果没有给出标签，请自行添加。\n3. 记得始终打开和关闭所有标签。\n\n例如，对于标签["foo", "bar", "baz"]：\n1. 字符串"<foo>\n   <bar>\n      <baz></baz>\n   </bar>\n</foo>"是模式的格式良好的实例。 \n2. 字符串"<foo>\n   <bar>\n   </foo>"是格式不良的实例。\n3. 字符串"<foo>\n   <tag>\n   </tag>\n</foo>"是格式不良的实例。\n\n以下是输出标签：\n```\n[\'movies\', \'actor\', \'film\', \'name\', \'genre\']\n```'
```

```python
prompt = PromptTemplate(
    template="""{query}\n{format_instructions}""",
    input_variables=["query"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
chain = prompt | model | parser
output = chain.invoke({"query": actor_query})
print(output)
```

```输出
{'电影': [{'演员': [{'姓名': '汤姆·汉克斯'}, {'电影': [{'名称': '阿甘正传'}, {'类型': '剧情'}]}, {'电影': [{'名称': '荒岛余生'}, {'类型': '冒险'}]}, {'电影': [{'名称': '拯救大兵瑞恩'}, {'类型': '战争'}]}]}]}
```

这个输出解析器还支持部分数据流的处理。以下是一个示例：

```python
for s in chain.stream({"query": actor_query}):
    print(s)
```

```输出
{'电影': [{'演员': [{'姓名': '汤姆·汉克斯'}]}]}
{'电影': [{'演员': [{'电影': [{'名称': '阿甘正传'}]}]}]}
{'电影': [{'演员': [{'电影': [{'类型': '剧情'}]}]}]}
{'电影': [{'演员': [{'电影': [{'名称': '荒岛余生'}]}]}]}
{'电影': [{'演员': [{'电影': [{'类型': '冒险'}]}]}]}
{'电影': [{'演员': [{'电影': [{'名称': '拯救大兵瑞恩'}]}]}]}
{'电影': [{'演员': [{'电影': [{'类型': '战争'}]}]}]}
```

## 下一步

您已经学会如何提示模型返回 XML。接下来，请查看[获取结构化输出的更广泛指南](/docs/how_to/structured_output)，了解其他相关技术。

```