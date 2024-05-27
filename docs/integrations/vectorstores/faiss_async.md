# Faiss（异步）

[Facebook AI Similarity Search (Faiss)](https://engineering.fb.com/2017/03/29/data-infrastructure/faiss-a-library-for-efficient-similarity-search/) 是一个用于高效相似性搜索和密集向量聚类的库。它包含了在任何大小的向量集合中进行搜索的算法，甚至可以处理可能无法放入内存的向量集合。它还包含了用于评估和参数调整的支持代码。

[Faiss 文档](https://faiss.ai/)。

这个笔记本展示了如何使用与 `FAISS` 向量数据库相关的功能，使用了 `asyncio`。

LangChain 实现了同步和异步向量存储功能。

查看 `同步` 版本[这里](/docs/integrations/vectorstores/faiss)。

```python
%pip install --upgrade --quiet  faiss-gpu # 适用于 CUDA 7.5+ 支持的 GPU。
# 或者
%pip install --upgrade --quiet  faiss-cpu # 适用于 CPU 安装
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
# 如果需要使用没有 AVX2 优化的 FAISS 进行初始化，请取消下一行的注释
# os.environ['FAISS_NO_AVX2'] = '1'
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../../extras/modules/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
db = await FAISS.afrom_documents(docs, embeddings)
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = await db.asimilarity_search(query)
print(docs[0].page_content)
```

## 带分数的相似性搜索

有一些 FAISS 特定的方法。其中之一是 `similarity_search_with_score`，它允许您不仅返回文档，还返回查询与它们之间的距离分数。返回的距离分数是 L2 距离。因此，得分越低越好。

```python
docs_and_scores = await db.asimilarity_search_with_score(query)
docs_and_scores[0]
```

还可以使用 `similarity_search_by_vector` 搜索与给定嵌入向量相似的文档，该方法接受嵌入向量作为参数，而不是字符串。

```python
embedding_vector = await embeddings.aembed_query(query)
docs_and_scores = await db.asimilarity_search_by_vector(embedding_vector)
```

## 保存和加载

您还可以保存和加载 FAISS 索引。这样做是很有用的，这样您就不必每次使用时都重新创建它。

```python
db.save_local("faiss_index")
new_db = FAISS.load_local("faiss_index", embeddings, asynchronous=True)
docs = await new_db.asimilarity_search(query)
docs[0]
```

# 序列化和反序列化为字节

您可以使用这些函数对 FAISS 索引进行 pickle。如果您使用的是 90 MB 的嵌入模型（如 sentence-transformers/all-MiniLM-L6-v2 或其他模型），则生成的 pickle 大小将超过 90 MB。模型的大小也包含在总体大小中。为了克服这一点，可以使用下面的函数。这些函数只序列化 FAISS 索引，大小会小得多。如果您希望将索引存储在诸如 SQL 数据库之类的数据库中，这可能会有所帮助。

```python
from langchain_huggingface import HuggingFaceEmbeddings
pkl = db.serialize_to_bytes()  # 序列化 faiss 索引
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
db = FAISS.deserialize_from_bytes(
    embeddings=embeddings, serialized=pkl, asynchronous=True
)  # 加载索引
```

## 合并

您还可以合并两个 FAISS 向量存储

```python
db1 = await FAISS.afrom_texts(["foo"], embeddings)
db2 = await FAISS.afrom_texts(["bar"], embeddings)
```

```python
db1.docstore._dict
```

```output
{'8164a453-9643-4959-87f7-9ba79f9e8fb0': Document(page_content='foo')}
```

```python
db2.docstore._dict
```

```output
{'4fbcf8a2-e80f-4f65-9308-2f4cb27cb6e7': Document(page_content='bar')}
```

```python
db1.merge_from(db2)
```

```python
db1.docstore._dict
```

```output
{'8164a453-9643-4959-87f7-9ba79f9e8fb0': Document(page_content='foo'),
 '4fbcf8a2-e80f-4f65-9308-2f4cb27cb6e7': Document(page_content='bar')}
```

## 带过滤的相似性搜索

FAISS 向量存储还可以支持过滤，因为 FAISS 不原生支持过滤，所以我们必须手动进行。首先获取比 `k` 更多的结果，然后进行过滤。您可以根据元数据对文档进行过滤。在调用任何搜索方法时，您还可以设置 `fetch_k` 参数，以设置在过滤之前要获取多少文档。以下是一个小例子：

```python
from langchain_core.documents import Document
list_of_documents = [
    Document(page_content="foo", metadata=dict(page=1)),
    Document(page_content="bar", metadata=dict(page=1)),
    Document(page_content="foo", metadata=dict(page=2)),
    Document(page_content="barbar", metadata=dict(page=2)),
    Document(page_content="foo", metadata=dict(page=3)),
    Document(page_content="bar burr", metadata=dict(page=3)),
    Document(page_content="foo", metadata=dict(page=4)),
    Document(page_content="bar bruh", metadata=dict(page=4)),
]
db = FAISS.from_documents(list_of_documents, embeddings)
results_with_scores = db.similarity_search_with_score("foo")
for doc, score in results_with_scores:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}, Score: {score}")
```

```output
内容：foo，元数据：{'page': 1}，得分：5.159960813797904e-15
内容：foo，元数据：{'page': 2}，得分：5.159960813797904e-15
内容：foo，元数据：{'page': 3}，得分：5.159960813797904e-15
内容：foo，元数据：{'page': 4}，得分：5.159960813797904e-15
```

现在我们进行相同的查询调用，但只过滤 `page = 1` 

```python
results_with_scores = await db.asimilarity_search_with_score("foo", filter=dict(page=1))
for doc, score in results_with_scores:
    print(f"内容：{doc.page_content}，元数据：{doc.metadata}，得分：{score}")
```

```output
内容：foo，元数据：{'page': 1}，得分：5.159960813797904e-15
内容：bar，元数据：{'page': 1}，得分：0.3131446838378906
```

同样的操作也可以用于 `max_marginal_relevance_search`。

```python
results = await db.amax_marginal_relevance_search("foo", filter=dict(page=1))
for doc in results:
    print(f"内容：{doc.page_content}，元数据：{doc.metadata}")
```

```output
内容：foo，元数据：{'page': 1}
内容：bar，元数据：{'page': 1}
```

这是在调用 `similarity_search` 时如何设置 `fetch_k` 参数的示例。通常，您希望 `fetch_k` 参数 >> `k` 参数。这是因为 `fetch_k` 参数是在过滤之前将被获取的文档数量。如果您将 `fetch_k` 设置为一个较低的数字，您可能无法获取足够的文档进行过滤。

```python
results = await db.asimilarity_search("foo", filter=dict(page=1), k=1, fetch_k=4)
for doc in results:
    print(f"内容：{doc.page_content}，元数据：{doc.metadata}")
```

```output
内容：foo，元数据：{'page': 1}
```

## 删除

您也可以删除 id。请注意，要删除的 id 应该是文档存储中的 id。

```python
db.delete([db.index_to_docstore_id[0]])
```

```output
True
```

```python
# 现在已经丢失
0 in db.index_to_docstore_id
```

```output
False
```