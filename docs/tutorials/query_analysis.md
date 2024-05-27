---

sidebar_position: 0

---

# 构建一个查询分析系统

本页面将展示如何在一个基本的端到端示例中使用查询分析。这将涵盖创建一个简单的搜索引擎，展示将原始用户问题传递给搜索引擎时出现的故障模式，以及查询分析如何帮助解决这个问题的示例。有很多不同的查询分析技术，这个端到端示例不会展示所有的技术。

在这个示例中，我们将在 LangChain 的 YouTube 视频上进行检索。

## 设置

#### 安装依赖

```python
# %pip install -qU langchain langchain-community langchain-openai youtube-transcript-api pytube langchain-chroma
```

#### 设置环境变量

我们将在这个示例中使用 OpenAI：

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# 可选的，取消注释以使用 LangSmith 进行跟踪运行。在这里注册：https://smith.langchain.com。
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

### 加载文档

我们可以使用 `YouTubeLoader` 来加载一些 LangChain 视频的字幕：

```python
from langchain_community.document_loaders import YoutubeLoader
urls = [
    "https://www.youtube.com/watch?v=HAn9vnJy6S4",
    "https://www.youtube.com/watch?v=dA1cHGACXCo",
    "https://www.youtube.com/watch?v=ZcEMLz27sL4",
    "https://www.youtube.com/watch?v=hvAPnpSfSGo",
    "https://www.youtube.com/watch?v=EhlPDL4QrWY",
    "https://www.youtube.com/watch?v=mmBo8nlu2j0",
    "https://www.youtube.com/watch?v=rQdibOsL1ps",
    "https://www.youtube.com/watch?v=28lC4fqukoc",
    "https://www.youtube.com/watch?v=es-9MgxB-uc",
    "https://www.youtube.com/watch?v=wLRHwKuKvOE",
    "https://www.youtube.com/watch?v=ObIltMaRJvY",
    "https://www.youtube.com/watch?v=DjuXACWYkkU",
    "https://www.youtube.com/watch?v=o7C9ld6Ln-M",
]
docs = []
for url in urls:
    docs.extend(YoutubeLoader.from_youtube_url(url, add_video_info=True).load())
```

```python
import datetime
# 添加一些额外的元数据：视频的发布年份
for doc in docs:
    doc.metadata["publish_year"] = int(
        datetime.datetime.strptime(
            doc.metadata["publish_date"], "%Y-%m-%d %H:%M:%S"
        ).strftime("%Y")
    )
```

这是我们加载的视频的标题：

```python
[doc.metadata["title"] for doc in docs]
```

```output
['OpenGPTs',
 'Building a web RAG chatbot: using LangChain, Exa (prev. Metaphor), LangSmith, and Hosted Langserve',
 'Streaming Events: Introducing a new `stream_events` method',
 'LangGraph: Multi-Agent Workflows',
 'Build and Deploy a RAG app with Pinecone Serverless',
 'Auto-Prompt Builder (with Hosted LangServe)',
 'Build a Full Stack RAG App With TypeScript',
 'Getting Started with Multi-Modal LLMs',
 'SQL Research Assistant',
 'Skeleton-of-Thought: Building a New Template from Scratch',
 'Benchmarking RAG over LangChain Docs',
 'Building a Research Assistant from Scratch',
 'LangServe and LangChain Templates Webinar']
```

这是每个视频关联的元数据。我们可以看到每个文档还有一个标题、观看次数、发布日期和长度：

```python
docs[0].metadata
```

```output
{'source': 'HAn9vnJy6S4',
 'title': 'OpenGPTs',
 'description': 'Unknown',
 'view_count': 7210,
 'thumbnail_url': 'https://i.ytimg.com/vi/HAn9vnJy6S4/hq720.jpg',
 'publish_date': '2024-01-31 00:00:00',
 'length': 1530,
 'author': 'LangChain',
 'publish_year': 2024}
```

这是一个文档内容的示例：

```python
docs[0].page_content[:500]
```

```output
"hello today I want to talk about open gpts open gpts is a project that we built here at linkchain uh that replicates the GPT store in a few ways so it creates uh end user-facing friendly interface to create different Bots and these Bots can have access to different tools and they can uh be given files to retrieve things over and basically it's a way to create a variety of bots and expose the configuration of these Bots to end users it's all open source um it can be used with open AI it can be us"
```

### 索引文档

每当我们进行检索时，我们需要创建一个文档索引，以便进行查询。我们将使用一个向量存储来索引我们的文档，并且我们将首先对它们进行分块，以使我们的检索更简洁和准确：

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(chunk_size=2000)
chunked_docs = text_splitter.split_documents(docs)
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_documents(
    chunked_docs,
    embeddings,
)
```

## 无查询分析的检索

我们可以直接对用户问题执行相似性搜索，以找到与问题相关的片段：

```python
search_results = vectorstore.similarity_search("如何构建 RAG 代理")
print(search_results[0].metadata["title"])
print(search_results[0].page_content[:500])
```

```output
使用 Pinecone 无服务器构建和部署 RAG 应用
嗨，我是 Lang 链团队的 Lance，今天我们将从头开始构建和部署一个 rag 应用，使用 Pinecone 无服务器列表，所以我们将逐步讲解完成此操作所需的所有代码，并使用这些幻灯片作为指南，为我们奠定基础。首先，什么是 rag？所以 Under Capoy 有一个非常好的可视化，展示了 LMS 作为一种新型操作系统的内核，当然，我们操作系统的核心组件之一是 th
```

这个方法效果相当不错！我们的第一个结果与问题非常相关。

如果我们想要搜索特定时间段内的结果呢？

```python
search_results = vectorstore.similarity_search("2023 年发布的关于 RAG 的视频")
print(search_results[0].metadata["title"])
print(search_results[0].metadata["publish_date"])
print(search_results[0].page_content[:500])
```

```output
OpenGPTs
2024-01-31
硬编码，它将始终在这里执行一个检索步骤，助手在这里决定是否执行检索步骤，有时这样做是好的，有时这样做是不好的，有时当我说嗨时，它不需要调用它的工具，但其他时候，您知道 llm 可能会出错，没有意识到它需要执行检索步骤，因此 rag 机器人将始终执行检索步骤，因此它更加专注，因为这也是一种更简单的架构，所以它总是
```

我们的第一个结果是来自 2024 年（尽管我们要求搜索 2023 年的视频），与输入不太相关。由于我们只是针对文档内容进行搜索，所以无法根据任何文档属性对结果进行过滤。

这只是可能出现的一个失败模式。现在让我们看看如何通过基本形式的查询分析来解决这个问题！

## 查询分析

我们可以使用查询分析来改善检索结果。这将涉及定义一个包含一些日期过滤器的**查询模式**，并使用一个函数调用模型将用户问题转换为结构化查询。

### 查询模式

在这种情况下，我们将为发布日期明确定义最小和最大属性，以便可以进行过滤。

```python
from typing import Optional
from langchain_core.pydantic_v1 import BaseModel, Field
class Search(BaseModel):
    """在关于软件库的教程视频数据库上进行搜索。"""
    query: str = Field(
        ...,
        description="应用于视频转录的相似性搜索查询。",
    )
    publish_year: Optional[int] = Field(None, description="视频发布年份")
```

### 查询生成

为了将用户问题转换为结构化查询，我们将利用 OpenAI 的工具调用 API。具体来说，我们将使用新的 [ChatModel.with_structured_output()](/docs/how_to/structured_output) 构造函数来处理将模式传递给模型和解析输出。

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
system = """您是将用户问题转换为数据库查询的专家。\
您可以访问一个关于构建 LLM 动力应用程序的软件库教程视频数据库。\
给定一个问题，返回一系列优化以检索最相关结果的数据库查询。\
如果有您不熟悉的缩写或词汇，请不要尝试重新表述。"""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{question}"),
    ]
)
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
structured_llm = llm.with_structured_output(Search)
query_analyzer = {"question": RunnablePassthrough()} | prompt | structured_llm
```

```output
/Users/bagatur/langchain/libs/core/langchain_core/_api/beta_decorator.py:86: LangChainBetaWarning: 函数 `with_structured_output` 处于测试阶段。它正在积极开发中，因此 API 可能会更改。
  warn_beta(
```

让我们看看我们的分析器为我们之前搜索的问题生成了什么查询：

```python
query_analyzer.invoke("如何构建 RAG 代理")
```

```output
Search(query='构建 RAG 代理', publish_year=None)
```

```python
query_analyzer.invoke("2023 年发布的关于 RAG 的视频")
```

```output
Search(query='RAG', publish_year=2023)
```

## 使用查询分析进行检索

我们的查询分析看起来相当不错；现在让我们尝试使用我们生成的查询来实际执行检索。

**注意：** 在我们的示例中，我们指定了 `tool_choice="Search"`。这将强制 LLM 调用一个 - 仅一个 - 工具，这意味着我们将始终有一个优化的查询来查找。请注意，情况并非总是如此 - 请参阅其他指南，了解如何处理未返回或返回多个优化查询的情况。

```python
from typing import List
from langchain_core.documents import Document
```

```python
def retrieval(search: Search) -> List[Document]:
    if search.publish_year is not None:
        # 这是特定于 Chroma 的语法，
        # 我们正在使用的向量数据库。
        _filter = {"publish_year": {"$eq": search.publish_year}}
    else:
        _filter = None
    return vectorstore.similarity_search(search.query, filter=_filter)
```

```python
retrieval_chain = query_analyzer | retrieval
```

现在，我们可以对之前的问题输入运行此链，看到它只产生了那一年的结果！

```python
results = retrieval_chain.invoke("RAG tutorial published in 2023")
```

```python
[(doc.metadata["title"], doc.metadata["publish_date"]) for doc in results]
```

```output
[('Getting Started with Multi-Modal LLMs', '2023-12-20 00:00:00'),
 ('LangServe and LangChain Templates Webinar', '2023-11-02 00:00:00'),
 ('Getting Started with Multi-Modal LLMs', '2023-12-20 00:00:00'),
 ('Building a Research Assistant from Scratch', '2023-11-16 00:00:00')]
```