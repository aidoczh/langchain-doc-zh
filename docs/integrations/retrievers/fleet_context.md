# Fleet AI Context

[Fleet AI Context](https://www.fleet.so/context) 是一个包含了最受欢迎且开放的 Python 库及其文档的高质量嵌入数据集。

Fleet AI 团队的使命是嵌入全球最重要的数据。他们已经开始嵌入前 1200 个 Python 库，以便使用最新知识生成代码。他们很慷慨地分享了他们对 [LangChain 文档](/docs/introduction) 和 [API 参考](https://api.python.langchain.com/en/latest/api_reference.html) 的嵌入。

让我们看看如何使用这些嵌入来支持文档检索系统，最终实现一个简单的代码生成链！

```python
%pip install --upgrade --quiet  langchain fleet-context langchain-openai pandas faiss-cpu # faiss-gpu for CUDA supported GPU
```

```python
from operator import itemgetter
from typing import Any, Optional, Type
import pandas as pd
from langchain.retrievers import MultiVectorRetriever
from langchain_community.vectorstores import FAISS
from langchain_core.documents import Document
from langchain_core.stores import BaseStore
from langchain_core.vectorstores import VectorStore
from langchain_openai import OpenAIEmbeddings
def load_fleet_retriever(
    df: pd.DataFrame,
    *,
    vectorstore_cls: Type[VectorStore] = FAISS,
    docstore: Optional[BaseStore] = None,
    **kwargs: Any,
):
    vectorstore = _populate_vectorstore(df, vectorstore_cls)
    if docstore is None:
        return vectorstore.as_retriever(**kwargs)
    else:
        _populate_docstore(df, docstore)
        return MultiVectorRetriever(
            vectorstore=vectorstore, docstore=docstore, id_key="parent", **kwargs
        )
def _populate_vectorstore(
    df: pd.DataFrame,
    vectorstore_cls: Type[VectorStore],
) -> VectorStore:
    if not hasattr(vectorstore_cls, "from_embeddings"):
        raise ValueError(
            f"Incompatible vector store class {vectorstore_cls}."
            "Must implement `from_embeddings` class method."
        )
    texts_embeddings = []
    metadatas = []
    for _, row in df.iterrows():
        texts_embeddings.append((row.metadata["text"], row["dense_embeddings"]))
        metadatas.append(row.metadata)
    return vectorstore_cls.from_embeddings(
        texts_embeddings,
        OpenAIEmbeddings(model="text-embedding-ada-002"),
        metadatas=metadatas,
    )
def _populate_docstore(df: pd.DataFrame, docstore: BaseStore) -> None:
    parent_docs = []
    df = df.copy()
    df["parent"] = df.metadata.apply(itemgetter("parent"))
    for parent_id, group in df.groupby("parent"):
        sorted_group = group.iloc[
            group.metadata.apply(itemgetter("section_index")).argsort()
        ]
        text = "".join(sorted_group.metadata.apply(itemgetter("text")))
        metadata = {
            k: sorted_group.iloc[0].metadata[k] for k in ("title", "type", "url")
        }
        text = metadata["title"] + "\n" + text
        metadata["id"] = parent_id
        parent_docs.append(Document(page_content=text, metadata=metadata))
    docstore.mset(((d.metadata["id"], d) for d in parent_docs))
```

## 检索器块

作为嵌入过程的一部分，Fleet AI 团队首先对长文档进行了分块，然后再进行嵌入。这意味着向量对应于 LangChain 文档页面的部分，而不是整个页面。默认情况下，当我们从这些嵌入中启动检索器时，我们将检索这些嵌入的块。

我们将使用 Fleet Context 的 `download_embeddings()` 来获取 Langchain 的文档嵌入。您可以在 https://fleet.so/context 查看所有支持的库的文档。

```python
from context import download_embeddings
df = download_embeddings("langchain")
vecstore_retriever = load_fleet_retriever(df)
```

```python
vecstore_retriever.invoke("How does the multi vector retriever work")
```

## 其他软件包

您可以从 [此 Dropbox 链接](https://www.dropbox.com/scl/fo/54t2e7fogtixo58pnlyub/h?rlkey=tne16wkssgf01jor0p1iqg6p9&dl=0) 下载并使用其他嵌入。

## 检索父文档

Fleet AI 提供的嵌入包含了指示哪些嵌入块对应于相同原始文档页面的元数据。如果需要，我们可以使用这些信息来检索整个父文档，而不仅仅是嵌入的块。在底层，我们将使用 MultiVectorRetriever 和 BaseStore 对象来搜索相关的块，然后将它们映射到它们的父文档。

```python
from langchain.storage import InMemoryStore
parent_retriever = load_fleet_retriever(
    "https://www.dropbox.com/scl/fi/4rescpkrg9970s3huz47l/libraries_langchain_release.parquet?rlkey=283knw4wamezfwiidgpgptkep&dl=1",
    docstore=InMemoryStore(),
)
```

```python
parent_retriever.invoke("How does the multi vector retriever work")
```

## 将其放入链中

让我们尝试在一个简单的链中使用我们的检索系统！

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            """你是一位非常熟悉 Python 的优秀软件工程师。给定一个关于名为 LangChain 的新 Python 库以及 LangChain 文档部分的用户问题或请求，回答问题或生成所需的代码。你的回答必须准确，尽可能包含代码，并假设关于 LangChain 的任何信息都没有在 LangChain 文档中明确说明。如果所需信息不可用，只需说明即可。
LangChain 文档
------------------
{context}""",
        ),
        ("human", "{question}"),
    ]
)
model = ChatOpenAI(model="gpt-3.5-turbo-16k")
chain = (
    {
        "question": RunnablePassthrough(),
        "context": parent_retriever
        | (lambda docs: "\n\n".join(d.page_content for d in docs)),
    }
    | prompt
    | model
    | StrOutputParser()
)
```

```python
for chunk in chain.invoke(
    "如何创建一个 FAISS 向量存储检索器，每次搜索查询返回 10 个文档"
):
    print(chunk, end="", flush=True)
```
