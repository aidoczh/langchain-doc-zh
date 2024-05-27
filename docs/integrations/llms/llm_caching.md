

# LLM 缓存集成

本文介绍如何使用不同的缓存来缓存单个 LLM 调用的结果。

```python
from langchain.globals import set_llm_cache
from langchain_openai import OpenAI
# 为了更明显地展示缓存效果，让我们使用一个速度较慢的模型。
llm = OpenAI(model_name="gpt-3.5-turbo-instruct", n=2, best_of=2)
```

## `内存` 缓存

```python
from langchain_community.cache import InMemoryCache
set_llm_cache(InMemoryCache())
```
```python
%%time
# 第一次调用时，尚未缓存，所以需要更长的时间
llm("Tell me a joke")
```
```output
CPU times: user 52.2 ms, sys: 15.2 ms, total: 67.4 ms
Wall time: 1.19 s
```
```output
"\n\n为什么自行车不能自己站起来？因为它...太累了！"
```
```python
%%time
# 第二次调用时，已经缓存，所以速度更快
llm("Tell me a joke")
```
```output
CPU times: user 191 µs, sys: 11 µs, total: 202 µs
Wall time: 205 µs
```
```output
"\n\n为什么自行车不能自己站起来？因为它...太累了！"
```

## `SQLite` 缓存

```python
!rm .langchain.db
```
```python
# 我们可以使用 SQLite 缓存做同样的事情
from langchain_community.cache import SQLiteCache
set_llm_cache(SQLiteCache(database_path=".langchain.db"))
```
```python
%%time
# 第一次调用时，尚未缓存，所以需要更长的时间
llm("Tell me a joke")
```
```output
CPU times: user 33.2 ms, sys: 18.1 ms, total: 51.2 ms
Wall time: 667 ms
```
```output
'\n\n为什么鸡要过马路？\n\n为了到达对面。'
```
```python
%%time
# 第二次调用时，已经缓存，所以速度更快
llm("Tell me a joke")
```
```output
CPU times: user 4.86 ms, sys: 1.97 ms, total: 6.83 ms
Wall time: 5.79 ms
```
```output
'\n\n为什么鸡要过马路？\n\n为了到达对面。'
```

## `Upstash Redis` 缓存

### 标准缓存

使用 [Upstash Redis](https://upstash.com) 来缓存提示和响应，使用无服务器 HTTP API。

```python
import langchain
from langchain_community.cache import UpstashRedisCache
from upstash_redis import Redis
URL = "<UPSTASH_REDIS_REST_URL>"
TOKEN = "<UPSTASH_REDIS_REST_TOKEN>"
langchain.llm_cache = UpstashRedisCache(redis_=Redis(url=URL, token=TOKEN))
```
```python
%%time
# 第一次调用时，尚未缓存，所以需要更长的时间
llm("Tell me a joke")
```
```output
CPU times: user 7.56 ms, sys: 2.98 ms, total: 10.5 ms
Wall time: 1.14 s
```
```output
'\n\n为什么鸡要过马路？\n\n为了到达对面！'
```
```python
%%time
# 第二次调用时，已经缓存，所以速度更快
llm("Tell me a joke")
```
```output
CPU times: user 2.78 ms, sys: 1.95 ms, total: 4.73 ms
Wall time: 82.9 ms
```
```output
'\n\n为什么鸡要过马路？\n\n为了到达对面！'
```

## `Redis` 缓存

### 标准缓存

使用 [Redis](/docs/integrations/providers/redis) 来缓存提示和响应。

```python
# 我们可以使用 Redis 缓存做同样的事情
# (在运行此示例之前，请确保您的本地 Redis 实例正在运行)
from langchain_community.cache import RedisCache
from redis import Redis
set_llm_cache(RedisCache(redis_=Redis()))
```
```python
%%time
# 第一次调用时，尚未缓存，所以需要更长的时间
llm("Tell me a joke")
```
```output
CPU times: user 6.88 ms, sys: 8.75 ms, total: 15.6 ms
Wall time: 1.04 s
```
```output
'\n\n为什么鸡要过马路？\n\n为了到达对面！'
```
```python
%%time
# 第二次调用时，已经缓存，所以速度更快
llm("Tell me a joke")
```
```output
CPU times: user 1.59 ms, sys: 610 µs, total: 2.2 ms
Wall time: 5.58 ms
```
```output
'\n\n为什么鸡要过马路？\n\n为了到达对面！'
```

### 语义缓存

使用 [Redis](/docs/integrations/providers/redis) 缓存提示和响应，并根据语义相似性评估命中情况。

```python
from langchain_community.cache import RedisSemanticCache
from langchain_openai import OpenAIEmbeddings
set_llm_cache(
    RedisSemanticCache(redis_url="redis://localhost:6379", embedding=OpenAIEmbeddings())
)
```
```python
%%time
# 第一次调用时，尚未缓存，所以需要更长的时间
llm("Tell me a joke")
```
```output
CPU times: user 351 ms, sys: 156 ms, total: 507 ms
Wall time: 3.37 s
```
```output
"\n\n为什么科学家不相信原子？\n因为它们构成了一切。"
```
```python
%%time
# 第二次调用时，虽然不是直接命中，但问题在语义上与原始问题相似，
# 因此使用了缓存的结果！
llm("Tell me one joke")
```
```output
CPU times: user 6.25 ms, sys: 2.72 ms, total: 8.97 ms
Wall time: 262 ms
```
```output
"\n\n为什么科学家不相信原子？\n因为它们构成了一切。"
```

## `GPTCache`

我们可以使用 [GPTCache](https://github.com/zilliztech/GPTCache) 来进行精确匹配缓存，或者基于语义相似性缓存结果。

让我们首先从一个精确匹配的示例开始。

```python
import hashlib
from gptcache import Cache
from gptcache.manager.factory import manager_factory
from gptcache.processor.pre import get_prompt
from langchain_community.cache import GPTCache
def get_hashed_name(name):
    return hashlib.sha256(name.encode()).hexdigest()
def init_gptcache(cache_obj: Cache, llm: str):
    hashed_llm = get_hashed_name(llm)
    cache_obj.init(
        pre_embedding_func=get_prompt,
        data_manager=manager_factory(manager="map", data_dir=f"map_cache_{hashed_llm}"),
    )
set_llm_cache(GPTCache(init_gptcache))
```
```python
%%time
# 第一次运行，尚未缓存，所以耗时较长
llm("Tell me a joke")
```
```output
CPU times: user 21.5 ms, sys: 21.3 ms, total: 42.8 ms
Wall time: 6.2 s
```
```output
'\n\n为什么小鸡要过马路？\n\n为了到达对面！'
```
```python
%%time
# 第二次运行，已经缓存，所以速度更快
llm("Tell me a joke")
```
```output
CPU times: user 571 µs, sys: 43 µs, total: 614 µs
Wall time: 635 µs
```
```output
'\n\n为什么小鸡要过马路？\n\n为了到达对面！'
```

现在让我们展示一个语义相似性缓存的示例。

```python
import hashlib
from gptcache import Cache
from gptcache.adapter.api import init_similar_cache
from langchain_community.cache import GPTCache
def get_hashed_name(name):
    return hashlib.sha256(name.encode()).hexdigest()
def init_gptcache(cache_obj: Cache, llm: str):
    hashed_llm = get_hashed_name(llm)
    init_similar_cache(cache_obj=cache_obj, data_dir=f"similar_cache_{hashed_llm}")
set_llm_cache(GPTCache(init_gptcache))
```
```python
%%time
# 第一次运行，尚未缓存，所以耗时较长
llm("Tell me a joke")
```
```output
CPU times: user 1.42 s, sys: 279 ms, total: 1.7 s
Wall time: 8.44 s
```
```output
'\n\n为什么小鸡要过马路？\n\n为了到达对面。'
```
```python
%%time
# 这是一个精确匹配，因此在缓存中找到
llm("Tell me a joke")
```
```output
CPU times: user 866 ms, sys: 20 ms, total: 886 ms
Wall time: 226 ms
```
```output
'\n\n为什么小鸡要过马路？\n\n为了到达对面！'
```
```python
%%time
# 这不是一个精确匹配，但在语义上相似，因此也能命中缓存
llm("Tell me joke")
```
```output
CPU times: user 853 ms, sys: 14.8 ms, total: 868 ms
Wall time: 224 ms
```
```output
'\n\n为什么小鸡要过马路？\n\n为了到达对面！'
```

## `Momento` 缓存

使用 [Momento](/docs/integrations/providers/momento) 缓存提示和响应。

需要使用 Momento，取消下面的注释以安装：

```python
%pip install --upgrade --quiet  momento
```

您需要获取一个 Momento 授权令牌才能使用这个类。这可以通过将其传递给 `MomentoChatMessageHistory.from_client_params` 的命名参数 `auth_token`，或者将其设置为环境变量 `MOMENTO_AUTH_TOKEN`。

```python
from datetime import timedelta
from langchain_community.cache import MomentoCache
cache_name = "langchain"
ttl = timedelta(days=1)
set_llm_cache(MomentoCache.from_client_params(cache_name, ttl))
```
```python
%%time
# 第一次运行，尚未缓存，所以耗时较长
llm("Tell me a joke")
```
```output
CPU times: user 40.7 ms, sys: 16.5 ms, total: 57.2 ms
Wall time: 1.73 s
```
```output
'\n\n为什么小鸡要过马路？\n\n为了到达对面！'
```
```python
%%time
# 第二次运行，已经缓存，所以速度更快
# 在与缓存相同区域运行时，延迟为个位数毫秒
llm("Tell me a joke")
```
```output
CPU times: user 3.16 ms, sys: 2.98 ms, total: 6.14 ms
Wall time: 57.9 ms
```
```output
'\n\n为什么小鸡要过马路？\n\n为了到达对面！'
```

## `SQLAlchemy` 缓存

您可以使用 `SQLAlchemyCache` 与 `SQLAlchemy` 支持的任何 SQL 数据库进行缓存。

```python
# from langchain.cache import SQLAlchemyCache
# from sqlalchemy import create_engine
# engine = create_engine("postgresql://postgres:postgres@localhost:5432/postgres")
# set_llm_cache(SQLAlchemyCache(engine))
```

### 自定义 SQLAlchemy 模式

您可以定义自己的声明性 SQLAlchemyCache 子类来自定义用于缓存的模式。例如，为了支持在 Postgres 中使用高速全文提示索引，可以使用以下方法：

```python
from langchain_community.cache import SQLAlchemyCache
from sqlalchemy import Column, Computed, Index, Integer, Sequence, String, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy_utils import TSVectorType
Base = declarative_base()
class FulltextLLMCache(Base):
    """用于全文索引的 Postgres 表 LLM 缓存"""
    __tablename__ = "llm_cache_fulltext"
    id = Column(Integer, Sequence("cache_id"), primary_key=True)
    prompt = Column(String, nullable=False)
    llm = Column(String, nullable=False)
    idx = Column(Integer)
    response = Column(String)
    prompt_tsv = Column(
        TSVectorType(),
        Computed("to_tsvector('english', llm || ' ' || prompt)", persisted=True),
    )
    __table_args__ = (
        Index("idx_fulltext_prompt_tsv", prompt_tsv, postgresql_using="gin"),
    )
engine = create_engine("postgresql://postgres:postgres@localhost:5432/postgres")
set_llm_cache(SQLAlchemyCache(engine, FulltextLLMCache))
```

## `Cassandra` 缓存

> [Apache Cassandra®](https://cassandra.apache.org/) 是一种 NoSQL、面向行的、高度可扩展且高度可用的数据库。从版本 5.0 开始，该数据库具备 [矢量搜索功能](https://cassandra.apache.org/doc/trunk/cassandra/vector-search/overview.html)。

您可以使用 Cassandra 缓存 LLM 响应，可以选择精确匹配的 `CassandraCache` 或（基于向量相似性的）`CassandraSemanticCache`。

让我们看看两者的运作方式。接下来的单元格将引导您完成（少量）必需的设置，接着的单元格展示了两种可用的缓存类。

### 必需的依赖项

```python
%pip install --upgrade --quiet "cassio>=0.1.4"
```

### 连接到数据库

本页面展示的 Cassandra 缓存可用于 Cassandra 以及其他派生数据库，比如使用 CQL（Cassandra 查询语言）协议的 Astra DB。

> DataStax [Astra DB](https://docs.datastax.com/en/astra-serverless/docs/vector-search/quickstart.html) 是建立在 Cassandra 基础上的托管式无服务器数据库，提供相同的接口和优势。

根据您是连接到 Cassandra 集群还是通过 CQL 连接到 Astra DB，实例化缓存时会提供不同的参数（通过初始化 CassIO 连接）。

#### 连接到 Cassandra 集群

您首先需要创建一个 `cassandra.cluster.Session` 对象，如 [Cassandra 驱动程序文档](https://docs.datastax.com/en/developer/python-driver/latest/api/cassandra/cluster/#module-cassandra.cluster) 中所述。细节会有所不同（例如网络设置和身份验证），但大致如下：

```python
from cassandra.cluster import Cluster
cluster = Cluster(["127.0.0.1"])
session = cluster.connect()
```

现在，您可以将会话与所需的 keyspace 名称一起设置为全局 CassIO 参数：

```python
import cassio
CASSANDRA_KEYSPACE = input("CASSANDRA_KEYSPACE = ")
cassio.init(session=session, keyspace=CASSANDRA_KEYSPACE)
```
```output
CASSANDRA_KEYSPACE =  demo_keyspace
```

#### 通过 CQL 连接到 Astra DB

在这种情况下，您可以使用以下连接参数初始化 CassIO：

- 数据库 ID，例如 `01234567-89ab-cdef-0123-456789abcdef`

- 令牌，例如 `AstraCS:6gBhNmsk135....`（必须是“数据库管理员”令牌）

- 可选的 Keyspace 名称（如果省略，将使用数据库的默认 Keyspace）

```python
import getpass
ASTRA_DB_ID = input("ASTRA_DB_ID = ")
ASTRA_DB_APPLICATION_TOKEN = getpass.getpass("ASTRA_DB_APPLICATION_TOKEN = ")
desired_keyspace = input("ASTRA_DB_KEYSPACE（可选，可以留空） = ")
if desired_keyspace:
    ASTRA_DB_KEYSPACE = desired_keyspace
else:
    ASTRA_DB_KEYSPACE = None
```
```output
ASTRA_DB_ID =  01234567-89ab-cdef-0123-456789abcdef
ASTRA_DB_APPLICATION_TOKEN =  ········
ASTRA_DB_KEYSPACE（可选，可以留空） =  my_keyspace
```
```python
import cassio
cassio.init(
    database_id=ASTRA_DB_ID,
    token=ASTRA_DB_APPLICATION_TOKEN,
    keyspace=ASTRA_DB_KEYSPACE,
)
```

### Cassandra：精确缓存

当提供的提示与已经遇到的内容 _完全_ 相同时，这将避免调用 LLM：

```python
from langchain_community.cache import CassandraCache
from langchain_core.globals import set_llm_cache
set_llm_cache(CassandraCache())
```
```python
%%time
print(llm.invoke("为什么月球总是展示同一面？"))
```
```output
月球与地球之间存在潮汐锁定，这意味着月球自转的轴与绕地球的轨道同步。这导致月球总是向地球展示同一面。这是因为地球和月球之间的引力作用导致月球的自转随时间减慢，直到达到一个点，月球绕自身轴旋转所需的时间与绕地球轨道所需的时间相同。这种现象在围绕母行星运行的卫星中很常见，被称为潮汐锁定。
CPU times: user 92.5 ms, sys: 8.89 ms, total: 101 ms
Wall time: 1.98 s
```
```python
%%time
print(llm.invoke("为什么月球总是展示同一面？"))
```
```output
月球与地球之间存在潮汐锁定，这意味着月球自转的轴与绕地球的轨道同步。这导致月球总是向地球展示同一面。这是因为地球和月球之间的引力作用导致月球的自转随时间减慢，直到达到一个点，月球绕自身轴旋转所需的时间与绕地球轨道所需的时间相同。这种现象在围绕母行星运行的卫星中很常见，被称为潮汐锁定。```

CPU 时间：用户 5.51 毫秒，系统：0 毫秒，总计：5.51 毫秒

墙上时间：5.78 毫秒

```
### 卡桑德拉：语义缓存
该缓存将进行语义相似性搜索，并在找到足够相似的缓存条目时返回命中。为此，您需要提供您选择的 `Embeddings` 实例。
```python
from langchain_openai import OpenAIEmbeddings
embedding = OpenAIEmbeddings()
```
```python
from langchain_community.cache import CassandraSemanticCache
from langchain_core.globals import set_llm_cache
set_llm_cache(
    CassandraSemanticCache(
        embedding=embedding,
        table_name="my_semantic_cache",
    )
)
```
```python
%%time
print(llm.invoke("为什么月球总是展示同一面？"))
```
```output

月球之所以总是展示同一面，是因为一种称为同步自转的现象。这意味着月球绕其轴线旋转的速度与绕地球轨道运行的速度相同，大约需要27.3天。这导致月球的同一面始终面向地球。这是由地球和月球之间的引力所致，这些引力导致月球的自转逐渐减慢并与其轨道同步。这在我们太阳系的许多卫星中是一种常见现象。

CPU 时间：用户 49.5 毫秒，系统：7.38 毫秒，总计：56.9 毫秒

墙上时间：2.55 秒

```
```python
%%time
print(llm.invoke("为什么我们总是看到月球的一面？"))
```
```output

月球之所以总是展示同一面，是因为一种称为同步自转的现象。这意味着月球绕其轴线旋转的速度与绕地球轨道运行的速度相同，大约需要27.3天。这导致月球的同一面始终面向地球。这是由地球和月球之间的引力所致，这些引力导致月球的自转逐渐减慢并与其轨道同步。这在我们太阳系的许多卫星中是一种常见现象。

CPU 时间：用户 21.2 毫秒，系统：3.38 毫秒，总计：24.6 毫秒

墙上时间：532 毫秒

```
#### 归属声明
>Apache Cassandra、Cassandra 和 Apache 分别是 [Apache Software Foundation](http://www.apache.org/) 在美国和/或其他国家的注册商标或商标。
## `Astra DB` 缓存
您可以轻松地将 [Astra DB](https://docs.datastax.com/en/astra/home/astra.html) 用作 LLM 缓存，可以选择“exact”或“基于语义”的缓存。
确保您有一个正在运行的数据库（必须是启用矢量的数据库才能使用语义缓存），并在 Astra 仪表板上获取所需的凭据：
- API 端点类似于 `https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com`
- 令牌类似于 `AstraCS:6gBhNmsk135....`
```python
import getpass
ASTRA_DB_API_ENDPOINT = input("ASTRA_DB_API_ENDPOINT = ")
ASTRA_DB_APPLICATION_TOKEN = getpass.getpass("ASTRA_DB_APPLICATION_TOKEN = ")
```
```output

ASTRA_DB_API_ENDPOINT =  https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com

ASTRA_DB_APPLICATION_TOKEN =  ········

```
### Astra DB 精确 LLM 缓存
当提供的提示与已经遇到的提示“完全”相同时，这将避免调用 LLM：
```python
from langchain.globals import set_llm_cache
from langchain_astradb import AstraDBCache
set_llm_cache(
    AstraDBCache(
        api_endpoint=ASTRA_DB_API_ENDPOINT,
        token=ASTRA_DB_APPLICATION_TOKEN,
    )
)
```
```python
%%time
print(llm.invoke("真正的伪造与虚假的真相相同吗？"))
```
```output

对于这个问题没有明确的答案，因为它取决于“真正的伪造”和“虚假的真相”这两个术语的解释。然而，一个可能的解释是，真正的伪造是一种旨在欺骗的伪造品或模仿品，而虚假的真相是一种被呈现为真实的虚假陈述。

CPU 时间：用户 70.8 毫秒，系统：4.13 毫秒，总计：74.9 毫秒

墙上时间：2.06 秒

```
```python
%%time
print(llm.invoke("真正的伪造与虚假的真相相同吗？"))
```
```output

对于这个问题没有明确的答案，因为它取决于“真正的伪造”和“虚假的真相”这两个术语的解释。然而，一个可能的解释是，真正的伪造是一种旨在欺骗的伪造品或模仿品，而虚假的真相是一种被呈现为真实的虚假陈述。

CPU 时间：用户 15.1 毫秒，系统：3.7 毫秒，总计：18.8 毫秒

墙上时间：531 毫秒

```
### Astra DB 语义缓存
该缓存将进行语义相似性搜索，并在找到足够相似的缓存条目时返回命中。为此，您需要提供您选择的 `Embeddings` 实例。
```python
from langchain_openai import OpenAIEmbeddings
embedding = OpenAIEmbeddings()
```
```python
from langchain_astradb import AstraDBSemanticCache
set_llm_cache(
```python
from langchain_community.cache import AzureCosmosDBSemanticCache
from langchain_community.vectorstores.azure_cosmos_db import (
    CosmosDBSimilarityType,
    CosmosDBVectorSearchType,
)
from langchain_openai import OpenAIEmbeddings
# 从 Azure Cosmos DB 语义缓存中调用 AstraDBSemanticCache
set_llm_cache(
    AzureCosmosDBSemanticCache(
        cosmosdb_connection_string=CONNECTION_STRING,
        cosmosdb_client=None,
        embedding=OpenAIEmbeddings(),
        database_name=DB_NAME,
        collection_name=COLLECTION_NAME,
        num_lists=num_lists,
        similarity=similarity_algorithm,
        kind=kind,
        dimensions=dimensions,
        m=m,
        ef_construction=ef_construction,
        ef_search=ef_search,
        score_threshold=score_threshold,
        application_name=application_name,
    )
)
```
```python
%%time
# 第一次调用时，由于尚未缓存，所以需要更长时间
llm("Tell me a joke")
```
```output
CPU times: user 45.6 ms, sys: 19.7 ms, total: 65.3 ms
Wall time: 2.29 s
```
```output
'\n\nWhy was the math book sad? Because it had too many problems.'
```
```python
%%time
# 第一次调用时，由于尚未缓存，所以需要更长时间
llm("Tell me a joke")
```
```output
CPU times: user 9.61 ms, sys: 3.42 ms, total: 13 ms
Wall time: 474 ms
```

## `Elasticsearch` 缓存

这是一个使用 Elasticsearch 的 LLM 缓存层。

首先安装 LangChain 与 Elasticsearch 的集成。

```python
%pip install -U langchain-elasticsearch
```

使用 `ElasticsearchCache` 类。

简单示例：

```python
from elasticsearch import Elasticsearch
from langchain.globals import set_llm_cache
from langchain_elasticsearch import ElasticsearchCache
es_client = Elasticsearch(hosts="http://localhost:9200")
set_llm_cache(
    ElasticsearchCache(
        es_connection=es_client,
        index_name="llm-chat-cache",
        metadata={"project": "my_chatgpt_project"},
    )
)
```

`index_name` 参数也可以接受别名。这允许使用我们建议考虑用于管理保留和控制缓存增长的 [ILM: 管理索引生命周期](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)。

查看类的文档字符串以获取所有参数。

### 对生成的文本建立索引

默认情况下，缓存数据是无法搜索的。开发人员可以自定义构建 Elasticsearch 文档的方式，以添加索引文本字段，例如将由 LLM 生成的文本放入其中。

可以通过子类化并重写方法来实现此目的。新的缓存类也可以应用于现有的缓存索引：

```python
import json
from typing import Any, Dict, List
```
```python
from elasticsearch import Elasticsearch
from langchain.globals import set_llm_cache
from langchain_core.caches import RETURN_VAL_TYPE
from langchain_elasticsearch import ElasticsearchCache
class SearchableElasticsearchCache(ElasticsearchCache):
    @property
    def mapping(self) -> Dict[str, Any]:
        mapping = super().mapping
        mapping["mappings"]["properties"]["parsed_llm_output"] = {
            "type": "text",
            "analyzer": "english",
        }
        return mapping
    def build_document(
        self, prompt: str, llm_string: str, return_val: RETURN_VAL_TYPE
    ) -> Dict[str, Any]:
        body = super().build_document(prompt, llm_string, return_val)
        body["parsed_llm_output"] = self._parse_output(body["llm_output"])
        return body
    @staticmethod
    def _parse_output(data: List[str]) -> List[str]:
        return [
            json.loads(output)["kwargs"]["message"]["kwargs"]["content"]
            for output in data
        ]
es_client = Elasticsearch(hosts="http://localhost:9200")
set_llm_cache(
    SearchableElasticsearchCache(es_connection=es_client, index_name="llm-chat-cache")
)
```

在覆盖映射和文档构建时，请仅进行增量修改，保持基本映射不变。

## 可选缓存

您还可以选择关闭特定 LLM 的缓存。在下面的示例中，即使全局缓存已启用，我们也会关闭特定 LLM 的缓存。

```python
llm = OpenAI(model_name="gpt-3.5-turbo-instruct", n=2, best_of=2, cache=False)
```
```python
%%time
llm("Tell me a joke")
```
```output
CPU times: user 5.8 ms, sys: 2.71 ms, total: 8.51 ms
Wall time: 745 ms
```
```output
'\n\n为什么小鸡要过马路？\n\n为了到达另一边！'
```
```python
%%time
llm("Tell me a joke")
```
```output
CPU times: user 4.91 ms, sys: 2.64 ms, total: 7.55 ms
Wall time: 623 ms
```
```output
'\n\n两个人偷了一个日历。他们各自获得了六个月。'
```

## 链中的可选缓存

您还可以为链中的特定节点关闭缓存。请注意，由于某些接口的原因，通常更容易先构建链，然后再编辑 LLM。

作为示例，我们将加载一个总结器映射-减少链。我们将为映射步骤缓存结果，但在合并步骤中不冻结它。

```python
llm = OpenAI(model_name="gpt-3.5-turbo-instruct")
no_cache_llm = OpenAI(model_name="gpt-3.5-turbo-instruct", cache=False)
```
```python
from langchain_text_splitters import CharacterTextSplitter
text_splitter = CharacterTextSplitter()
```
```python
with open("../../how_to/state_of_the_union.txt") as f:
    state_of_the_union = f.read()
texts = text_splitter.split_text(state_of_the_union)
```
```python
from langchain_core.documents import Document
docs = [Document(page_content=t) for t in texts[:3]]
from langchain.chains.summarize import load_summarize_chain
```
```python
chain = load_summarize_chain(llm, chain_type="map_reduce", reduce_llm=no_cache_llm)
```
```python
%%time
chain.run(docs)
```
```output
CPU times: user 452 ms, sys: 60.3 ms, total: 512 ms
Wall time: 5.09 s
```
```output
'\n\n拜登总统正在讨论美国拯救计划和两党基础设施法案，这将创造就业机会并帮助美国人。他还谈到了他对美国的愿景，其中包括投资于教育和基础设施。针对俄罗斯在乌克兰的侵略，美国正在与欧洲盟友一起实施制裁并孤立俄罗斯。美国部队正在动员起来，以保护北约国家，以防普京决定继续向西推进。乌克兰人正在勇敢地反击，但接下来的几周对他们来说将是艰难的。普京最终将为他的行动付出高昂的代价。美国人不应感到惊慌，因为美国正在采取行动保护自己的利益和盟友。'
```

再次运行时，我们看到它运行速度大大加快，但最终答案不同。这是由于在映射步骤进行缓存，但在减少步骤不进行缓存。

```python
%%time
chain.run(docs)
```
```output
CPU times: user 11.5 ms, sys: 4.33 ms, total: 15.8 ms
Wall time: 1.04 s
```
```output
'\n\n拜登总统正在讨论美国拯救计划和两党基础设施法案，这将创造就业机会并帮助美国人。他还谈到了他对美国的愿景，其中包括投资于教育和基础设施。'
```
```python
!rm .langchain.db sqlite.db
```

## OpenSearch 语义缓存

使用 [OpenSearch](https://python.langchain.com/docs/integrations/vectorstores/opensearch/) 作为语义缓存，缓存提示和响应，并根据语义相似性评估命中。

```
```python
from langchain_openai import OpenAIEmbeddings
set_llm_cache(
    OpenSearchSemanticCache(
        opensearch_url="http://localhost:9200", embedding=OpenAIEmbeddings()
    )
)
```
```python
%%time
# 第一次调用，因为还未缓存，所以需要更长的时间
llm("告诉我一个笑话")
```
```output

CPU times: user 39.4 ms, sys: 11.8 ms, total: 51.2 ms

Wall time: 1.55 s

```
```output

"\n\n为什么科学家不相信原子？\n\n因为它们组成了一切。"

```
```python
%%time
# 第二次调用，虽然不是直接命中，但问题在语义上与原始问题相似，
# 所以它使用了缓存结果！
llm("告诉我一个笑话")
```
```output

CPU times: user 4.66 ms, sys: 1.1 ms, total: 5.76 ms

Wall time: 113 ms

```
```output

"\n\n为什么科学家不相信原子？\n\n因为它们组成了一切。"

```
```