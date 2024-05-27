# 贝果

> [贝果](https://www.bagel.net/)（`用于AI的开放推理平台`），就像是面向AI数据的GitHub。

这是一个协作平台，用户可以在其中创建、分享和管理推理数据集。它可以支持独立开发者的私人项目、企业内部的协作以及数据DAO的公共贡献。

### 安装和设置

```bash
pip install bagelML
```

## 从文本创建向量存储

```python
from langchain_community.vectorstores import Bagel
texts = ["hello bagel", "hello langchain", "I love salad", "my car", "a dog"]
# 创建聚类并添加文本
cluster = Bagel.from_texts(cluster_name="testing", texts=texts)
```

```python
# 相似性搜索
cluster.similarity_search("bagel", k=3)
```

```output
[Document(page_content='hello bagel', metadata={}),
 Document(page_content='my car', metadata={}),
 Document(page_content='I love salad', metadata={})]
```

```python
# 分数是一个距离度量，所以分数越低越好
cluster.similarity_search_with_score("bagel", k=3)
```

```output
[(Document(page_content='hello bagel', metadata={}), 0.27392977476119995),
 (Document(page_content='my car', metadata={}), 1.4783176183700562),
 (Document(page_content='I love salad', metadata={}), 1.5342965126037598)]
```

```python
# 删除聚类
cluster.delete_cluster()
```

## 从文档创建向量存储

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)[:10]
```

```python
# 使用文档创建聚类
cluster = Bagel.from_documents(cluster_name="testing_with_docs", documents=docs)
```

```python
# 相似性搜索
query = "What did the president say about Ketanji Brown Jackson"
docs = cluster.similarity_search(query)
print(docs[0].page_content[:102])
```

```output
Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the
```

## 获取聚类中的所有文本/文档

```python
texts = ["hello bagel", "this is langchain"]
cluster = Bagel.from_texts(cluster_name="testing", texts=texts)
cluster_data = cluster.get()
```

```python
# 所有键
cluster_data.keys()
```

```output
dict_keys(['ids', 'embeddings', 'metadatas', 'documents'])
```

```python
# 所有键和值
cluster_data
```

```output
{'ids': ['578c6d24-3763-11ee-a8ab-b7b7b34f99ba',
  '578c6d25-3763-11ee-a8ab-b7b7b34f99ba',
  'fb2fc7d8-3762-11ee-a8ab-b7b7b34f99ba',
  'fb2fc7d9-3762-11ee-a8ab-b7b7b34f99ba',
  '6b40881a-3762-11ee-a8ab-b7b7b34f99ba',
  '6b40881b-3762-11ee-a8ab-b7b7b34f99ba',
  '581e691e-3762-11ee-a8ab-b7b7b34f99ba',
  '581e691f-3762-11ee-a8ab-b7b7b34f99ba'],
 'embeddings': None,
 'metadatas': [{}, {}, {}, {}, {}, {}, {}, {}],
 'documents': ['hello bagel',
  'this is langchain',
  'hello bagel',
  'this is langchain',
  'hello bagel',
  'this is langchain',
  'hello bagel',
  'this is langchain']}
```

```python
cluster.delete_cluster()
```

## 创建带有元数据的聚类并使用元数据进行筛选

```python
texts = ["hello bagel", "this is langchain"]
metadatas = [{"source": "notion"}, {"source": "google"}]
cluster = Bagel.from_texts(cluster_name="testing", texts=texts, metadatas=metadatas)
cluster.similarity_search_with_score("hello bagel", where={"source": "notion"})
```

```output
[(Document(page_content='hello bagel', metadata={'source': 'notion'}), 0.0)]
```

```python
# 删除聚类
cluster.delete_cluster()
```