# AnalyticDB

>[AnalyticDB for PostgreSQL](https://www.alibabacloud.com/help/zh/analyticdb-for-postgresql/latest/product-introduction-overview) 是一个用于在线分析大数据的大规模并行处理（MPP）数据仓库服务。

`AnalyticDB for PostgreSQL` 是基于开源项目 `Greenplum Database` 开发的，由 `阿里云` 进行了深度扩展。AnalyticDB for PostgreSQL 兼容 ANSI SQL 2003 语法以及 PostgreSQL 和 Oracle 数据库生态系统。AnalyticDB for PostgreSQL 还支持行存储和列存储。AnalyticDB for PostgreSQL 在离线环境下以高性能处理 PB 级别的数据，并支持高并发的在线查询。

本文档展示了如何使用与 `AnalyticDB` 向量数据库相关的功能。

要运行此示例，您需要启动并运行一个 [AnalyticDB](https://www.alibabacloud.com/help/zh/analyticdb-for-postgresql/latest/product-introduction-overview) 实例：

- 使用 [AnalyticDB Cloud Vector Database](https://www.alibabacloud.com/product/hybriddb-postgresql)。点击此处快速部署。

```python
from langchain_community.vectorstores import AnalyticDB
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

通过调用 OpenAI API，将文档拆分并获取嵌入向量。

```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```

通过设置相关环境变量连接到 AnalyticDB。

```
export PG_HOST={your_analyticdb_hostname}
export PG_PORT={your_analyticdb_port} # 可选，默认为 5432
export PG_DATABASE={your_database} # 可选，默认为 postgres
export PG_USER={database_username}
export PG_PASSWORD={database_password}
```

然后将嵌入向量和文档存储到 AnalyticDB 中。

```python
import os
connection_string = AnalyticDB.connection_string_from_db_params(
    driver=os.environ.get("PG_DRIVER", "psycopg2cffi"),
    host=os.environ.get("PG_HOST", "localhost"),
    port=int(os.environ.get("PG_PORT", "5432")),
    database=os.environ.get("PG_DATABASE", "postgres"),
    user=os.environ.get("PG_USER", "postgres"),
    password=os.environ.get("PG_PASSWORD", "postgres"),
)
vector_db = AnalyticDB.from_documents(
    docs,
    embeddings,
    connection_string=connection_string,
)
```

查询和检索数据。

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = vector_db.similarity_search(query)
```

```python
print(docs[0].page_content)
```

```output
今晚，我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯选举权法案》。而且，在此期间，通过《披露法案》，以便美国人民可以知道谁在资助我们的选举。
今晚，我想向一个致力于为这个国家服务的人表示敬意：司法部长斯蒂芬·布雷耶——一位退伍军人、宪法学者和即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。
作为总统，最重要的宪法责任之一就是提名人选担任美国最高法院法官。
而我在4天前就已经做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶级的法律专家之一，将继续延续布雷耶法官的卓越传统。
```