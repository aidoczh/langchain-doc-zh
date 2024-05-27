# DocArray HnswSearch

[DocArrayHnswSearch](https://docs.docarray.org/user_guide/storing/index_hnswlib/) 是由 [Docarray](https://github.com/docarray/docarray) 提供的一种轻量级文档索引实现，完全在本地运行，最适合小到中等大小的数据集。它在磁盘上存储向量使用 [hnswlib](https://github.com/nmslib/hnswlib)，并将所有其他数据存储在 [SQLite](https://www.sqlite.org/index.html) 中。

本笔记展示了如何使用与 `DocArrayHnswSearch` 相关的功能。

## 设置

取消下面的单元格注释以安装 docarray 并获取/设置您的 OpenAI API 密钥（如果尚未这样做）。

```python
%pip install --upgrade --quiet  "docarray[hnswlib]"
```

```python
# 获取 OpenAI token: https://platform.openai.com/account/api-keys
# import os
# from getpass import getpass
# OPENAI_API_KEY = getpass()
# os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

## 使用 DocArrayHnswSearch

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import DocArrayHnswSearch
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
db = DocArrayHnswSearch.from_documents(
    docs, embeddings, work_dir="hnswlib_store/", n_dim=1536
)
```

### 相似性搜索

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

### 带分数的相似性搜索

返回的距离分数是余弦距离。因此，得分越低越好。

```python
docs = db.similarity_search_with_score(query)
```

```python
docs[0]
```

```output
(Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={}),
 0.36962226)
```

```python
import shutil
# 删除目录
shutil.rmtree("hnswlib_store")
```