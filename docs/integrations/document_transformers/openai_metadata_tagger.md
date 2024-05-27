# OpenAI 元数据标记器

通常，将摄入的文档标记为结构化元数据（如文档标题、语气或长度）往往很有用，以便以后进行更有针对性的相似性搜索。但是，对于大量文档，手动执行此标记过程可能会很繁琐。

`OpenAIMetadataTagger` 文档转换器通过根据提供的模式从每个提供的文档中提取元数据来自动化此过程。它在底层使用可配置的 `OpenAI Functions` 驱动链，因此，如果您传递自定义的 LLM 实例，它必须是支持函数的 `OpenAI` 模型。

**注意：** 此文档转换器最适合完整的文档，因此最好在进行任何其他拆分或处理之前首先对其进行运行！

例如，假设您想要索引一组电影评论。您可以使用有效的 `JSON Schema` 对象初始化文档转换器，如下所示：

```python
from langchain_community.document_transformers.openai_functions import (
    create_metadata_tagger,
)
from langchain_core.documents import Document
from langchain_openai import ChatOpenAI
```

```python
schema = {
    "properties": {
        "movie_title": {"type": "string"},
        "critic": {"type": "string"},
        "tone": {"type": "string", "enum": ["positive", "negative"]},
        "rating": {
            "type": "integer",
            "description": "评论家对电影的星级评定",
        },
    },
    "required": ["movie_title", "critic", "tone"],
}
# 必须是支持函数的 OpenAI 模型
llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613")
document_transformer = create_metadata_tagger(metadata_schema=schema, llm=llm)
```

然后，您只需将文档转换器传递给一组文档，它将从内容中提取元数据：

```python
original_documents = [
    Document(
        page_content="The Bee Movie 评论\nBy Roger Ebert\n\n这是有史以来最棒的电影。5颗星中的4颗。"
    ),
    Document(
        page_content="教父 评论\nBy Anonymous\n\n这部电影超级无聊。1颗星中的1颗。",
        metadata={"reliable": False},
    ),
]
enhanced_documents = document_transformer.transform_documents(original_documents)
```

```python
import json
print(
    *[d.page_content + "\n\n" + json.dumps(d.metadata) for d in enhanced_documents],
    sep="\n\n---------------\n\n",
)
```

```output
The Bee Movie 评论
By Roger Ebert
这是有史以来最棒的电影。5颗星中的4颗。
{"movie_title": "The Bee Movie", "critic": "Roger Ebert", "tone": "positive", "rating": 4}
---------------
教父 评论
By Anonymous
这部电影超级无聊。1颗星中的1颗。
{"movie_title": "The Godfather", "critic": "Anonymous", "tone": "negative", "rating": 1, "reliable": false}
```

然后，新文档可以在加载到向量存储之前通过文本拆分器进一步处理。提取的字段不会覆盖现有的元数据。

您还可以使用 Pydantic 模式初始化文档转换器：

```python
from typing import Literal
from pydantic import BaseModel, Field
class Properties(BaseModel):
    movie_title: str
    critic: str
    tone: Literal["positive", "negative"]
    rating: int = Field(description="5颗星中的评分")
document_transformer = create_metadata_tagger(Properties, llm)
enhanced_documents = document_transformer.transform_documents(original_documents)
print(
    *[d.page_content + "\n\n" + json.dumps(d.metadata) for d in enhanced_documents],
    sep="\n\n---------------\n\n",
)
```

```output
The Bee Movie 评论
By Roger Ebert
这是有史以来最棒的电影。5颗星中的4颗。
{"movie_title": "The Bee Movie", "critic": "Roger Ebert", "tone": "positive", "rating": 4}
---------------
教父 评论
By Anonymous
这部电影超级无聊。1颗星中的1颗。
{"movie_title": "The Godfather", "critic": "Anonymous", "tone": "negative", "rating": 1, "reliable": false}
```

## 自定义

您可以在文档转换器构造函数中传递标记链的标准 LLMChain 参数。例如，如果您想要要求 LLM 关注输入文档中的特定细节，或以某种风格提取元数据，您可以传递自定义提示：

```python
from langchain_core.prompts import ChatPromptTemplate
prompt = ChatPromptTemplate.from_template(
    """从以下文本中提取相关信息。
匿名评论家实际上是罗杰·伊伯特。
{input}
"""
)
document_transformer = create_metadata_tagger(schema, llm, prompt=prompt)
enhanced_documents = document_transformer.transform_documents(original_documents)
print(
    *[d.page_content + "\n\n" + json.dumps(d.metadata) for d in enhanced_documents],
    sep="\n\n---------------\n\n",
)
```

```output
《蜜蜂电影》评论
罗杰·艾伯特（Roger Ebert）
这是有史以来最伟大的电影。5颗星中的4颗。
{"movie_title": "The Bee Movie", "critic": "Roger Ebert", "tone": "positive", "rating": 4}
---------------
《教父》评论
匿名
这部电影实在是太无聊了。5颗星中的1颗。
{"movie_title": "The Godfather", "critic": "Roger Ebert", "tone": "negative", "rating": 1, "reliable": false}
```