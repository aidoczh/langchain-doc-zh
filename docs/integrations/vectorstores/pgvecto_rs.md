# PGVecto.rs

本笔记本展示了如何使用与Postgres向量数据库相关的功能 ([pgvecto.rs](https://github.com/tensorchord/pgvecto.rs))。

```python
%pip install "pgvecto_rs[sdk]"
```

```python
from typing import List
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.fake import FakeEmbeddings
from langchain_community.vectorstores.pgvecto_rs import PGVecto_rs
from langchain_core.documents import Document
from langchain_text_splitters import CharacterTextSplitter
```

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = FakeEmbeddings(size=3)
```

使用[官方演示docker镜像](https://github.com/tensorchord/pgvecto.rs#installation)启动数据库。

```python
! docker run --name pgvecto-rs-demo -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d tensorchord/pgvecto-rs:latest
```

然后构建数据库URL。

```python
## PGVecto.rs需要数据库的连接字符串。
## 我们将从环境变量中加载它。
import os
PORT = os.getenv("DB_PORT", 5432)
HOST = os.getenv("DB_HOST", "localhost")
USER = os.getenv("DB_USER", "postgres")
PASS = os.getenv("DB_PASS", "mysecretpassword")
DB_NAME = os.getenv("DB_NAME", "postgres")
# 使用shell运行测试：
URL = "postgresql+psycopg://{username}:{password}@{host}:{port}/{db_name}".format(
    port=PORT,
    host=HOST,
    username=USER,
    password=PASS,
    db_name=DB_NAME,
)
```

最后，从文档创建VectorStore：

```python
db1 = PGVecto_rs.from_documents(
    documents=docs,
    embedding=embeddings,
    db_url=URL,
    # 表名为f"collection_{collection_name}"，因此应该是唯一的。
    collection_name="state_of_the_union",
)
```

您可以稍后连接到表：

```python
# 使用collection_name创建新的空向量存储。
# 或者如果存在的话，连接到数据库中的现有向量存储。
# 参数应与创建向量存储时相同。
db1 = PGVecto_rs.from_collection_name(
    embedding=embeddings,
    db_url=URL,
    collection_name="state_of_the_union",
)
```

确保用户有权限创建表。

## 使用分数进行相似性搜索

### 使用欧几里德距离进行相似性搜索（默认）

```python
query = "总统对Ketanji Brown Jackson有什么看法"
docs: List[Document] = db1.similarity_search(query, k=4)
for doc in docs:
    print(doc.page_content)
    print("======================")
```

### 使用过滤器进行相似性搜索

```python
from pgvecto_rs.sdk.filters import meta_contains
query = "总统对Ketanji Brown Jackson有什么看法"
docs: List[Document] = db1.similarity_search(
    query, k=4, filter=meta_contains({"source": "../../how_to/state_of_the_union.txt"})
)
for doc in docs:
    print(doc.page_content)
    print("======================")
```

或者：

```python
query = "总统对Ketanji Brown Jackson有什么看法"
docs: List[Document] = db1.similarity_search(
    query, k=4, filter={"source": "../../how_to/state_of_the_union.txt"}
)
for doc in docs:
    print(doc.page_content)
    print("======================")
```