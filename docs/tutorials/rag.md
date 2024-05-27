# 构建一个检索增强生成（RAG）应用程序

LLM（大型语言模型）赋予的最强大的应用之一是复杂的问答（Q&A）聊天机器人。这些应用程序可以回答关于特定来源信息的问题。这些应用程序使用一种称为检索增强生成（RAG）的技术。

本教程将展示如何构建一个简单的基于文本数据源的问答应用程序。在此过程中，我们将介绍典型的问答架构，并突出展示更高级问答技术的额外资源。我们还将看到 LangSmith 如何帮助我们跟踪和理解我们的应用程序。随着应用程序复杂性的增加，LangSmith 将变得越来越有帮助。

## 什么是 RAG？

RAG 是一种用额外数据增强 LLM 知识的技术。

LLM 可以推理广泛的主题，但它们的知识仅限于它们在训练时的公共数据，到达特定时间点为止。如果您想构建能够推理私有数据或模型截止日期之后引入的数据的 AI 应用程序，您需要用特定信息增强模型的知识。将适当的信息引入并插入到模型提示中的过程称为检索增强生成（RAG）。

LangChain 有许多组件旨在帮助构建问答应用程序，以及更一般的 RAG 应用程序。

**注意**：这里我们专注于非结构化数据的问答。如果您对结构化数据上的 RAG 感兴趣，请查看我们关于在 SQL 数据上进行问答的教程。

## 概念

典型的 RAG 应用程序有两个主要组件：

**索引化**：从源中摄取数据并对其进行索引的流水线。*这通常是离线进行的*。

**检索和生成**：实际的 RAG 链，它在运行时接受用户查询，并从索引中检索相关数据，然后将其传递给模型。

从原始数据到答案的最常见完整序列如下：

### 索引化

1. **加载**：首先，我们需要加载我们的数据。这是通过[文档加载器](/docs/concepts/#document-loaders)完成的。

2. **拆分**：[文本拆分器](/docs/concepts/#text-splitters)将大的“文档”拆分为较小的块。这对于索引数据和将其传递给模型都很有用，因为大块更难搜索，也无法适应模型的有限上下文窗口。

3. **存储**：我们需要一个地方来存储和索引我们的拆分，以便以后可以对其进行搜索。通常使用[向量存储](/docs/concepts/#vectorstores)和[嵌入模型](/docs/concepts/#embedding-models)来完成。

![index_diagram](../../static/img/rag_indexing.png)

### 检索和生成

4. **检索**：给定用户输入，使用[检索器](/docs/concepts/#retrievers)从存储中检索相关拆分。

5. **生成**：[聊天模型](/docs/concepts/#chat-models) / [LLM](/docs/concepts/#llms)使用包含问题和检索数据的提示生成答案。

![retrieval_diagram](../../static/img/rag_retrieval_generation.png)

## 设置

### Jupyter Notebook

本指南（以及文档中的大多数其他指南）使用[Jupyter 笔记本](https://jupyter.org/)，并假定读者也在使用。Jupyter 笔记本非常适合学习如何使用 LLM 系统，因为通常会出现问题（意外输出、API 崩溃等），在交互式环境中阅读指南是更好地理解它们的好方法。

这些教程可能最方便在 Jupyter 笔记本中运行。请查看[此处](https://jupyter.org/install)获取安装说明。

### 安装

要安装 LangChain，请运行：

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

使用 LangChain 构建的许多应用程序将包含多个步骤，多次调用 LLM 调用。

随着这些应用程序变得越来越复杂，能够检查链或代理内部发生的情况变得至关重要。

最佳方法是使用[LangSmith](https://smith.langchain.com)。

在上面的链接注册后，请确保设置您的环境变量以开始记录跟踪：

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```

或者，在笔记本中，您可以使用以下代码设置：

```python
import getpass
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 预览

在本指南中，我们将在一个网站上构建一个问答应用程序。我们将使用Lilian Weng的[LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)博客文章作为特定的网站，这个网站允许我们提问关于博客内容的问题。

我们可以创建一个简单的索引流水线和RAG链来实现这一点，只需大约20行代码：

<ChatModelTabs customVarName="llm" />

```python
import bs4
from langchain import hub
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
# 加载、分块和索引博客内容。
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)
docs = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = text_splitter.split_documents(docs)
vectorstore = Chroma.from_documents(documents=splits, embedding=OpenAIEmbeddings())
# 检索并生成博客的相关片段。
retriever = vectorstore.as_retriever()
prompt = hub.pull("rlm/rag-prompt")
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)
rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
rag_chain.invoke("What is Task Decomposition?")
```

```output
'Task Decomposition is a process where a complex task is broken down into smaller, simpler steps or subtasks. This technique is utilized to enhance model performance on complex tasks by making them more manageable. It can be done by using language models with simple prompting, task-specific instructions, or with human inputs.'
```

```python
# 清理
vectorstore.delete_collection()
```

查看[LangSmith trace](https://smith.langchain.com/public/1c6ca97e-445b-4d00-84b4-c7befcbc59fe/r)。

## 详细步骤

让我们逐步详细地了解上述代码的执行过程。

### 1. 索引：加载 {#indexing-load}

我们首先需要加载博客文章的内容。我们可以使用[DocumentLoaders](/docs/concepts#document-loaders)来实现这一点，它们是从源加载数据并返回一系列[Documents](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)的对象。`Document`是一个带有一些`page_content`（str）和`metadata`（dict）的对象。

在这种情况下，我们将使用[WebBaseLoader](/docs/integrations/document_loaders/web_base)，它使用`urllib`从网址加载HTML，并使用`BeautifulSoup`将其解析为文本。我们可以通过向`bs_kwargs`传递参数来自定义HTML到文本的解析过程（参见[BeautifulSoup文档](https://beautiful-soup-4.readthedocs.io/en/latest/#beautifulsoup)）。在这种情况下，只有具有"class"为"post-content"、"post-title"或"post-header"的HTML标签是相关的，因此我们将删除所有其他标签。

```python
import bs4
from langchain_community.document_loaders import WebBaseLoader
# 仅保留完整HTML中的文章标题、标题和内容。
bs4_strainer = bs4.SoupStrainer(class_=("post-title", "post-header", "post-content"))
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs={"parse_only": bs4_strainer},
)
docs = loader.load()
len(docs[0].page_content)
```

```output
43131
```

```python
print(docs[0].page_content[:500])
```

```output
LLM Powered Autonomous Agents
Date: June 23, 2023  |  Estimated Reading Time: 31 min  |  Author: Lilian Weng
Building agents with LLM (large language model) as its core controller is a cool concept. Several proof-of-concepts demos, such as AutoGPT, GPT-Engineer and BabyAGI, serve as inspiring examples. The potentiality of LLM extends beyond generating well-written copies, stories, essays and programs; it can be framed as a powerful general problem solver.
Agent System Overview#
In
```

### 深入了解

`DocumentLoader`：从源加载数据并返回`Documents`列表的对象。

- [Docs](/docs/how_to#document-loaders)：关于如何使用`DocumentLoaders`的详细文档。

- [Integrations](/docs/integrations/document_loaders/)：160多个可供选择的集成。

- [Interface](https://api.python.langchain.com/en/latest/document_loaders/langchain_core.document_loaders.base.BaseLoader.html)：基本接口的API参考。

### 2. 索引：分块 {#indexing-split}

我们加载的文档长度超过42,000个字符。这对于许多模型的上下文窗口来说太长了。即使对于那些可以将完整的博客文章放入上下文窗口的模型来说，模型在非常长的输入中查找信息时也可能遇到困难。

为了解决这个问题，我们将把`Document`分成块进行嵌入和向量存储。这应该帮助我们在运行时只检索到博客文章的最相关部分。

在这种情况下，我们将把文档分成每个块1000个字符，块之间有200个字符的重叠。重叠有助于减轻将语句与与之相关的重要上下文分开的可能性。我们使用[RecursiveCharacterTextSplitter](/docs/how_to/recursive_text_splitter)，它将使用常见的分隔符（如换行符）递归地将文档分割，直到每个块的大小适合。这是通用文本用例的推荐文本分割器。

我们设置`add_start_index=True`，以便保留每个分割的文档在初始文档中开始的字符索引作为元数据属性"start_index"。

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000, chunk_overlap=200, add_start_index=True
)
all_splits = text_splitter.split_documents(docs)
len(all_splits)
```

```output
66
```

```python
len(all_splits[0].page_content)
```

```output
969
```

```python
all_splits[10].metadata
```

```output
{'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/',
 'start_index': 7056}
```

### 深入了解

`TextSplitter`: 将`Document`列表拆分为更小块的对象。是`DocumentTransformer`的子类。

- 探索[上下文感知的拆分器](/docs/how_to#text-splitters)，保留每个拆分在原始`Document`中的位置（“上下文”）

- [代码（py或js）](/docs/integrations/document_loaders/source_code)

- [科学论文](/docs/integrations/document_loaders/grobid)

- [接口](https://api.python.langchain.com/en/latest/base/langchain_text_splitters.base.TextSplitter.html)：基础接口的 API 参考。

`DocumentTransformer`: 对`Document`对象列表执行转换的对象。

- [文档](/docs/how_to#text-splitters)：关于如何使用`DocumentTransformers`的详细文档

- [集成](/docs/integrations/document_transformers/)

- [接口](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.transformers.BaseDocumentTransformer.html)：基础接口的 API 参考。

## 3. 索引：存储 {#indexing-store}

现在我们需要索引我们的 66 个文本块，以便在运行时可以对它们进行搜索。最常见的方法是嵌入每个文档拆分的内容，并将这些嵌入插入到矢量数据库（或矢量存储）中。当我们想要搜索我们的拆分时，我们会获取一个文本搜索查询，对其进行嵌入，并执行某种“相似度”搜索，以识别与我们查询嵌入最相似的存储拆分。最简单的相似度度量是余弦相似度 — 我们测量每对嵌入之间的角度的余弦值（它们是高维向量）。

我们可以使用 [Chroma](/docs/integrations/vectorstores/chroma) 矢量存储和 [OpenAIEmbeddings](/docs/integrations/text_embedding/openai) 模型，通过一个命令嵌入并存储所有文档拆分。

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
vectorstore = Chroma.from_documents(documents=all_splits, embedding=OpenAIEmbeddings())
```

### 深入了解

`Embeddings`: 文本嵌入模型的包装器，用于将文本转换为嵌入。

- [文档](/docs/how_to/embed_text)：关于如何使用嵌入的详细文档

- [集成](/docs/integrations/text_embedding/)：可供选择的 30 多种集成

- [接口](https://api.python.langchain.com/en/latest/embeddings/langchain_core.embeddings.Embeddings.html)：基础接口的 API 参考。

`VectorStore`: 矢量数据库的包装器，用于存储和查询嵌入。

- [文档](/docs/how_to/vectorstores)：关于如何使用矢量存储的详细文档

- [集成](/docs/integrations/vectorstores/)：可供选择的 40 多种集成

- [接口](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html)：基础接口的 API 参考。

这完成了**索引**部分的流程。此时，我们有一个可查询的矢量存储，其中包含博客文章的分块内容。给定用户问题，我们理想情况下应该能够返回回答问题的博客文章片段。

## 4. 检索和生成：检索 {#retrieval-and-generation-retrieve}

现在让我们编写实际的应用逻辑。我们希望创建一个简单的应用程序，接受用户问题，搜索与该问题相关的文档，将检索到的文档和初始问题传递给模型，并返回一个答案。

首先，我们需要定义搜索文档的逻辑。LangChain 定义了一个[Retriever](/docs/concepts#retrievers/)接口，它包装了一个可以根据字符串查询返回相关`Documents`的索引。

最常见的`Retriever`类型是[VectorStoreRetriever](/docs/how_to/vectorstore_retriever)，它使用矢量存储的相似度搜索功能来进行检索。任何`VectorStore`都可以轻松转换为`Retriever`，使用`VectorStore.as_retriever()`：

```python
retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": 6})
retrieved_docs = retriever.invoke("What are the approaches to Task Decomposition?")
len(retrieved_docs)
```

```output
6
```

```python
print(retrieved_docs[0].page_content)
```

```output
Tree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.
Task decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.
```

### 深入了解

向量存储通常用于检索，但也有其他方法可以进行检索。

`Retriever`：给定文本查询返回`Document`的对象

- [文档](/docs/how_to#retrievers)：接口和内置检索技术的进一步文档。其中包括：

  - `MultiQueryRetriever` [生成输入问题的变体](/docs/how_to/MultiQueryRetriever) 以提高检索命中率。

  - `MultiVectorRetriever`（下图）生成[嵌入的变体](/docs/how_to/multi_vector) 以提高检索命中率。

  - `Max marginal relevance` 选择检索到的文档中的相关性和多样性，以避免传递重复的上下文。

  - 可以使用元数据过滤器在向量存储检索期间过滤文档，例如使用[Self Query Retriever](/docs/how_to/self_query)。

- [集成](/docs/integrations/retrievers/)：与检索服务的集成。

- [接口](https://api.python.langchain.com/en/latest/retrievers/langchain_core.retrievers.BaseRetriever.html)：基本接口的 API 参考。

## 5. 检索和生成：生成

让我们将所有内容整合到一个链中，该链接收一个问题，检索相关文档，构建提示，将其传递给模型，并解析输出。

我们将使用 gpt-3.5-turbo OpenAI 聊天模型，但可以替换为任何 LangChain `LLM` 或 `ChatModel`。

<ChatModelTabs
  customVarName="llm"
  anthropicParams={`"model="claude-3-sonnet-20240229", temperature=0.2, max_tokens=1024"`}
/>

们将使用一个为 RAG 准备的提示，该提示已经上传到 LangChain 提示中心（[这里](https://smith.langchain.com/hub/rlm/rag-prompt)）。

```python
from langchain import hub
prompt = hub.pull("rlm/rag-prompt")
example_messages = prompt.invoke(
    {"context": "填充内容", "question": "填充问题"}
).to_messages()
example_messages
```

```output
[HumanMessage(content="您是用于问答任务的助手。使用以下检索到的上下文片段来回答问题。如果您不知道答案，只需说不知道。最多使用三个句子，保持答案简洁。\n问题：填充问题 \n上下文：填充内容 \n答案：")]
```

```python
print(example_messages[0].content)
```

```output
您是用于问答任务的助手。使用以下检索到的上下文片段来回答问题。如果您不知道答案，只需说不知道。最多使用三个句子，保持答案简洁。
问题：填充问题 
上下文：填充内容 
答案：
```

我们将使用[LCEL Runnable](/docs/concepts#langchain-expression-language)协议来定义链，从而使我们能够

- 以透明的方式将组件和函数串联在一起

- 在 LangSmith 中自动跟踪我们的链

- 获得流式、异步和批量调用的开箱即用功能。

以下是实现：

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)
rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
for chunk in rag_chain.stream("什么是任务分解？"):
    print(chunk, end="", flush=True)
```

```output
任务分解是将复杂任务分解为更小、更易管理的步骤或部分的过程。通常使用“思维链”或“思维树”等技术来完成此过程，这些技术指导模型“逐步思考”，将大型任务转化为多个简单任务。任务分解可以在模型中提示，由特定任务的指令指导，或受人类输入的影响。
```

让我们解析 LCEL 以了解其中的运作原理。

首先，这些组件（`retriever`、`prompt`、`llm` 等）都是[Ruunable](/docs/concepts#langchain-expression-language)的实例。这意味着它们实现了相同的方法，例如同步和异步的`.invoke`、`.stream`或`.batch`，这使它们更容易连接在一起。它们可以通过`|`运算符连接成[RuunableSequence](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableSequence.html)——另一个 Runnable。

当遇到`|`运算符时，LangChain 会自动将某些对象转换为 Runnable。在这里，`format_docs`被转换为[RuunableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)，而带有`"context"`和`"question"`的字典被转换为[RuunableParallel](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableParallel.html)。细节不如更重要的一点，即每个对象都是 Runnable。

让我们追踪输入问题是如何通过上述 Runnable 流动的。

正如我们上面看到的，`prompt`的输入预期是一个带有键`"context"`和`"question"`的字典。因此，此链的第一个元素构建了将从输入问题计算出这两个值的 Runnable：

- `retriever | format_docs` 将问题通过 retriever，生成[Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)对象，然后通过`format_docs`生成字符串；

- `RunnablePassthrough()` 无修改地传递输入问题。

也就是说，如果您构建

```python
chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
)
```

然后 `chain.invoke(question)` 将构建一个格式化的提示，准备进行推理。（注意：在使用 LCEL 进行开发时，可以使用这样的子链进行测试。）

链的最后几步是 `llm`，它运行推理，以及 `StrOutputParser()`，它只是从 LLM 的输出消息中提取字符串内容。

您可以通过链的 [LangSmith 跟踪](https://smith.langchain.com/public/1799e8db-8a6d-4eb2-84d5-46e8d7d5a99b/r) 分析该链的各个步骤。

### 内置链

如果需要，LangChain 包括了实现上述 LCEL 的便利函数。我们组合了两个函数：

- [create_stuff_documents_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.combine_documents.stuff.create_stuff_documents_chain.html) 指定了如何将检索到的上下文输入到提示和 LLM 中。在这种情况下，我们将上下文的内容“stuff”到提示中，即不进行任何摘要或其他处理，它基本上实现了我们上面的 `rag_chain`，输入键为 `context` 和 `input`-- 它使用检索到的上下文和查询生成答案。

- [create_retrieval_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval.create_retrieval_chain.html) 添加了检索步骤，并将检索到的上下文通过链传播，将其与最终答案一起提供。它的输入键为 `input`，在其输出中包括 `input`、`context` 和 `answer`。

```python
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate
system_prompt = (
    "您是一个用于问答任务的助手。"
    "使用以下检索到的上下文来回答问题。如果您不知道答案，请说不知道。最多使用三个句子，并保持答案简洁。"
    "\n\n"
    "{context}"
)
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{input}"),
    ]
)
question_answer_chain = create_stuff_documents_chain(llm, prompt)
rag_chain = create_retrieval_chain(retriever, question_answer_chain)
response = rag_chain.invoke({"input": "什么是任务分解？"})
print(response["answer"])
```

```output
任务分解是将复杂任务分解为较小和简单的步骤的过程。在这些任务上，使用了 Chain of Thought (CoT) 和 Tree of Thoughts 等技术来提高模型的性能。CoT 方法指导模型逐步思考，将困难的任务分解为可管理的任务，而 Tree of Thoughts 在每个步骤上探索多个推理可能性，创建了一个思维树结构。
```

#### 返回来源

在问答应用程序中，通常需要向用户显示用于生成答案的来源。LangChain 的内置 `create_retrieval_chain` 将检索到的源文档通过 `"context"` 键传播到输出中：

```python
for document in response["context"]:
    print(document)
    print()
```

```output
page_content='图1. LLM 驱动的自主代理系统概览。\n组件一：规划#\n复杂任务通常涉及许多步骤。代理需要知道这些步骤并提前规划。\n任务分解#\nChain of thought (CoT; Wei et al. 2022) 已成为增强模型在复杂任务上性能的标准提示技术。模型被指导“逐步思考”，利用更多的测试时间计算将困难任务分解为较小和简单的步骤。CoT 将大任务转化为多个可管理的任务，并为模型的思考过程提供了一种解释。' metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}
page_content='图1. LLM 驱动的自主代理系统概览。\n组件一：规划#\n复杂任务通常涉及许多步骤。代理需要知道这些步骤并提前规划。\n任务分解#\nChain of thought (CoT; Wei et al. 2022) 已成为增强模型在复杂任务上性能的标准提示技术。模型被指导“逐步思考”，利用更多的测试时间计算将困难任务分解为较小和简单的步骤。CoT 将大任务转化为多个可管理的任务，并为模型的思考过程提供了一种解释。' metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/', 'start_index': 1585}
page_content='Tree of Thoughts (Yao et al. 2023) 通过在每个步骤上探索多个推理可能性来扩展 CoT。它首先将问题分解为多个思考步骤，并在每个步骤上生成多个思考，创建了一个树形结构。搜索过程可以是 BFS（广度优先搜索）或 DFS（深度优先搜索），每个状态通过分类器（通过提示）或多数表决进行评估。\n任务分解可以通过以下方式进行：（1）LLM 通过简单的提示，如“XYZ 的步骤。\\n1.”、“实现 XYZ 的子目标是什么？”（2）使用任务特定的指令；例如，写小说时使用“撰写故事大纲。”，或者（3）使用人类输入。' metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/', 'start_index': 2192}
page_content='Tree of Thoughts (Yao et al. 2023) 通过在每个步骤上探索多个推理可能性来扩展 CoT。它首先将问题分解为多个思考步骤，并在每个步骤上生成多个思考，创建了一个树形结构。搜索过程可以是 BFS（广度优先搜索）或 DFS（深度优先搜索），每个状态通过分类器（通过提示）或多数表决进行评估。\n任务分解可以通过以下方式进行：（1）LLM 通过简单的提示，如“XYZ 的步骤。\\n1.”、“实现 XYZ 的子目标是什么？”（2）使用任务特定的指令；例如，写小说时使用“撰写故事大纲。”，或者（3）使用人类输入。' metadata={'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/'}
page_content='资源：\n1. 用于搜索和信息收集的互联网访问。\n2. 长期记忆管理。\n3. 用于委派简单任务的 GPT-3.5 动力代理。\n4. 文件输出。\n\n性能评估：\n1. 不断回顾和分析您的行动，以确保您发挥出最佳水平。\n2. 不断对自己的整体行为进行建设性的自我批评。\n3. 反思过去的决策和策略，以完善您的方法。\n4. 每个命令都有成本，因此要聪明高效。目标是以最少的步骤完成任务。' metadata={'source': 'https://lilianweng.github.io/posts
```

### 深入了解

#### 选择模型

`ChatModel`: 一个由LLM支持的聊天模型。接收一系列消息并返回一条消息。

- [文档](/docs/how_to#chat-models)

- [集成](/docs/integrations/chat/): 可选择的25个以上的集成。

- [接口](https://api.python.langchain.com/en/latest/language_models/langchain_core.language_models.chat_models.BaseChatModel.html): 基础接口的API参考。

`LLM`: 一个文本输入文本输出的LLM。接收一个字符串并返回一个字符串。

- [文档](/docs/how_to#llms)

- [集成](/docs/integrations/llms): 可选择的75个以上的集成。

- [接口](https://api.python.langchain.com/en/latest/language_models/langchain_core.language_models.llms.BaseLLM.html): 基础接口的API参考。

查看一个关于本地运行模型的RAG指南

[这里](/docs/tutorials/local_rag)。

#### 自定义提示

如上所示，我们可以从提示中心加载提示（例如[这个RAG提示](https://smith.langchain.com/hub/rlm/rag-prompt)）。提示也可以很容易地进行定制：

```python
from langchain_core.prompts import PromptTemplate
template = """使用以下上下文片段来回答最后的问题。
如果你不知道答案，只需说你不知道，不要试图凭空编造答案。
最多使用三个句子，保持答案尽可能简洁。
在回答结束时始终说“谢谢你的提问！”。
{context}
问题：{question}
有用的回答:"""
custom_rag_prompt = PromptTemplate.from_template(template)
rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | custom_rag_prompt
    | llm
    | StrOutputParser()
)
rag_chain.invoke("什么是任务分解？")
```

```output
'任务分解是将复杂任务分解为更小、更易管理的部分的过程。像思维链（CoT）和思维树这样的技术允许代理人“逐步思考”和探索多种推理可能性。这一过程可以由语言模型通过简单提示、任务特定说明或人类输入来执行。谢谢你的提问！'
```

查看[LangSmith 追踪](https://smith.langchain.com/public/da23c4d8-3b33-47fd-84df-a3a582eedf84/r)

## 下一步

我们已经介绍了构建基本问答应用程序的步骤：

- 使用[文档加载器](/docs/concepts#document-loaders)加载数据

- 使用[文本分割器](/docs/concepts#text-splitters)对索引数据进行分块，以便模型更容易使用

- [嵌入数据](/docs/concepts#embedding-models)并将数据存储在[向量存储](/docs/how_to/vectorstores)中

- [检索](/docs/concepts#retrievers)以前存储的块以回答传入的问题

- 使用检索到的块作为上下文生成答案

在上述各节中，有许多功能、集成和扩展可以探索。除了上述**深入了解**来源之外，下一步可以包括：

- [返回来源](/docs/how_to/qa_sources): 学习如何返回源文件

- [流式处理](/docs/how_to/streaming):学习如何流式处理输出和中间步骤

- [添加聊天记录](/docs/how_to/message_history):

  学习如何将聊天记录添加到您的应用程序中

