# SurrealDB

[SurrealDB](https://surrealdb.com/) 是一款端到端的云原生数据库，专为现代应用程序设计，包括 Web、移动、无服务器、Jamstack、后端和传统应用程序。使用 SurrealDB，您可以简化数据库和 API 基础架构，减少开发时间，并快速、经济高效地构建安全、高性能的应用程序。

SurrealDB 的主要特点包括：

- **减少开发时间：** SurrealDB 通过消除大多数服务器端组件的需求，简化了数据库和 API 堆栈，使您能够更快、更便宜地构建安全、高性能的应用程序。

- **实时协作 API 后端服务：** SurrealDB 既可以作为数据库，也可以作为 API 后端服务，实现实时协作。

- **支持多种查询语言：** SurrealDB 支持客户端设备的 SQL 查询、GraphQL、ACID 事务、WebSocket 连接、结构化和非结构化数据、图形查询、全文索引和地理空间查询。

- **细粒度访问控制：** SurrealDB 提供基于行级权限的访问控制，让您能够精确管理数据访问。

查看 [功能](https://surrealdb.com/features)、最新的 [发布](https://surrealdb.com/releases) 和 [文档](https://surrealdb.com/docs)。

这篇笔记展示了如何使用与 `SurrealDBStore` 相关的功能。

## 设置

取消下面的单元格注释以安装 surrealdb。

```python
# %pip install --upgrade --quiet  surrealdb langchain langchain-community
```

## 使用 SurrealDBStore

```python
# add this import for running in jupyter notebook
import nest_asyncio
nest_asyncio.apply()
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import SurrealDBStore
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

```python
documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = HuggingFaceEmbeddings()
```

### 创建 SurrealDBStore 对象

```python
db = SurrealDBStore(
    dburl="ws://localhost:8000/rpc",  # 托管 SurrealDB 数据库的 URL
    embedding_function=embeddings,
    db_user="root",  # 如果需要 SurrealDB 凭据：数据库用户名
    db_pass="root",  # 如果需要 SurrealDB 凭据：数据库密码
    # ns="langchain", # 用于 vectorstore 的命名空间
    # db="database",  # 用于 vectorstore 的数据库
    # collection="documents", # 用于 vectorstore 的集合
)
# 这是为了初始化 SurrealDB 的底层异步库
await db.initialize()
# 从 vectorstore 集合中删除所有现有文档
await db.adelete()
# 将文档添加到 vectorstore
ids = await db.aadd_documents(docs)
# 添加文档的文档 ID
ids[:5]
```

```output
['documents:38hz49bv1p58f5lrvrdc',
 'documents:niayw63vzwm2vcbh6w2s',
 'documents:it1fa3ktplbuye43n0ch',
 'documents:il8f7vgbbp9tywmsn98c',
 'documents:vza4c6cqje0avqd58gal']
```

### （或者）创建 SurrealDBStore 对象并添加文档

```python
await db.adelete()
db = await SurrealDBStore.afrom_documents(
    dburl="ws://localhost:8000/rpc",  # 托管 SurrealDB 数据库的 URL
    embedding=embeddings,
    documents=docs,
    db_user="root",  # 如果需要 SurrealDB 凭据：数据库用户名
    db_pass="root",  # 如果需要 SurrealDB 凭据：数据库密码
    # ns="langchain", # 用于 vectorstore 的命名空间
    # db="database",  # 用于 vectorstore 的数据库
    # collection="documents", # 用于 vectorstore 的集合
)
```

### 相似性搜索

```python
query = "总统对 Ketanji Brown Jackson 有何看法"
docs = await db.asimilarity_search(query)
```

```python
print(docs[0].page_content)
```

```output
今晚。我呼吁参议院：通过《自由选举法》。通过《约翰·刘易斯选举权法案》。顺便说一句，通过《披露法》，这样美国人就可以知道谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一名陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶法官，感谢您的服务。
总统的最严肃宪法责任之一是提名某人担任美国最高法院的法官。
4 天前，我提名了联邦上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律专家之一，将继续延续布雷耶法官的卓越传统。
```

返回的距离分数是余弦距离。因此，得分越低越好。

```python
docs = await db.asimilarity_search_with_score(query)
```

```python
docs[0]
```

```output
(Document(page_content='今晚，我呼吁参议院：通过《自由选举法案》。通过《约翰·刘易斯选举权法案》。而且，顺便通过《披露法案》，这样美国人就可以知道谁在资助我们的选举。\n\n今晚，我想向一个致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者和即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。\n\n总统最重要的宪法责任之一是提名人选担任美国最高法院法官。\n\n我在4天前就这样做了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律智慧之一，将继续延续布雷耶法官的卓越传统。', metadata={'id': 'documents:slgdlhjkfknhqo15xz0w', 'source': '../../how_to/state_of_the_union.txt'}),
 0.39839531721941895)
```

