# Annoy

[Annoy](https://github.com/spotify/annoy)（`Approximate Nearest Neighbors Oh Yeah`）是一个 C++ 库，带有 Python 绑定，用于搜索空间中与给定查询点接近的点。它还创建了大型的只读基于文件的数据结构，这些数据结构被映射到内存中，以便许多进程可以共享相同的数据。

这个笔记本展示了如何使用与 `Annoy` 向量数据库相关的功能。

```{note}
注意：Annoy 是只读的 - 一旦建立索引，就不能再添加任何嵌入！如果您想逐渐向您的 VectorStore 添加新条目，最好选择另一种方法！
```

```python
%pip install --upgrade --quiet  annoy
```

## 从文本创建 VectorStore

```python
from langchain_community.vectorstores import Annoy
from langchain_huggingface import HuggingFaceEmbeddings
embeddings_func = HuggingFaceEmbeddings()
```

```python
texts = ["pizza is great", "I love salad", "my car", "a dog"]
# 默认度量是角度
vector_store = Annoy.from_texts(texts, embeddings_func)
```

```python
# 允许自定义 annoy 参数，默认值为 n_trees=100, n_jobs=-1, metric="angular"
vector_store_v2 = Annoy.from_texts(
    texts, embeddings_func, metric="dot", n_trees=100, n_jobs=1
)
```

```python
vector_store.similarity_search("food", k=3)
```

```output
[Document(page_content='pizza is great', metadata={}),
 Document(page_content='I love salad', metadata={}),
 Document(page_content='my car', metadata={})]
```

```python
# 分数是一个距离度量，所以分数越低越好
vector_store.similarity_search_with_score("food", k=3)
```

```output
[(Document(page_content='pizza is great', metadata={}), 1.0944390296936035),
 (Document(page_content='I love salad', metadata={}), 1.1273186206817627),
 (Document(page_content='my car', metadata={}), 1.1580758094787598)]
```

## 从文档创建 VectorStore

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txtn.txtn.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

```python
docs[:5]
```

```output
[Document(page_content='Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.  \n\nLast year COVID-19 kept us apart. This year we are finally together again. \n\nTonight, we meet as Democrats Republicans and Independents. But most importantly as Americans. \n\nWith a duty to one another to the American people to the Constitution. \n\nAnd with an unwavering resolve that freedom will always triumph over tyranny. \n\nSix days ago, Russia’s Vladimir Putin sought to shake the foundations of the free world thinking he could make it bend to his menacing ways. But he badly miscalculated. \n\nHe thought he could roll into Ukraine and the world would roll over. Instead he met a wall of strength he never imagined. \n\nHe met the Ukrainian people. \n\nFrom President Zelenskyy to every Ukrainian, their fearlessness, their courage, their determination, inspires the world.', metadata={'source': '../../../state_of_the_union.txt'}),
 ...
]
```

```python
vector_store_from_docs = Annoy.from_documents(docs, embeddings_func)
```

```python
query = "总统对Ketanji Brown Jackson有什么评论"
docs = vector_store_from_docs.similarity_search(query)
```

```python
print(docs[0].page_content[:100])
```

```output
今晚。我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯投票权法案》
```

## 通过现有嵌入创建 VectorStore

```python
embs = embeddings_func.embed_documents(texts)
```

```python
data = list(zip(texts, embs))
vector_store_from_embeddings = Annoy.from_embeddings(data, embeddings_func)
```

```python
vector_store_from_embeddings.similarity_search_with_score("食物", k=3)
```

```output
[(Document(page_content='披萨很棒', metadata={}), 1.0944390296936035),
 (Document(page_content='我喜欢沙拉', metadata={}), 1.1273186206817627),
 (Document(page_content='我的车', metadata={}), 1.1580758094787598)]
```

## 通过嵌入搜索

```python
motorbike_emb = embeddings_func.embed_query("摩托车")
```

```python
vector_store.similarity_search_by_vector(motorbike_emb, k=3)
```

```output
[Document(page_content='我的车', metadata={}),
 Document(page_content='一只狗', metadata={}),
 Document(page_content='披萨很棒', metadata={})]
```

```python
vector_store.similarity_search_with_score_by_vector(motorbike_emb, k=3)
```

```output
[(Document(page_content='我的车', metadata={}), 1.0870471000671387),
 (Document(page_content='一只狗', metadata={}), 1.2095637321472168),
 (Document(page_content='披萨很棒', metadata={}), 1.3254905939102173)]
```

## 通过文档存储 ID 搜索

```python
vector_store.index_to_docstore_id
```

```output
{0: '2d1498a8-a37c-4798-acb9-0016504ed798',
 1: '2d30aecc-88e0-4469-9d51-0ef7e9858e6d',
 2: '927f1120-985b-4691-b577-ad5cb42e011c',
 3: '3056ddcf-a62f-48c8-bd98-b9e57a3dfcae'}
```

```python
some_docstore_id = 0  # texts[0]
vector_store.docstore._dict[vector_store.index_to_docstore_id[some_docstore_id]]
```

```output
Document(page_content='披萨很棒', metadata={})
```

```python
# 同一文档的距离为 0
vector_store.similarity_search_with_score_by_index(some_docstore_id, k=3)
```

```output
[(Document(page_content='披萨很棒', metadata={}), 0.0),
 (Document(page_content='我喜欢沙拉', metadata={}), 1.0734446048736572),
 (Document(page_content='我的车', metadata={}), 1.2895267009735107)]
```

## 保存和加载

```python
vector_store.save_local("my_annoy_index_and_docstore")
```

```output
saving config
```

```python
loaded_vector_store = Annoy.load_local(
    "my_annoy_index_and_docstore", embeddings=embeddings_func
)
```

```python
# 同一文档的距离为 0
loaded_vector_store.similarity_search_with_score_by_index(some_docstore_id, k=3)
```

```output
[(Document(page_content='披萨很棒', metadata={}), 0.0),
 (Document(page_content='我喜欢沙拉', metadata={}), 1.0734446048736572),
 (Document(page_content='我的车', metadata={}), 1.2895267009735107)]
```

## 从头开始构建

```python
import uuid
from annoy import AnnoyIndex
from langchain_community.docstore.in_memory import InMemoryDocstore
from langchain_core.documents import Document
metadatas = [{"x": "food"}, {"x": "food"}, {"x": "stuff"}, {"x": "animal"}]
# embeddings
embeddings = embeddings_func.embed_documents(texts)
# embedding dim
f = len(embeddings[0])
# index
metric = "angular"
index = AnnoyIndex(f, metric=metric)
for i, emb in enumerate(embeddings):
    index.add_item(i, emb)
index.build(10)
# docstore
documents = []
for i, text in enumerate(texts):
    metadata = metadatas[i] if metadatas else {}
    documents.append(Document(page_content=text, metadata=metadata))
index_to_docstore_id = {i: str(uuid.uuid4()) for i in range(len(documents))}
docstore = InMemoryDocstore(
    {index_to_docstore_id[i]: doc for i, doc in enumerate(documents)}
)
db_manually = Annoy(
    embeddings_func.embed_query, index, metric, docstore, index_to_docstore_id
)
```

```python
db_manually.similarity_search_with_score("吃饭！", k=3)
```

```output
[(Document(page_content='披萨很棒', metadata={'x': 'food'}), 1.1314140558242798),
 (Document(page_content='我喜欢沙拉', metadata={'x': 'food'}), 1.1668788194656372),
 (Document(page_content='我的车', metadata={'x': 'stuff'}), 1.226445198059082)]
```