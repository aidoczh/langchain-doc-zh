# 如何进行按用户检索

本指南演示了如何配置检索链的运行时属性。一个示例应用是根据用户限制可供检索器访问的文档。

在构建检索应用时，通常需要考虑多个用户。这意味着您可能不仅为一个用户存储数据，还为许多不同的用户存储数据，并且它们不应该能够看到彼此的数据。这意味着您需要能够配置您的检索链以仅检索特定信息。这通常涉及两个步骤。

**步骤1：确保您正在使用的检索器支持多个用户**

目前，在 LangChain 中没有统一的标志或过滤器来实现这一点。相反，每个向量存储和检索器可能有自己的标志或过滤器，并且可能被称为不同的名称（命名空间，多租户等）。对于向量存储，这通常作为一个关键字参数暴露，该参数在 `similarity_search` 过程中传入。通过阅读文档或源代码，找出您正在使用的检索器是否支持多个用户，如果支持，如何使用它。

注意：为不支持多用户的检索器添加文档和/或支持（或记录）多用户是贡献给 LangChain 的一个很好的方式。

**步骤2：将该参数添加为链的可配置字段**

这将使您能够在运行时轻松调用链并配置任何相关标志。有关配置的更多信息，请参阅[此文档](/docs/how_to/configure)。

现在，在运行时，您可以使用可配置字段调用此链。

## 代码示例

让我们看一个在代码中是什么样子的具体示例。我们将以 Pinecone 为例。

要配置 Pinecone，请设置以下环境变量：

- `PINECONE_API_KEY`：您的 Pinecone API 密钥

```python
from langchain_openai import OpenAIEmbeddings
from langchain_pinecone import PineconeVectorStore
embeddings = OpenAIEmbeddings()
vectorstore = PineconeVectorStore(index_name="test-example", embedding=embeddings)
vectorstore.add_texts(["i worked at kensho"], namespace="harrison")
vectorstore.add_texts(["i worked at facebook"], namespace="ankush")
```

```output
['ce15571e-4e2f-44c9-98df-7e83f6f63095']
```

`namespace` 的 pinecone 参数可用于分隔文档

```python
# 这将仅获取 Ankush 的文档
vectorstore.as_retriever(search_kwargs={"namespace": "ankush"}).get_relevant_documents(
    "where did i work?"
)
```

```output
[Document(page_content='i worked at facebook')]
```

```python
# 这将仅获取 Harrison 的文档
vectorstore.as_retriever(
    search_kwargs={"namespace": "harrison"}
).get_relevant_documents("where did i work?")
```

```output
[Document(page_content='i worked at kensho')]
```

现在我们可以创建要用于问答的链。

首先选择一个 LLM。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

这是基本的问答链设置。

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import (
    ConfigurableField,
    RunnablePassthrough,
)
template = """基于以下上下文回答问题：
{context}
问题：{question}
"""
prompt = ChatPromptTemplate.from_template(template)
retriever = vectorstore.as_retriever()
```

在这里，我们将检索器标记为具有可配置字段。所有向量存储检索器都有 `search_kwargs` 作为一个字段。这只是一个字典，具有特定于向量存储的字段。

这将使我们在调用链时传入 `search_kwargs` 的值。

```python
configurable_retriever = retriever.configurable_fields(
    search_kwargs=ConfigurableField(
        id="search_kwargs",
        name="搜索参数",
        description="要使用的搜索参数",
    )
)
```

现在我们可以使用可配置的检索器创建链

```python
chain = (
    {"context": configurable_retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```

现在我们可以使用可配置选项调用链。`search_kwargs` 是可配置字段的 id。值是用于 Pinecone 的搜索参数

```python
chain.invoke(
    "用户在哪里工作？",
    config={"configurable": {"search_kwargs": {"namespace": "harrison"}}},
)
```

```output
'用户在 Kensho 工作。'
```

```python
chain.invoke(
    "用户在哪里工作？",
    config={"configurable": {"search_kwargs": {"namespace": "ankush"}}},
)
```

```output
'用户在 Facebook 工作。'
```

要了解更多关于多用户的向量存储实现，请参考特定页面，如[Milvus](/docs/integrations/vectorstores/milvus)。