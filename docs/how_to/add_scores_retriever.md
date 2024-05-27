# 如何为检索结果添加分数

检索器将返回一系列[文档](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)对象，默认情况下不包含有关检索过程的任何信息（例如与查询的相似度分数）。下面我们将演示如何将检索分数添加到文档的`.metadata`中：

1. 从[向量存储检索器](/docs/how_to/vectorstore_retriever)；

2. 从高阶 LangChain 检索器，例如[SelfQueryRetriever](/docs/how_to/self_query)或[MultiVectorRetriever](/docs/how_to/multi_vector)。

对于（1），我们将在相应的向量存储周围实现一个简短的包装函数。对于（2），我们将更新相应类的一个方法。

## 创建向量存储

首先，我们使用一些数据填充向量存储。我们将使用[PineconeVectorStore](https://api.python.langchain.com/en/latest/vectorstores/langchain_pinecone.vectorstores.PineconeVectorStore.html)，但本指南与任何实现了`.similarity_search_with_score`方法的 LangChain 向量存储兼容。

```python
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings
from langchain_pinecone import PineconeVectorStore
docs = [
    Document(
        page_content="一群科学家带回恐龙，然后混乱不堪",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在一个梦中的梦中的梦中...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重复了这个想法",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通大小的女性非常纯真，一些男性对她们倾慕",
        metadata={"year": 2019, "director": "格蕾塔·葛韦格", "rating": 8.3},
    ),
    Document(
        page_content="玩具活了起来，并且玩得很开心",
        metadata={"year": 1995, "genre": "动画"},
    ),
    Document(
        page_content="三个人走进区域，三个人走出区域",
        metadata={"year": 1979, "director": "安德烈·塔尔科夫斯基", "genre": "惊悚", "rating": 9.9},
    ),
]
vectorstore = PineconeVectorStore.from_documents(
    docs, index_name="sample", embedding=OpenAIEmbeddings()
)
```

## 检索器

为了从向量存储检索器中获取分数，我们将在一个简短的函数中包装底层向量存储的`.similarity_search_with_score`方法，将分数打包到相关文档的元数据中。

我们在函数中添加了`@chain`装饰器，以创建一个[Runnable](/docs/concepts/#langchain-expression-language)，可以类似于典型的检索器使用。

```python
from typing import List
from langchain_core.documents import Document
from langchain_core.runnables import chain
@chain
def retriever(query: str) -> List[Document]:
    docs, scores = zip(*vectorstore.similarity_search_with_score(query))
    for doc, score in zip(docs, scores):
        doc.metadata["score"] = score
    return docs
```

```python
result = retriever.invoke("恐龙")
result
```

```output
(Document(page_content='一群科学家带回恐龙，然后混乱不堪', metadata={'genre': '科幻', 'rating': 7.7, 'year': 1993.0, 'score': 0.84429127}),
 Document(page_content='玩具活了起来，并且玩得很开心', metadata={'genre': '动画', 'year': 1995.0, 'score': 0.792038262}),
 Document(page_content='三个人走进区域，三个人走出区域', metadata={'director': '安德烈·塔尔科夫斯基', 'genre': '惊悚', 'rating': 9.9, 'year': 1979.0, 'score': 0.751571238}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦境中，而《盗梦空间》重复了这个想法', metadata={'director': '今敏', 'rating': 8.6, 'year': 2006.0, 'score': 0.747471571}))
```

请注意，检索步骤中的相似度分数包含在上述文档的元数据中。

## SelfQueryRetriever

`SelfQueryRetriever`将使用 LLM 生成一个可能结构化的查询--例如，它可以在通常的语义相似性驱动选择的基础上构建检索的过滤器。更多详细信息请参阅[此指南](/docs/how_to/self_query)。

`SelfQueryRetriever`包括一个短（1 - 2行）的方法`_get_docs_with_query`，执行`vectorstore`搜索。我们可以子类化`SelfQueryRetriever`并覆盖此方法以传播相似度分数。

首先，根据[操作指南](/docs/how_to/self_query)，我们需要建立一些元数据来进行筛选：

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import ChatOpenAI
metadata_field_info = [
    AttributeInfo(
        name="genre",
        description="电影的类型。['science fiction', 'comedy', 'drama', 'thriller', 'romance', 'action', 'animated'] 中的一个",
        type="string",
    ),
    AttributeInfo(
        name="year",
        description="电影上映的年份",
        type="integer",
    ),
    AttributeInfo(
        name="director",
        description="电影导演的姓名",
        type="string",
    ),
    AttributeInfo(
        name="rating",
        description="电影的评分，范围为1-10",
        type="float"
    ),
]
document_content_description = "电影的简要概述"
llm = ChatOpenAI(temperature=0)
```

然后，我们覆盖`_get_docs_with_query`方法，使用底层向量存储的`similarity_search_with_score`方法：

```python
from typing import Any, Dict
class CustomSelfQueryRetriever(SelfQueryRetriever):
    def _get_docs_with_query(
        self, query: str, search_kwargs: Dict[str, Any]
    ) -> List[Document]:
        """获取文档，并添加分数信息。"""
        docs, scores = zip(
            *vectorstore.similarity_search_with_score(query, **search_kwargs)
        )
        for doc, score in zip(docs, scores):
            doc.metadata["score"] = score
        return docs
```

现在调用这个检索器将在文档元数据中包含相似度分数。请注意，保留了`SelfQueryRetriever`的底层结构化查询功能。

```python
retriever = CustomSelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
)
result = retriever.invoke("评分低于8分的恐龙电影")
result
```

```output
(Document(page_content='一群科学家复活了恐龙，混乱不断', metadata={'genre': 'science fiction', 'rating': 7.7, 'year': 1993.0, 'score': 0.84429127}),)
```

## 多向量检索器

`MultiVectorRetriever`允许您将多个向量与单个文档关联。这在许多应用程序中非常有用。例如，我们可以索引较大文档的小块并在这些块上运行检索，但在调用检索器时返回较大的“父”文档。[ParentDocumentRetriever](/docs/how_to/parent_document_retriever/)是`MultiVectorRetriever`的子类，包含方便的方法来填充向量存储以支持此功能。更多应用程序细节请参阅此[操作指南](/docs/how_to/multi_vector/)。

要通过此检索器传播相似度分数，我们可以再次创建`MultiVectorRetriever`的子类并覆盖一个方法。这次我们将覆盖`_get_relevant_documents`。

首先，我们准备一些虚假数据。我们生成虚假的“整个文档”并将它们存储在文档存储中；在这里，我们将使用简单的[InMemoryStore](https://api.python.langchain.com/en/latest/stores/langchain_core.stores.InMemoryBaseStore.html)。

```python
from langchain.storage import InMemoryStore
from langchain_text_splitters import RecursiveCharacterTextSplitter
# 用于父文档的存储层
docstore = InMemoryStore()
fake_whole_documents = [
    ("fake_id_1", Document(page_content="虚假整个文档 1")),
    ("fake_id_2", Document(page_content="虚假整个文档 2")),
]
docstore.mset(fake_whole_documents)
```

接下来，我们将一些虚假的“子文档”添加到我们的向量存储中。我们可以通过在其元数据中填充“doc_id”键将这些子文档与父文档关联。

```python
docs = [
    Document(
        page_content="来自讨论猫的较大文档的片段。",
        metadata={"doc_id": "fake_id_1"},
    ),
    Document(
        page_content="来自讨论话语的较大文档的片段。",
        metadata={"doc_id": "fake_id_1"},
    ),
    Document(
        page_content="来自讨论巧克力的较大文档的片段。",
        metadata={"doc_id": "fake_id_2"},
    ),
]
vectorstore.add_documents(docs)
```

```output
['62a85353-41ff-4346-bff7-be6c8ec2ed89',
 '5d4a0e83-4cc5-40f1-bc73-ed9cbad0ee15',
 '8c1d9a56-120f-45e4-ba70-a19cd19a38f4']
```

为了传播分数，我们创建`MultiVectorRetriever`的子类并覆盖其`_get_relevant_documents`方法。这里我们将做两个更改：

1. 我们将使用与上述相同的底层向量存储的`similarity_search_with_score`方法将相似度分数添加到相应“子文档”的元数据中；

2. 我们将在检索到的父文档的元数据中包含这些子文档的列表。这样可以展示检索中识别出的文本片段，以及它们对应的相似度分数。

```python
from collections import defaultdict
from langchain.retrievers import MultiVectorRetriever
from langchain_core.callbacks import CallbackManagerForRetrieverRun
class CustomMultiVectorRetriever(MultiVectorRetriever):
    def _get_relevant_documents(
        self, query: str, *, run_manager: CallbackManagerForRetrieverRun
    ) -> List[Document]:
        """获取与查询相关的文档。
        Args:
            query: 要查找相关文档的字符串
            run_manager: 要使用的回调处理程序
        Returns:
            相关文档的列表
        """
        results = self.vectorstore.similarity_search_with_score(
            query, **self.search_kwargs
        )
        # 将文档 ID 映射到子文档列表，将分数添加到元数据中
        id_to_doc = defaultdict(list)
        for doc, score in results:
            doc_id = doc.metadata.get("doc_id")
            if doc_id:
                doc.metadata["score"] = score
                id_to_doc[doc_id].append(doc)
        # 获取与文档 ID 对应的文档，保留子文档在元数据中
        docs = []
        for _id, sub_docs in id_to_doc.items():
            docstore_docs = self.docstore.mget([_id])
            if docstore_docs:
                if doc := docstore_docs[0]:
                    doc.metadata["sub_docs"] = sub_docs
                    docs.append(doc)
        return docs
```

调用这个检索器，我们可以看到它识别出了正确的父文档，包括与子文档相似度得分相关的片段。

```python
retriever = CustomMultiVectorRetriever(vectorstore=vectorstore, docstore=docstore)
retriever.invoke("cat")
```

```output
[Document(page_content='fake whole document 1', metadata={'sub_docs': [Document(page_content='A snippet from a larger document discussing cats.', metadata={'doc_id': 'fake_id_1', 'score': 0.831276655})]})]
```