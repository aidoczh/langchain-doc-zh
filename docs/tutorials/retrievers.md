# 向量存储和检索器

本教程将使您熟悉 LangChain 的向量存储和检索器抽象。这些抽象旨在支持从（向量）数据库和其他来源检索数据，以便与LLM工作流集成。它们对于需要获取数据以进行模型推理的应用程序非常重要，例如检索增强生成，即 RAG（请参阅我们的 RAG 教程[这里](/docs/tutorials/rag)）。

## 概念

本指南侧重于文本数据的检索。我们将涵盖以下概念：

- 文档；

- 向量存储；

- 检索器。

## 设置

### Jupyter Notebook

这个教程以及其他教程可能最方便在 Jupyter Notebook 中运行。请参阅[这里](https://jupyter.org/install)获取安装说明。

### 安装

此教程需要安装 `langchain`、`langchain-chroma` 和 `langchain-openai` 包：

import Tabs from '@theme/Tabs';

import TabItem from '@theme/TabItem';

import CodeBlock from "@theme/CodeBlock";

<Tabs>

  <TabItem value="pip" label="Pip" default>

    <CodeBlock language="bash">pip install langchain langchain-chroma langchain-openai</CodeBlock>

  </TabItem>

  <TabItem value="conda" label="Conda">

    <CodeBlock language="bash">conda install langchain langchain-chroma langchain-openai -c conda-forge</CodeBlock>

  </TabItem>

</Tabs>

更多详情，请参阅我们的[安装指南](/docs/how_to/installation)。

### LangSmith

您使用 LangChain 构建的许多应用程序将包含多个步骤，多次调用LLM调用。

随着这些应用程序变得越来越复杂，能够检查链或代理内部发生的情况变得至关重要。

最好的方法是使用[LangSmith](https://smith.langchain.com)。

在上面的链接注册后，请确保设置您的环境变量以开始记录跟踪：

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```

或者，在笔记本中，您可以这样设置：

```python
import getpass
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 文档

LangChain 实现了一个[Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)抽象，旨在表示文本单元和相关元数据。它有两个属性：

- `page_content`：表示内容的字符串；

- `metadata`：包含任意元数据的字典。

`metadata` 属性可以捕获有关文档来源、与其他文档的关系以及其他信息的信息。请注意，单个 `Document` 对象通常表示较大文档的一部分。

让我们生成一些示例文档：

```python
from langchain_core.documents import Document
documents = [
    Document(
        page_content="Dogs are great companions, known for their loyalty and friendliness.",
        metadata={"source": "mammal-pets-doc"},
    ),
    Document(
        page_content="Cats are independent pets that often enjoy their own space.",
        metadata={"source": "mammal-pets-doc"},
    ),
    Document(
        page_content="Goldfish are popular pets for beginners, requiring relatively simple care.",
        metadata={"source": "fish-pets-doc"},
    ),
    Document(
        page_content="Parrots are intelligent birds capable of mimicking human speech.",
        metadata={"source": "bird-pets-doc"},
    ),
    Document(
        page_content="Rabbits are social animals that need plenty of space to hop around.",
        metadata={"source": "mammal-pets-doc"},
    ),
]
```

在这里，我们生成了五个文档，其中包含三个不同“来源”的元数据。

## 向量存储

向量搜索是存储和搜索非结构化数据（例如非结构化文本）的常见方法。其思想是存储与文本相关联的数值向量。给定一个查询，我们可以将其嵌入为相同维度的向量，并使用向量相似度度量来识别存储中的相关数据。

LangChain的[VectorStore](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html)对象包含用于将文本和`Document`对象添加到存储中，并使用各种相似度度量进行查询的方法。它们通常使用[embedding](/docs/how_to/embed_text)模型进行初始化，该模型确定文本数据如何转换为数值向量。

LangChain 包含一套与不同向量存储技术集成的[集成](/docs/integrations/vectorstores)。一些向量存储由提供者（例如各种云提供商）托管，并需要特定凭据才能使用；一些（比如[Postgres](/docs/integrations/vectorstores/pgvector)）在可以在本地运行或通过第三方运行的独立基础设施中运行；其他一些可以在内存中运行，用于轻量级工作负载。在这里，我们将演示使用[Chroma](/docs/integrations/vectorstores/chroma)的 LangChain VectorStores 的用法，其中包括一个内存实现。

要实例化一个向量存储，通常需要提供一个[嵌入](/docs/how_to/embed_text)模型，以指定如何将文本转换为数值向量。在这里，我们将使用[OpenAI embeddings](/docs/integrations/text_embedding/openai/)。

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
vectorstore = Chroma.from_documents(
    documents,
    embedding=OpenAIEmbeddings(),
)
```

在这里调用`.from_documents`将文档添加到向量存储中。[VectorStore](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html)实现了用于添加文档的方法，这些方法也可以在实例化对象之后调用。大多数实现都允许您连接到现有的向量存储，例如，通过提供客户端、索引名称或其他信息。有关特定[集成](/docs/integrations/vectorstores)的详细信息，请参阅文档。

一旦我们实例化了包含文档的`VectorStore`，我们就可以对其进行查询。[VectorStore](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html)包括以下查询方法：

- 同步和异步；

- 通过字符串查询和通过向量；

- 返回相似性分数和不返回相似性分数；

- 通过相似性和[最大边际相关性](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html#langchain_core.vectorstores.VectorStore.max_marginal_relevance_search)（以平衡相似性和查询结果的多样性）。

这些方法通常会在输出中包含一系列[Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html#langchain_core.documents.base.Document)对象。

### 示例

根据字符串查询返回文档：

```python
vectorstore.similarity_search("cat")
```

```output
[Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Dogs are great companions, known for their loyalty and friendliness.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Rabbits are social animals that need plenty of space to hop around.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Parrots are intelligent birds capable of mimicking human speech.', metadata={'source': 'bird-pets-doc'})]
```

异步查询：

```python
await vectorstore.asimilarity_search("cat")
```

```output
[Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Dogs are great companions, known for their loyalty and friendliness.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Rabbits are social animals that need plenty of space to hop around.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Parrots are intelligent birds capable of mimicking human speech.', metadata={'source': 'bird-pets-doc'})]
```

返回分数：

```python
# 请注意，不同的提供者实现了不同的分数；这里的 Chroma
# 返回一个距离度量，应该与相似性成反比。
vectorstore.similarity_search_with_score("cat")
```

```output
[(Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'source': 'mammal-pets-doc'}),
  0.3751849830150604),
 (Document(page_content='Dogs are great companions, known for their loyalty and friendliness.', metadata={'source': 'mammal-pets-doc'}),
  0.48316916823387146),
 (Document(page_content='Rabbits are social animals that need plenty of space to hop around.', metadata={'source': 'mammal-pets-doc'}),
  0.49601367115974426),
 (Document(page_content='Parrots are intelligent birds capable of mimicking human speech.', metadata={'source': 'bird-pets-doc'}),
  0.4972994923591614)]
```

根据嵌入式查询返回文档：

```python
embedding = OpenAIEmbeddings().embed_query("cat")
vectorstore.similarity_search_by_vector(embedding)
```

```output
[Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Dogs are great companions, known for their loyalty and friendliness.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Rabbits are social animals that need plenty of space to hop around.', metadata={'source': 'mammal-pets-doc'}),
 Document(page_content='Parrots are intelligent birds capable of mimicking human speech.', metadata={'source': 'bird-pets-doc'})]
```

## 了解更多：

- [API 参考](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html)

- [操作指南](/docs/how_to/vectorstores)

- [特定集成文档](/docs/integrations/vectorstores)

## 检索器

LangChain `VectorStore` 对象不是 [Runnable](https://api.python.langchain.com/en/latest/core_api_reference.html#module-langchain_core.runnables) 的子类，因此不能直接集成到 LangChain 表达语言 [chains](/docs/concepts/#langchain-expression-language-lcel) 中。

LangChain [Retrievers](https://api.python.langchain.com/en/latest/core_api_reference.html#module-langchain_core.retrievers) 是可运行的，因此它们实现了一套标准方法（例如同步和异步的 `invoke` 和 `batch` 操作），并且设计用于在 LCEL chains 中使用。

我们可以自己创建一个简单版本的检索器，而不需要子类化 `Retriever`。如果我们选择要使用哪种方法来检索文档，我们可以轻松创建一个可运行的。下面我们将围绕 `similarity_search` 方法构建一个检索器：

```python
from typing import List
from langchain_core.documents import Document
from langchain_core.runnables import RunnableLambda
retriever = RunnableLambda(vectorstore.similarity_search).bind(k=1)  # 选择顶部结果
retriever.batch(["cat", "shark"])
```

```output
[[Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'source': 'mammal-pets-doc'})],
 [Document(page_content='Goldfish are popular pets for beginners, requiring relatively simple care.', metadata={'source': 'fish-pets-doc'})]]
```

Vectorstores 实现了一个 `as_retriever` 方法，它将生成一个检索器，具体来说是 [VectorStoreRetriever](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStoreRetriever.html)。这些检索器包括特定的 `search_type` 和 `search_kwargs` 属性，用于标识要调用的基础向量存储的方法以及如何对其进行参数化。例如，我们可以使用以下方式复制上述内容：

```python
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 1},
)
retriever.batch(["cat", "shark"])
```

```output
[[Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'source': 'mammal-pets-doc'})],
 [Document(page_content='Goldfish are popular pets for beginners, requiring relatively simple care.', metadata={'source': 'fish-pets-doc'})]]
```

`VectorStoreRetriever` 支持 `"similarity"`（默认）、`"mmr"`（最大边际相关性，如上所述）和 `"similarity_score_threshold"` 的搜索类型。我们可以使用后者通过相似性分数对检索器输出的文档进行阈值处理。

检索器可以轻松地集成到更复杂的应用程序中，例如检索增强生成（RAG）应用程序，它将给定问题与检索到的上下文结合起来，形成 LLM 的提示。下面我们展示一个最简单的例子。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
message = """
使用提供的上下文回答这个问题。
{question}
上下文：
{context}
"""
prompt = ChatPromptTemplate.from_messages([("human", message)])
rag_chain = {"context": retriever, "question": RunnablePassthrough()} | prompt | llm
```

```python
response = rag_chain.invoke("tell me about cats")
print(response.content)
```

```output
Cats are independent pets that often enjoy their own space.
```

## 了解更多：

检索策略可以丰富而复杂。例如：

- 我们可以从查询中 [推断出硬规则和过滤器](/docs/how_to/self_query/)（例如，“使用 2020 年后发布的文档”）；

- 我们可以 [返回与检索到的上下文有关的文档](/docs/how_to/parent_document_retriever/)（例如，通过某种文档分类法）；

- 我们可以为每个上下文单元生成 [多个嵌入](/docs/how_to/multi_vector)；

- 我们可以从多个检索器中 [集成结果](/docs/how_to/ensemble_retriever)；

- 我们可以为文档分配权重，例如，将 [最近的文档](/docs/how_to/time_weighted_vectorstore/) 加权。

操作指南中的 [检索器](/docs/how_to#retrievers) 部分涵盖了这些和其他内置的检索策略。

还可以轻松地扩展 [BaseRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_core.retrievers.BaseRetriever.html) 类，以实现自定义检索器。请参阅我们的操作指南 [此处](/docs/how_to/custom_retriever)。