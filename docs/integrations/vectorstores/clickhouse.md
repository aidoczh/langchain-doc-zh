# ClickHouse
> [ClickHouse](https://clickhouse.com/) 是用于实时应用程序和分析的最快速、资源利用率最高的开源数据库，具有完整的 SQL 支持和广泛的功能，可帮助用户编写分析查询。最近增加的数据结构和距离搜索功能（如 `L2Distance`），以及[近似最近邻搜索索引](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/annindexes)使 ClickHouse 可以用作高性能、可扩展的向量数据库，用于存储和搜索具有 SQL 的向量。
本笔记展示了如何使用与 `ClickHouse` 向量搜索相关的功能。
## 设置环境
使用 Docker 设置本地 ClickHouse 服务器（可选）
```python
! docker run -d -p 8123:8123 -p9000:9000 --name langchain-clickhouse-server --ulimit nofile=262144:262144 clickhouse/clickhouse-server:23.4.2.11
```
设置 ClickHouse 客户端驱动程序
```python
%pip install --upgrade --quiet  clickhouse-connect
```
我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。
```python
import getpass
import os
if not os.environ["OPENAI_API_KEY"]:
    os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```python
from langchain_community.vectorstores import Clickhouse, ClickhouseSettings
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```
```python
from langchain_community.document_loaders import TextLoader
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
```
```python
for d in docs:
    d.metadata = {"some": "metadata"}
settings = ClickhouseSettings(table="clickhouse_vector_search_example")
docsearch = Clickhouse.from_documents(docs, embeddings, config=settings)
query = "总统对 Ketanji Brown Jackson 有何看法"
docs = docsearch.similarity_search(query)
```
```output
正在插入数据...: 100%|██████████| 42/42 [00:00<00:00, 2801.49it/s]
```
```python
print(docs[0].page_content)
```
```output
今晚，我呼吁参议院：通过《自由投票法案》。通过《约翰·刘易斯投票权法案》。而且，在此期间，通过《披露法案》，以便美国人知道是谁在资助我们的选举。
今晚，我想向一个一生致力于为这个国家服务的人致敬：司法部长斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，也是美国最高法院即将退休的法官。布雷耶司法部长，感谢您的服务。
总统拥有的最严肃的宪法责任之一是提名某人担任美国最高法院法官。
而我在4天前就做到了，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律专家之一，将继续延续布雷耶司法部长的卓越传统。
```
## 获取连接信息和数据架构
```python
print(str(docsearch))
```
```output
default.clickhouse_vector_search_example @ localhost:8123
用户名: 无
表结构:
---------------------------------------------------
|id                      |Nullable(String)        ||document                |Nullable(String)        ||embedding               |Array(Float32)          ||metadata                |Object('json')          ||uuid                    |UUID                    |
---------------------------------------------------
```
### ClickHouse 表结构
> 如果不存在，ClickHouse 表将默认自动创建。高级用户可以使用优化设置预先创建表。对于带有分片的分布式 ClickHouse 集群，表引擎应配置为 `Distributed`。
```python
print(f"ClickHouse 表 DDL:\n\n{docsearch.schema}")
```
```output
ClickHouse 表 DDL:
CREATE TABLE IF NOT EXISTS default.clickhouse_vector_search_example(
    id Nullable(String),
    document Nullable(String),
    embedding Array(Float32),
    metadata JSON,
    uuid UUID DEFAULT generateUUIDv4(),
    CONSTRAINT cons_vec_len CHECK length(embedding) = 1536,
    INDEX vec_idx embedding TYPE annoy(100,'L2Distance') GRANULARITY 1000
) ENGINE = MergeTree ORDER BY uuid SETTINGS index_granularity = 8192
```
## 过滤
您可以直接访问 ClickHouse SQL 的 `WHERE` 语句。您可以按照标准 SQL 编写 `WHERE` 子句。
**注意**：请注意 SQL 注入，此接口不得由最终用户直接调用。
如果您在设置中自定义了 `column_map`，则可以像这样使用过滤器进行搜索：
```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Clickhouse, ClickhouseSettings
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
for i, d in enumerate(docs):
    d.metadata = {"doc_id": i}
docsearch = Clickhouse.from_documents(docs, embeddings)
```
```output
插入数据中...: 100%|██████████| 42/42 [00:00<00:00, 6939.56it/s]
```
```python
meta = docsearch.metadata_column
output = docsearch.similarity_search_with_relevance_scores(
    "总统对Ketanji Brown Jackson有什么评论？",
    k=4,
    where_str=f"{meta}.doc_id<10",
)
for d, dist in output:
    print(dist, d.metadata, d.page_content[:20] + "...")
```
```output
0.6779101415357189 {'doc_id': 0} 尊敬的议长，尊敬的...
0.6997970363474885 {'doc_id': 8} 还有许多家庭...
0.7044504914336727 {'doc_id': 1} 一些公民团体...
0.7053558702165094 {'doc_id': 6} 我正在采取有力...
```
## 删除您的数据
```python
docsearch.drop()
```