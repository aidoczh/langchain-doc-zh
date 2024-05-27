# Relyt

[Relyt](https://docs.relyt.cn/docs/vector-engine/use/) 是一个云原生数据仓库服务，旨在在线分析大量数据。

`Relyt` 兼容 ANSI SQL 2003 语法以及 PostgreSQL 和 Oracle 数据库生态系统。Relyt 还支持行存储和列存储。Relyt 在离线模式下处理 PB 级别的数据，并支持高并发的在线查询。

这个笔记展示了如何使用与 `Relyt` 向量数据库相关的功能。

要运行，您应该已经启动并运行了一个 [Relyt](https://docs.relyt.cn/) 实例：

- 使用 [Relyt Vector Database](https://docs.relyt.cn/docs/vector-engine/use/)。点击这里快速部署它。

```python
%pip install "pgvecto_rs[sdk]"
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.fake import FakeEmbeddings
from langchain_community.vectorstores import Relyt
from langchain_text_splitters import CharacterTextSplitter
```

通过调用社区 API 分割文档并获取嵌入向量

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = FakeEmbeddings(size=1536)
```

通过设置相关环境变量连接到 Relyt

```
export PG_HOST={your_relyt_hostname}
export PG_PORT={your_relyt_port} # 可选，默认为 5432
export PG_DATABASE={your_database} # 可选，默认为 postgres
export PG_USER={database_username}
export PG_PASSWORD={database_password}
```

然后将您的嵌入向量和文档存储到 Relyt

```python
import os
connection_string = Relyt.connection_string_from_db_params(
    driver=os.environ.get("PG_DRIVER", "psycopg2cffi"),
    host=os.environ.get("PG_HOST", "localhost"),
    port=int(os.environ.get("PG_PORT", "5432")),
    database=os.environ.get("PG_DATABASE", "postgres"),
    user=os.environ.get("PG_USER", "postgres"),
    password=os.environ.get("PG_PASSWORD", "postgres"),
)
vector_db = Relyt.from_documents(
    docs,
    embeddings,
    connection_string=connection_string,
)
```

查询和检索数据

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = vector_db.similarity_search(query)
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