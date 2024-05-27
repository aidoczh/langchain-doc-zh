# Faiss

[Facebook AI Similarity Search (Faiss)](https://engineering.fb.com/2017/03/29/data-infrastructure/faiss-a-library-for-efficient-similarity-search/) 是一个用于高效相似度搜索和密集向量聚类的库。它包含了在任意大小的向量集合中进行搜索的算法，甚至可以处理可能无法完全放入内存的向量集合。它还包含用于评估和参数调整的支持代码。

[Faiss documentation](https://faiss.ai/)。

这个笔记本展示了如何使用与 `FAISS` 向量数据库相关的功能。它将展示特定于此集成的功能。在学习完这些内容后，探索[相关的用例页面](/docs/how_to#qa-with-rag)可能会很有帮助，以了解如何将这个向量存储作为更大链条的一部分来使用。

## 设置

该集成位于 `langchain-community` 包中。我们还需要安装 `faiss` 包本身。我们还将使用 OpenAI 进行嵌入，因此需要安装这些要求。我们可以使用以下命令进行安装：

```bash
pip install -U langchain-community faiss-cpu langchain-openai tiktoken
```

请注意，如果您想使用启用了 GPU 的版本，也可以安装 `faiss-gpu`。

由于我们将使用 OpenAI，您将需要一个 OpenAI API 密钥。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
```

设置 [LangSmith](https://smith.langchain.com/) 以获得最佳的可观测性也会很有帮助（但不是必需的）。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 导入

在这里，我们将文档导入到向量存储中。

```python
# 如果您需要使用没有 AVX2 优化的 FAISS 进行初始化，请取消下面一行的注释
# os.environ['FAISS_NO_AVX2'] = '1'
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
db = FAISS.from_documents(docs, embeddings)
print(db.index.ntotal)
```
```output
42
```

## 查询

现在，我们可以查询向量存储。有几种方法可以做到这一点。最常见的方法是使用 `similarity_search`。

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
```
```python
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 作为检索器

我们还可以将向量存储转换为 [Retriever](/docs/how_to#retrievers) 类。这使我们能够轻松地在其他 LangChain 方法中使用它，这些方法主要用于检索器。

```python
retriever = db.as_retriever()
docs = retriever.invoke(query)
```
```python
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 
One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 带分数的相似度搜索

还有一些 FAISS 特定的方法。其中之一是 `similarity_search_with_score`，它允许您返回文档以及查询与它们之间的距离分数。返回的距离分数是 L2 距离。因此，得分越低越好。

```python
docs_and_scores = db.similarity_search_with_score(query)
今晚，我呼吁参议院：通过《自由选举法案》（Freedom to Vote Act）。通过《约翰·刘易斯选举权法案》（John Lewis Voting Rights Act）。还有，顺便通过《揭露法案》（Disclose Act），这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个终生致力于为这个国家服务的人致敬：司法部长史蒂芬·布雷耶（Stephen Breyer）——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的大法官。布雷耶司法部长，感谢您的服务。
总统最严肃的宪法责任之一就是提名某人担任美国最高法院的法官。
而我在4天前就做到了，当时我提名了联邦上诉法院法官凯坦吉·布朗·杰克逊（Ketanji Brown Jackson）。她是我们国家最顶尖的法律专家之一，将继续延续布雷耶司法部长的卓越传统。[20]（资料来源：../../how_to/state_of_the_union.txt）
此外，还可以使用`similarity_search_by_vector`来搜索与给定嵌入向量相似的文档，该函数接受一个嵌入向量作为参数，而不是字符串。
```python

embedding_vector = embeddings.embed_query(query)

docs_and_scores = db.similarity_search_by_vector(embedding_vector)

```
## 保存和加载
您还可以保存和加载 FAISS 索引。这样做很有用，因为您不必每次使用时都重新创建它。
```python
db.save_local("faiss_index")
new_db = FAISS.load_local("faiss_index", embeddings)
docs = new_db.similarity_search(query)
```
```python
docs[0]
```
```output

Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../../state_of_the_union.txt'})

```
# 序列化和反序列化为字节
您可以使用以下函数将 FAISS 索引进行 pickle。如果您使用的是大小为 90 MB 的嵌入模型（如 sentence-transformers/all-MiniLM-L6-v2 或其他模型），则生成的 pickle 大小将超过 90 MB。模型的大小也包含在总体大小中。为了克服这一问题，可以使用以下函数。这些函数仅序列化 FAISS 索引，大小会小得多。如果您希望将索引存储在数据库中，比如 SQL，这将会很有帮助。
```python
from langchain_huggingface import HuggingFaceEmbeddings
pkl = db.serialize_to_bytes()  # 序列化 faiss
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
db = FAISS.deserialize_from_bytes(embeddings=embeddings, serialized=pkl)  # 加载索引
```
## 合并
您还可以合并两个 FAISS 向量存储。
```python
db1 = FAISS.from_texts(["foo"], embeddings)
db2 = FAISS.from_texts(["bar"], embeddings)
db1.docstore._dict
```
```python
db2.docstore._dict
```
```output

{'807e0c63-13f6-4070-9774-5c6f0fbb9866': Document(page_content='bar', metadata={})}

```
```python
db1.merge_from(db2)
```
```python
db1.docstore._dict
```
```output

{'068c473b-d420-487a-806b-fb0ccea7f711': Document(page_content='foo', metadata={}),

 '807e0c63-13f6-4070-9774-5c6f0fbb9866': Document(page_content='bar', metadata={})}

```
## 带过滤的相似性搜索
FAISS 向量存储还可以支持过滤，因为 FAISS 不原生支持过滤，所以我们必须手动进行。首先获取比 `k` 更多的结果，然后进行过滤。此过滤器可以是一个接受元数据字典并返回布尔值的可调用对象，也可以是一个元数据字典，其中每个缺失的键都被忽略，每个存在的键必须在一个值列表中。在调用任何搜索方法时，还可以设置 `fetch_k` 参数，以设置在过滤之前要获取多少个文档。以下是一个小例子：
```python
from langchain_core.documents import Document
list_of_documents = [
    Document(page_content="foo", metadata=dict(page=1)),
    Document(page_content="bar", metadata=dict(page=1)),
    Document(page_content="foo", metadata=dict(page=2)),
    Document(page_content="barbar", metadata=dict(page=2)),
    Document(page_content="foo", metadata=dict(page=3)),
    Document(page_content="bar burr", metadata=dict(page=3)),
```
```python
Document(page_content="foo", metadata=dict(page=4)),
    Document(page_content="bar bruh", metadata=dict(page=4)),
]
db = FAISS.from_documents(list_of_documents, embeddings)
results_with_scores = db.similarity_search_with_score("foo")
for doc, score in results_with_scores:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}, Score: {score}")
```
```output

Content: foo, Metadata: {'page': 1}, Score: 5.159960813797904e-15

Content: foo, Metadata: {'page': 2}, Score: 5.159960813797904e-15

Content: foo, Metadata: {'page': 3}, Score: 5.159960813797904e-15

Content: foo, Metadata: {'page': 4}, Score: 5.159960813797904e-15

```
现在我们进行相同的查询，但只筛选 `page = 1` 的结果。
```python
results_with_scores = db.similarity_search_with_score("foo", filter=dict(page=1))
# 或者使用可调用对象：
# results_with_scores = db.similarity_search_with_score("foo", filter=lambda d: d["page"] == 1)
for doc, score in results_with_scores:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}, Score: {score}")
```
```output

Content: foo, Metadata: {'page': 1}, Score: 5.159960813797904e-15

Content: bar, Metadata: {'page': 1}, Score: 0.3131446838378906

```
同样的操作也适用于 `max_marginal_relevance_search`。
```python
results = db.max_marginal_relevance_search("foo", filter=dict(page=1))
for doc in results:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}")
```
```output

Content: foo, Metadata: {'page': 1}

Content: bar, Metadata: {'page': 1}

```
以下是在调用 `similarity_search` 时设置 `fetch_k` 参数的示例。通常情况下，您希望 `fetch_k` 参数 >> `k` 参数。这是因为 `fetch_k` 参数是在过滤之前将被获取的文档数量。如果将 `fetch_k` 设置为一个较小的数字，可能无法获取足够的文档进行过滤。
```python
results = db.similarity_search("foo", filter=dict(page=1), k=1, fetch_k=4)
for doc in results:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}")
```
```output

Content: foo, Metadata: {'page': 1}

```
## 删除
您也可以从向量存储中删除记录。在下面的示例中，`db.index_to_docstore_id` 表示一个具有 FAISS 索引元素的字典。
```python
print("删除前数量:", db.index.ntotal)
db.delete([db.index_to_docstore_id[0]])
print("删除后数量:", db.index.ntotal)
```
```output

删除前数量: 8

删除后数量: 7

```