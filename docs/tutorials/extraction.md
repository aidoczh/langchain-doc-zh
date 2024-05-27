---

sidebar_position: 4

---

# 构建一个信息提取链

在本教程中，我们将构建一个链来从非结构化文本中提取结构化信息。

:::important

本教程仅适用于支持**函数/工具调用**的模型

:::

## 概念

我们将涵盖的概念包括：

- 使用[语言模型](/docs/concepts/#chat-models)

- 使用[函数/工具调用](/docs/concepts/#function-tool-calling)

- 使用[LangSmith](/docs/concepts/#langsmith)调试和跟踪应用程序

## 设置

### Jupyter Notebook

本指南（以及文档中的大多数其他指南）使用[Jupyter笔记本](https://jupyter.org/)，并假设读者也是如此。Jupyter笔记本非常适合学习如何使用LLM系统，因为往往会出现一些问题（意外的输出，API停机等），在交互环境中阅读指南是更好地理解它们的好方法。

这个和其他教程可能最方便在Jupyter笔记本中运行。请参阅[这里](https://jupyter.org/install)以获取安装说明。

### 安装

要安装LangChain，请运行以下命令：

import Tabs from '@theme/Tabs';

import TabItem from '@theme/TabItem';

import CodeBlock from "@theme/CodeBlock";

<Tabs>

  <TabItem value="pip" label="Pip" default>

    <CodeBlock language="bash">pip install langchain</CodeBlock>

  </TabItem>

  <TabItem value="conda" label="Conda">

    <CodeBlock language="bash">conda install langchain -c conda-forge</CodeBlock>

  </TabItem>

</Tabs>

有关更多详细信息，请参阅我们的[安装指南](/docs/how_to/installation)。

### LangSmith

使用LangChain构建的许多应用程序将包含多个步骤，并调用多个LLM调用。

随着这些应用程序变得越来越复杂，能够检查链或代理内部发生的情况变得至关重要。

最好的方法是使用[LangSmith](https://smith.langchain.com)。

在上面的链接上注册后，请确保设置环境变量以开始记录跟踪：

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```

或者，如果在笔记本中，可以使用以下代码设置：

```python
import getpass
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 模式

首先，我们需要描述我们想从文本中提取的信息。

我们将使用Pydantic定义一个示例模式来提取个人信息。

```python
from typing import Optional
from langchain_core.pydantic_v1 import BaseModel, Field
class Person(BaseModel):
    """Information about a person."""
    # ^ Person实体的文档字符串。
    # 此文档字符串将作为模式Person的描述发送给LLM，
    # 它可以帮助提高提取结果。
    # 注意：
    # 1. 每个字段都是`optional`的--这允许模型拒绝提取它！
    # 2. 每个字段都有一个`description`--这个描述由LLM使用。
    # 有一个好的描述可以帮助提高提取结果。
    name: Optional[str] = Field(default=None, description="The name of the person")
    hair_color: Optional[str] = Field(
        default=None, description="The color of the person's hair if known"
    )
    height_in_meters: Optional[str] = Field(
        default=None, description="Height measured in meters"
    )
```

在定义模式时有两个最佳实践：

1. 记录**属性**和**模式**本身：这些信息将发送给LLM，并用于提高信息提取的质量。

2. 不要强迫LLM编造信息！上面我们使用了`Optional`来允许LLM在不知道答案的情况下输出`None`。

:::important

为了获得最佳性能，请详细记录模式，并确保模型不会在文本中没有可提取的信息时强制返回结果。

:::

## 提取器

让我们使用上面定义的模式创建一个信息提取器。

```python
from typing import Optional
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.pydantic_v1 import BaseModel, Field
# 定义一个自定义提示来提供说明和任何其他上下文。
# 1) 您可以在提示模板中添加示例以提高提取质量
# 2) 引入其他参数以考虑上下文（例如，包括从中提取文本的文档的元数据）
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are an expert extraction algorithm. "
            "Only extract relevant information from the text. "
            "If you do not know the value of an attribute asked to extract, "
            "return null for the attribute's value.",
        ),
        # 请参阅有关使用参考示例提高性能的操作指南。
        # MessagesPlaceholder('examples'),
        ("human", "{text}"),
    ]
)
```

我们需要使用支持函数/工具调用的模型。

请查看[文档](/docs/concepts#function-tool-calling)，了解可以与此API一起使用的一些模型列表。

```python
from langchain_mistralai import ChatMistralAI
llm = ChatMistralAI(model="mistral-large-latest", temperature=0)
runnable = prompt | llm.with_structured_output(schema=Person)
```

```output
/Users/harrisonchase/workplace/langchain/libs/core/langchain_core/_api/beta_decorator.py:87: LangChainBetaWarning: 方法`ChatMistralAI.with_structured_output`处于测试阶段。正在积极开发中，因此API可能会更改。
  warn_beta(
```

让我们来测试一下

```python
text = "Alan Smith is 6 feet tall and has blond hair."
runnable.invoke({"text": text})
```

```output
Person(name='Alan Smith', hair_color='blond', height_in_meters='1.83')
```

:::important 

提取是生成式的 🤯

LLM是生成模型，因此它们可以做一些非常酷的事情，比如正确提取以英尺提供的人的身高并转换成米！

:::

我们可以在这里看到LangSmith的追踪: https://smith.langchain.com/public/44b69a63-3b3b-47b8-8a6d-61b46533f015/r

## 多个实体

在**大多数情况下**，您应该提取一个实体列表，而不是单个实体。

通过在彼此内部嵌套模型，可以轻松实现这一点。

```python
from typing import List, Optional
from langchain_core.pydantic_v1 import BaseModel, Field
class Person(BaseModel):
    """关于一个人的信息。"""
    # ^ 人物Person的文档字符串。
    # 此文档字符串将作为模式Person的描述发送到LLM，
    # 它可以帮助改善提取结果。
    # 请注意：
    # 1. 每个字段都是`optional` -- 这允许模型拒绝提取它！
    # 2. 每个字段都有一个`description` -- 此描述由LLM使用。
    # 有一个良好的描述可以帮助改善提取结果。
    name: Optional[str] = Field(default=None, description="人的姓名")
    hair_color: Optional[str] = Field(
        default=None, description="如果已知，人的头发颜色"
    )
    height_in_meters: Optional[str] = Field(
        default=None, description="以米为单位的身高"
    )
class Data(BaseModel):
    """关于人的提取数据。"""
    # 创建一个模型，以便我们可以提取多个实体。
    people: List[Person]
```

:::important

这里的提取可能不完美。请继续查看如何使用**参考示例**来提高提取质量，并查看**指南**部分！

:::

```python
runnable = prompt | llm.with_structured_output(schema=Data)
text = "我的名字是杰夫，我的头发是黑色的，身高是6英尺。安娜的头发颜色和我一样。"
runnable.invoke({"text": text})
```

```output
Data(people=[Person(name='Jeff', hair_color=None, height_in_meters=None), Person(name='Anna', hair_color=None, height_in_meters=None)])
```

:::tip

当模式适应提取**多个实体**时，它还允许模型在文本中没有相关信息时提取**零个实体**，通过提供一个空列表。

这通常是一个**好**的事情！它允许在实体上指定**必需**属性，而不一定强制模型检测此实体。

:::

我们可以在这里看到LangSmith的追踪: https://smith.langchain.com/public/7173764d-5e76-45fe-8496-84460bd9cdef/r

## 下一步

现在您已经了解了使用LangChain进行提取的基础知识，可以继续阅读其他操作指南：

- [添加示例](/docs/how_to/extraction_examples): 学习如何使用**参考示例**来提高性能。

- [处理长文本](/docs/how_to/extraction_long_text): 如果文本不适合LLM的上下文窗口，应该怎么办？

- [使用解析方法](/docs/how_to/extraction_parse): 使用基于提示的方法来提取不支持**工具/函数调用**的模型。

```