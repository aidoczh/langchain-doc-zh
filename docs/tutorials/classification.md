---

title: Tagging

sidebar_class_name: hidden

---

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/use_cases/tagging.ipynb)

# 文本分类

标记（Tagging）意味着使用类别对文档进行标记，例如：

- 情感

- 语言

- 风格（正式、非正式等）

- 涉及的主题

- 政治倾向

![图片描述](../../static/img/tagging.png)

## 概述

标记有几个组成部分：

- `function`：与[提取](/docs/tutorials/extraction)类似，标记使用[functions](https://openai.com/blog/function-calling-and-other-api-updates)来指定模型如何标记文档

- `schema`：定义我们希望如何标记文档

## 快速开始

让我们看一个非常简单的示例，演示如何在 LangChain 中使用 OpenAI 工具进行标记。我们将使用 OpenAI 模型支持的 [`with_structured_output`](/docs/how_to/structured_output) 方法：

```python
%pip install --upgrade --quiet langchain langchain-openai
# 设置环境变量 OPENAI_API_KEY 或从 .env 文件加载:
# import dotenv
# dotenv.load_dotenv()
```

让我们在我们的 schema 中指定一个 Pydantic 模型，包括一些属性及其在 schema 中的预期类型。

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI
tagging_prompt = ChatPromptTemplate.from_template(
    """
    从以下段落中提取所需的信息。
    仅提取“Classification”功能中提到的属性。
    段落：
    {input}
    """
)
class Classification(BaseModel):
    sentiment: str = Field(description="文本的情感")
    aggressiveness: int = Field(
        description="文本的攻击性，范围从1到10"
    )
    language: str = Field(description="文本所使用的语言")
# LLM
llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0125").with_structured_output(
    Classification
)
tagging_chain = tagging_prompt | llm
```

```python
inp = "Estoy increiblemente contento de haberte conocido! Creo que seremos muy buenos amigos!"
tagging_chain.invoke({"input": inp})
```

```output
Classification(sentiment='positive', aggressiveness=1, language='Spanish')
```

如果我们想要 JSON 输出，只需调用 `.dict()`：

```python
inp = "Estoy muy enojado con vos! Te voy a dar tu merecido!"
res = tagging_chain.invoke({"input": inp})
res.dict()
```

```output
{'sentiment': 'negative', 'aggressiveness': 8, 'language': 'Spanish'}
```

正如我们在示例中所看到的，它正确地解释了我们的意图。结果会有所不同，例如，不同语言的情感（'positive'，'enojado' 等）。

我们将在下一节中看到如何控制这些结果。

## 更精细的控制

仔细的 schema 定义可以让我们更好地控制模型的输出。

具体来说，我们可以定义：

- 每个属性的可能值

- 描述以确保模型理解该属性

- 要返回的必需属性

让我们重新声明我们的 Pydantic 模型，使用枚举来控制前面提到的每个方面：

```python
class Classification(BaseModel):
    sentiment: str = Field(..., enum=["happy", "neutral", "sad"])
    aggressiveness: int = Field(
        ...,
        description="描述陈述的攻击性，数字越高攻击性越强",
        enum=[1, 2, 3, 4, 5],
    )
    language: str = Field(
        ..., enum=["spanish", "english", "french", "german", "italian"]
    )
```

```python
tagging_prompt = ChatPromptTemplate.from_template(
    """
    从以下段落中提取所需的信息。
    仅提取“Classification”功能中提到的属性。
    段落：
    {input}
    """
)
llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0125").with_structured_output(
    Classification
)
chain = tagging_prompt | llm
```

现在，答案将按我们期望的方式受限制：

```python
inp = "Estoy increiblemente contento de haberte conocido! Creo que seremos muy buenos amigos!"
chain.invoke({"input": inp})
```

```output
Classification(sentiment='happy', aggressiveness=1, language='spanish')
```

```python
inp = "Estoy muy enojado con vos! Te voy a dar tu merecido!"
chain.invoke({"input": inp})
```

```output
Classification(sentiment='sad', aggressiveness=5, language='spanish')
```

```python
inp = "Weather is ok here, I can go outside without much more than a coat"
chain.invoke({"input": inp})
```

```output
Classification(sentiment='neutral', aggressiveness=2, language='english')
```

[LangSmith追踪](https://smith.langchain.com/public/38294e04-33d8-4c5a-ae92-c2fe68be8332/r)让我们能够深入了解其内部结构：

![图片描述](../../static/img/tagging_trace.png)

### 深入了解

* 您可以使用[元数据标记器](/docs/integrations/document_transformers/openai_metadata_tagger)文档转换器从LangChain `Document`中提取元数据。

* 这涵盖了与标记链相同的基本功能，只是应用于LangChain `Document`。