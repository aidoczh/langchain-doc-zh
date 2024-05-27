# 如何创建一个集成检索器

`EnsembleRetriever` 接受一个检索器列表作为输入，并根据[Reciprocal Rank Fusion](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf)算法对它们的`get_relevant_documents()`方法的结果进行集成和重新排序。

通过利用不同算法的优势，`EnsembleRetriever`可以比任何单一算法获得更好的性能。

最常见的模式是将稀疏检索器（如BM25）与密集检索器（如嵌入相似度）结合起来，因为它们的优势是互补的。这也被称为“混合搜索”。稀疏检索器擅长基于关键词找到相关文档，而密集检索器擅长基于语义相似性找到相关文档。

```python
%pip install --upgrade --quiet  rank_bm25 > /dev/null
```

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
```

```python
doc_list_1 = [
    "我喜欢苹果",
    "我喜欢橙子",
    "苹果和橙子是水果",
]
# 初始化bm25检索器和faiss检索器
bm25_retriever = BM25Retriever.from_texts(
    doc_list_1, metadatas=[{"source": 1}] * len(doc_list_1)
)
bm25_retriever.k = 2
doc_list_2 = [
    "你喜欢苹果",
    "你喜欢橙子",
]
embedding = OpenAIEmbeddings()
faiss_vectorstore = FAISS.from_texts(
    doc_list_2, embedding, metadatas=[{"source": 2}] * len(doc_list_2)
)
faiss_retriever = faiss_vectorstore.as_retriever(search_kwargs={"k": 2})
# 初始化集成检索器
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, faiss_retriever], weights=[0.5, 0.5]
)
```

```python
docs = ensemble_retriever.invoke("苹果")
docs
```

```output
[Document(page_content='你喜欢苹果', metadata={'source': 2}),
 Document(page_content='我喜欢苹果', metadata={'source': 1}),
 Document(page_content='你喜欢橙子', metadata={'source': 2}),
 Document(page_content='苹果和橙子是水果', metadata={'source': 1})]
```

## 运行时配置

我们还可以在运行时配置检索器。为了做到这一点，我们需要将字段标记为可配置的。

```python
from langchain_core.runnables import ConfigurableField
```

```python
faiss_retriever = faiss_vectorstore.as_retriever(
    search_kwargs={"k": 2}
).configurable_fields(
    search_kwargs=ConfigurableField(
        id="search_kwargs_faiss",
        name="搜索参数",
        description="要使用的搜索参数",
    )
)
```

```python
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, faiss_retriever], weights=[0.5, 0.5]
)
```

```python
config = {"configurable": {"search_kwargs_faiss": {"k": 1}}}
docs = ensemble_retriever.invoke("苹果", config=config)
docs
```

请注意，这只返回了FAISS检索器的一个来源，因为我们在运行时传入了相关的配置。