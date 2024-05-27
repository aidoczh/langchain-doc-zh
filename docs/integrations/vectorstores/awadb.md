# AwaDB

[AwaDB](https://github.com/awa-ai/awadb) 是一种用于搜索和存储由 LLM 应用程序使用的嵌入向量的 AI 本地数据库。

本笔记本展示了如何使用与 `AwaDB` 相关的功能。

```python
%pip install --upgrade --quiet  awadb
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import AwaDB
from langchain_text_splitters import CharacterTextSplitter
```

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=100, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

```python
db = AwaDB.from_documents(docs)
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
```

```python
print(docs[0].page_content)
```

```output
And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 使用分数进行相似性搜索

返回的距离分数在 0-1 之间。0 表示不相似，1 表示最相似。

```python
docs = db.similarity_search_with_score(query)
```

```python
print(docs[0])
```

```output
(Document(page_content='And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt'}), 0.561813814013747)
```

## 恢复之前创建并添加数据的表

AwaDB 会自动保留已添加的文档数据。

如果您想要恢复之前创建并添加的表，只需按照以下步骤操作：

```python
import awadb
awadb_client = awadb.Client()
ret = awadb_client.Load("langchain_awadb")
if ret:
    print("awadb load table success")
else:
    print("awadb load table failed")
```

awadb load table success