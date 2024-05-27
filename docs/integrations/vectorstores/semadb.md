# SemaDB

> [SemaDB](https://www.semafind.com/products/semadb) 是由 [SemaFind](https://www.semafind.com) 提供的一个无需烦恼的向量相似度数据库，用于构建人工智能应用程序。托管的 `SemaDB Cloud` 提供了一个无需烦恼的开发者体验，让您可以立即开始使用。

API 的完整文档以及示例和交互式演示可在 [RapidAPI](https://rapidapi.com/semafind-semadb/api/semadb) 上找到。

这个笔记本演示了如何使用 `SemaDB Cloud` 向量存储。

## 加载文档嵌入

为了在本地运行，我们使用了 [Sentence Transformers](https://www.sbert.net/)，这是常用于嵌入句子的工具。您可以使用 LangChain 提供的任何嵌入模型。

```python
%pip install --upgrade --quiet  sentence_transformers
```

```python
from langchain_huggingface import HuggingFaceEmbeddings
embeddings = HuggingFaceEmbeddings()
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=400, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
print(len(docs))
```

```output
114
```

## 连接到 SemaDB

SemaDB Cloud 使用 [RapidAPI keys](https://rapidapi.com/semafind-semadb/api/semadb) 进行身份验证。您可以通过创建免费的 RapidAPI 帐户来获取您自己的 API 密钥。

```python
import getpass
import os
os.environ["SEMADB_API_KEY"] = getpass.getpass("SemaDB API Key:")
```

```output
SemaDB API Key: ········
```

```python
from langchain_community.vectorstores import SemaDB
from langchain_community.vectorstores.utils import DistanceStrategy
```

SemaDB 向量存储的参数直接反映了 API：

- "mycollection"：是我们将存储这些向量的集合名称。

- 768：是向量的维度。在我们的情况下，句子转换器嵌入产生 768 维向量。

- API_KEY：是您的 RapidAPI 密钥。

- embeddings：对应于如何生成文档、文本和查询的嵌入。

- DistanceStrategy：是使用的距离度量。如果使用 COSINE，则包装器会自动归一化向量。

```python
db = SemaDB("mycollection", 768, embeddings, DistanceStrategy.COSINE)
# 如果是第一次运行，创建集合。如果集合已经存在，这将失败。
db.create_collection()
```

```output
True
```

SemaDB 向量存储包装器将文档文本作为点元数据添加以供以后收集。*不建议*存储大块文本。如果要索引大型集合，我们建议存储文档的引用，例如外部 ID。

```python
db.add_documents(docs)[:2]
```

## 相似度搜索

我们使用默认的 LangChain 相似度搜索接口来搜索最相似的句子。

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
print(docs[0].page_content)
```

```output
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

```python
docs = db.similarity_search_with_score(query)
docs[0]
```

```output
(Document(page_content='And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt', 'text': 'And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.'}),
 0.42369342)
```

## 清理

您可以删除集合以删除所有数据。

```python
db.delete_collection()
```

```output
True
```