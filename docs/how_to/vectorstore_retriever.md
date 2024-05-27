---

sidebar_position: 0

---

# 如何使用向量存储作为检索器

向量存储检索器是使用向量存储来检索文档的检索器。它是对向量存储类的轻量级封装，使其符合检索器接口。它使用向量存储实现的搜索方法，如相似性搜索和最大边际相关性（MMR），来查询向量存储中的文本。

在本指南中，我们将介绍：

1. 如何从向量存储中实例化检索器；

2. 如何为检索器指定搜索类型；

3. 如何指定额外的搜索参数，如阈值分数和前 k 个结果。

## 从向量存储创建检索器

您可以使用其 [.as_retriever](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html#langchain_core.vectorstores.VectorStore.as_retriever) 方法从向量存储构建检索器。让我们通过一个示例来了解。

首先，我们实例化一个向量存储。我们将使用一个内存中的 [FAISS](https://api.python.langchain.com/en/latest/vectorstores/langchain_community.vectorstores.faiss.FAISS.html) 向量存储：

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(texts, embeddings)
```

然后我们实例化一个检索器：

```python
retriever = vectorstore.as_retriever()
```

这将创建一个检索器（具体来说是 [VectorStoreRetriever](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStoreRetriever.html)），我们可以像平常一样使用它：

```python
docs = retriever.invoke("总统对凯坦吉·布朗·杰克逊有什么看法？")
```

## 最大边际相关性检索

默认情况下，向量存储检索器使用相似性搜索。如果底层向量存储支持最大边际相关性搜索，您可以将其指定为搜索类型。

这实际上指定了底层向量存储上使用的方法（例如 `similarity_search`、`max_marginal_relevance_search` 等）。

```python
retriever = vectorstore.as_retriever(search_type="mmr")
```

```python
docs = retriever.invoke("总统对凯坦吉·布朗·杰克逊有什么看法？")
```

## 传递搜索参数

我们可以使用 `search_kwargs` 将参数传递给底层向量存储的搜索方法。

### 相似性分数阈值检索

例如，我们可以设置相似性分数阈值，并仅返回分数高于该阈值的文档。

```python
retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold", search_kwargs={"score_threshold": 0.5}
)
```

```python
docs = retriever.invoke("总统对凯坦吉·布朗·杰克逊有什么看法？")
```

### 指定前 k 个结果

我们还可以限制检索器返回的文档数量 `k`。

```python
retriever = vectorstore.as_retriever(search_kwargs={"k": 1})
```

```python
docs = retriever.invoke("总统对凯坦吉·布朗·杰克逊有什么看法？")
len(docs)
```

```output
1
```