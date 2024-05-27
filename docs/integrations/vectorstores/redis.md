# Redis
[Redis 矢量数据库](https://redis.io/docs/get-started/vector-database/) 介绍和 Langchain 集成指南。
## 什么是 Redis？
大多数来自 Web 服务背景的开发人员都熟悉 `Redis`。在其核心，`Redis` 是一个开源的键值存储，用作缓存、消息代理和数据库。开发人员选择 `Redis` 是因为它快速，拥有庞大的客户端库生态系统，并且多年来已被主要企业部署使用。
除了这些传统用例，`Redis` 还提供了额外的功能，比如搜索和查询功能，允许用户在 `Redis` 中创建二级索引结构。这使得 `Redis` 成为一个矢量数据库，速度快如缓存。
## Redis 作为矢量数据库
`Redis` 使用压缩的倒排索引进行快速索引，并且内存占用低。它还支持许多高级功能，比如：
- 在 Redis 哈希和 JSON 中对多个字段进行索引
- 矢量相似度搜索（使用 HNSW（ANN）或 FLAT（KNN））
- 矢量范围搜索（例如，查找查询向量半径内的所有向量）
- 增量索引而不会降低性能
- 文档排名（使用 [tf-idf](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)，可选用户提供的权重）
- 字段加权
- 复杂布尔查询，包括 AND、OR 和 NOT 运算符
- 前缀匹配、模糊匹配和精确短语查询
- 支持 [double-metaphone 音标匹配](https://redis.io/docs/stack/search/reference/phonetic_matching/)
- 自动完成建议（带有模糊前缀建议）
- 基于 [Snowball](http://snowballstem.org/) 的 [多种语言](https://redis.io/docs/stack/search/reference/stemming/) 的词干扩展
- 支持中文分词和查询（使用 [Friso](https://github.com/lionsoul2014/friso)）
- 数字过滤器和范围
- 使用 Redis 地理空间索引进行地理空间搜索
- 强大的聚合引擎
- 支持所有 utf-8 编码文本
- 检索完整文档、选定字段或仅文档 ID
- 对结果进行排序（例如，按创建日期）
## 客户端
由于 `Redis` 不仅仅是一个矢量数据库，通常有一些用例需要除了 `LangChain` 集成之外使用 `Redis` 客户端。您可以使用任何标准的 `Redis` 客户端库来运行搜索和查询命令，但最简单的方法是使用包装了搜索和查询 API 的库。以下是一些示例，但您可以在[这里](https://redis.io/resources/clients/)找到更多客户端库。
| 项目 | 语言 | 许可证 | 作者 | Stars |
|----------|---------|--------|---------|-------|
| [jedis][jedis-url] | Java | MIT | [Redis][redis-url] | ![Stars][jedis-stars] |
| [redisvl][redisvl-url] | Python | MIT | [Redis][redis-url] | ![Stars][redisvl-stars] |
| [redis-py][redis-py-url] | Python | MIT | [Redis][redis-url] | ![Stars][redis-py-stars] |
| [node-redis][node-redis-url] | Node.js | MIT | [Redis][redis-url] | ![Stars][node-redis-stars] |
| [nredisstack][nredisstack-url] | .NET | MIT | [Redis][redis-url] | ![Stars][nredisstack-stars] |
[redis-url]: https://redis.com
[redisvl-url]: https://github.com/RedisVentures/redisvl
[redisvl-stars]: https://img.shields.io/github/stars/RedisVentures/redisvl.svg?style=social&amp;label=Star&amp;maxAge=2592000
[redisvl-package]: https://pypi.python.org/pypi/redisvl
[redis-py-url]: https://github.com/redis/redis-py
[redis-py-stars]: https://img.shields.io/github/stars/redis/redis-py.svg?style=social&amp;label=Star&amp;maxAge=2592000
[redis-py-package]: https://pypi.python.org/pypi/redis
[jedis-url]: https://github.com/redis/jedis
[jedis-stars]: https://img.shields.io/github/stars/redis/jedis.svg?style=social&amp;label=Star&amp;maxAge=2592000
[Jedis-package]: https://search.maven.org/artifact/redis.clients/jedis
[nredisstack-url]: https://github.com/redis/nredisstack
[nredisstack-stars]: https://img.shields.io/github/stars/redis/nredisstack.svg?style=social&amp;label=Star&amp;maxAge=2592000
[nredisstack-package]: https://www.nuget.org/packages/nredisstack/
[node-redis-url]: https://github.com/redis/node-redis
[node-redis-stars]: https://img.shields.io/github/stars/redis/node-redis.svg?style=social&amp;label=Star&amp;maxAge=2592000
[node-redis-package]: https://www.npmjs.com/package/redis
[redis-om-python-url]: https://github.com/redis/redis-om-python
[redis-om-python-author]: https://redis.com
[redis-om-python-stars]: https://img.shields.io/github/stars/redis/redis-om-python.svg?style=social&amp;label=Star&amp;maxAge=2592000
[redisearch-go-url]: https://github.com/RediSearch/redisearch-go
[redisearch-go-author]: https://redis.com
[redisearch-go-stars]: https://img.shields.io/github/stars/RediSearch/redisearch-go.svg?style=social&amp;label=Star&amp;maxAge=2592000
[redisearch-api-rs-url]: https://github.com/RediSearch/redisearch-api-rs
## 部署选项
有许多种方法可以部署带有 RediSearch 的 Redis。最简单的方法是使用 Docker 来启动，但也有许多其他部署选项，例如：
- [Redis Cloud](https://redis.com/redis-enterprise-cloud/overview/)
- [Docker (Redis Stack)](https://hub.docker.com/r/redis/redis-stack)
- 云市场：[AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-e6y7ork67pjwg?sr=0-2&ref_=beagle&applicationId=AWSMPContessa)、[Google Marketplace](https://console.cloud.google.com/marketplace/details/redislabs-public/redis-enterprise?pli=1) 或 [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/garantiadata.redis_enterprise_1sp_public_preview?tab=Overview)
- 自建部署：[Redis Enterprise Software](https://redis.com/redis-enterprise-software/overview/)
- Kubernetes：[Redis Enterprise Software on Kubernetes](https://docs.redis.com/latest/kubernetes/)
## 更多示例
在 [Redis AI 团队的 GitHub](https://github.com/RedisVentures/) 上可以找到许多示例，例如：
- [Awesome Redis AI Resources](https://github.com/RedisVentures/redis-ai-resources) - 展示了在 AI 工作负载中使用 Redis 的示例列表
- [Azure OpenAI Embeddings Q&A](https://github.com/ruoccofabrizio/azure-open-ai-embeddings-qna) - 在 Azure 上使用 OpenAI 和 Redis 作为问答服务
- [ArXiv Paper Search](https://github.com/RedisVentures/redis-arXiv-search) - 对 arXiv 学术论文进行语义搜索
- [Vector Search on Azure](https://learn.microsoft.com/azure/azure-cache-for-redis/cache-tutorial-vector-similarity) - 在 Azure 上使用 Azure Cache for Redis 和 Azure OpenAI 进行向量搜索
## 更多资源
要了解如何将 Redis 用作向量数据库的更多信息，请查看以下资源：
- [RedisVL 文档](https://redisvl.com) - Redis Vector Library 客户端的文档
- [Redis Vector Similarity 文档](https://redis.io/docs/stack/search/reference/vectors/) - Redis 官方的向量搜索文档
- [Redis-py 搜索文档](https://redis.readthedocs.io/en/latest/redismodules.html#redisearch-commands) - redis-py 客户端库的文档
- [Vector Similarity Search: From Basics to Production](https://mlops.community/vector-similarity-search-from-basics-to-production/) - 介绍 VSS 和 Redis 作为 VectorDB 的博客文章
## 设置
### 安装 Redis Python 客户端
`Redis-py` 是 Redis 的官方支持客户端。最近发布的 `RedisVL` 客户端专为向量数据库用例而构建。两者都可以使用 pip 安装。
```python
%pip install --upgrade --quiet  redis redisvl langchain-openai tiktoken
```
我们希望使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。
```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```python
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings()
```
### 本地部署 Redis
要在本地部署 Redis，请运行：
```console
docker run -d -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```
如果一切正常运行，您应该可以在 `http://localhost:8001` 上看到一个漂亮的 Redis UI。有关其他部署方式，请参阅上面的[部署选项](#deployment-options)部分。
### Redis 连接 URL 模式
有效的 Redis 连接 URL 模式包括：
1. `redis://`  - 连接到 Redis 独立实例，未加密
2. `rediss://` - 连接到带有 TLS 加密的 Redis 独立实例
3. `redis+sentinel://`  - 通过 Redis Sentinel 连接到 Redis 服务器，未加密
4. `rediss+sentinel://` - 通过 Redis Sentinel 连接到 Redis 服务器，两者都使用 TLS 加密
有关其他连接参数的更多信息，请参阅[redis-py 文档](https://redis-py.readthedocs.io/en/stable/connections.html)。 
```python
# 连接到本地的 Redis 独立实例，数据库 0，无密码
redis_url = "redis://localhost:6379"
# 连接到主机名为 "redis" 端口为 7379，使用数据库 2 和密码 "secret"（旧的身份验证方案，不带用户名/6.x 之前的版本）
redis_url = "redis://:secret@redis:7379/2"
# 连接到主机名为 redis 默认端口，使用用户名 "joe"，密码 "secret"，使用 Redis 6+ ACL
redis_url = "redis://joe:secret@redis/0"
# 连接到本地的 Sentinel，默认组为 mymaster，数据库 0，无密码
redis_url = "redis+sentinel://localhost:26379"
# 连接到主机名为 redis 默认端口 26379，用户名 "joe"，密码 "secret"，默认组为 mymaster，数据库 0
redis_url = "redis+sentinel://joe:secret@redis"
# 连接到 Sentinel，无认证，监视组为 "zone-1"，数据库 2
redis_url = "redis+sentinel://redis:26379/zone-1/2"
# 连接到本地的 Redis 独立实例，数据库 0，无密码，但支持 TLS
redis_url = "rediss://localhost:6379"
# 连接到本地的 Sentinel，默认端口，数据库 0，无密码，但 Sentinel 和 Redis 服务器都支持 TLS
redis_url = "rediss+sentinel://localhost"
```
### 样本数据
首先，我们将描述一些样本数据，以便演示 Redis 向量存储的各种属性。
```python
metadata = [
    {
        "user": "john",
        "age": 18,
        "job": "engineer",
        "credit_score": "high",
    },
    {
        "user": "derrick",
        "age": 45,
        "job": "doctor",
        "credit_score": "low",
    },
    {
        "user": "nancy",
        "age": 94,
        "job": "doctor",
        "credit_score": "high",
    },
    {
        "user": "tyler",
        "age": 100,
        "job": "engineer",
        "credit_score": "high",
    },
    {
        "user": "joe",
        "age": 35,
        "job": "dentist",
        "credit_score": "medium",
    },
]
texts = ["foo", "foo", "foo", "bar", "bar"]
```
### 创建 Redis 向量存储
Redis VectorStore 实例可以通过多种方式进行初始化。有多个类方法可用于初始化 Redis VectorStore 实例。
- ``Redis.__init__`` - 直接初始化
- ``Redis.from_documents`` - 从 ``Langchain.docstore.Document`` 对象列表初始化
- ``Redis.from_texts`` - 从文本列表初始化（可选包含元数据）
- ``Redis.from_texts_return_keys`` - 从文本列表初始化（可选包含元数据）并返回键
- ``Redis.from_existing_index`` - 从现有的 Redis 索引初始化
下面我们将使用 ``Redis.from_texts`` 方法。
```python
from langchain_community.vectorstores.redis import Redis
rds = Redis.from_texts(
    texts,
    embeddings,
    metadatas=metadata,
    redis_url="redis://localhost:6379",
    index_name="users",
)
```
```python
rds.index_name
```
```output
'users'
```
## 检查创建的索引
一旦创建了 ``Redis`` VectorStore 对象，如果索引尚不存在，则 Redis 中将创建一个索引。可以使用 ``rvl`` 和 ``redis-cli`` 命令行工具来检查索引。如果您在上面安装了 ``redisvl``，则可以使用 ``rvl`` 命令行工具来检查索引。
```python
# 假设您在本地运行 Redis（使用 --host、--port、--password、--username 来更改此设置）
!rvl index listall
```
```output
16:58:26 [RedisVL] INFO   Indices:
16:58:26 [RedisVL] INFO   1. users
```
``Redis`` VectorStore 实现将尝试为通过 ``from_texts``、``from_texts_return_keys`` 和 ``from_documents`` 方法传递的任何元数据生成索引模式（用于过滤的字段）。这样，传递的任何元数据都将被索引到 Redis 搜索索引中，从而允许对这些字段进行过滤。
下面我们展示了从上面定义的元数据中创建的字段。
```python
!rvl index info -i users
```
```output
Index Information:
╭──────────────┬────────────────┬───────────────┬─────────────────┬────────────╮
│ Index Name   │ Storage Type   │ Prefixes      │ Index Options   │   Indexing │
├──────────────┼────────────────┼───────────────┼─────────────────┼────────────┤
│ users        │ HASH           │ ['doc:users'] │ []              │          0 │
╰──────────────┴────────────────┴───────────────┴─────────────────┴────────────╯
Index Fields:
╭────────────────┬────────────────┬─────────┬────────────────┬────────────────╮
│ Name           │ Attribute      │ Type    │ Field Option   │   Option Value │
├────────────────┼────────────────┼─────────┼────────────────┼────────────────┤
│ user           │ user           │ TEXT    │ WEIGHT         │              1 │
│ job            │ job            │ TEXT    │ WEIGHT         │              1 │
│ credit_score   │ credit_score   │ TEXT    │ WEIGHT         │              1 │
│ content        │ content        │ TEXT    │ WEIGHT         │              1 │
│ age            │ age            │ NUMERIC │                │                │
│ content_vector │ content_vector │ VECTOR  │                │                │
╰────────────────┴────────────────┴─────────┴────────────────┴────────────────╯
```
```python
!rvl stats -i users
```
```output
Statistics:
╭─────────────────────────────┬─────────────╮
│ Stat Key                    │ Value       │
├─────────────────────────────┼─────────────┤
│ num_docs                    │ 5           │
│ num_terms                   │ 15          │
│ max_doc_id                  │ 5           │
│ num_records                 │ 33          │
│ percent_indexed             │ 1           │
│ hash_indexing_failures      │ 0           │
│ number_of_uses              │ 4           │
│ bytes_per_record_avg        │ 4.60606     │
│ doc_table_size_mb           │ 0.000524521 │
│ inverted_sz_mb              │ 0.000144958 │
│ key_table_size_mb           │ 0.000193596 │
│ offset_bits_per_record_avg  │ 8           │
│ offset_vectors_sz_mb        │ 2.19345e-05 │
│ offsets_per_term_avg        │ 0.69697     │
```
│ 平均每个文档的记录数 │ 6.6         │
│ 可排序值的大小（MB） │ 0           │
│ 总索引时间 │ 0.32        │
│ 总倒排索引块数 │ 16          │
│ 向量索引大小（MB） │ 6.0126      │
╰─────────────────────────────┴─────────────╯
需要注意的是，我们没有指定元数据中的 ``user``、``job``、``credit_score`` 和 ``age`` 应该是索引中的字段，这是因为 ``Redis`` 的 VectorStore 对象会根据传入的元数据自动生成索引模式。有关索引字段生成的更多信息，请参阅 API 文档。
## 查询
根据您的用例，可以使用多种方式查询 ``Redis`` 的 VectorStore 实现：
- ``similarity_search``：查找与给定向量最相似的向量。
- ``similarity_search_with_score``：查找与给定向量最相似的向量，并返回向量距离。
- ``similarity_search_limit_score``：查找与给定向量最相似的向量，并将结果数量限制为 ``score_threshold``。
- ``similarity_search_with_relevance_scores``：查找与给定向量最相似的向量，并返回向量相似度。
- ``max_marginal_relevance_search``：查找与给定向量最相似的向量，同时优化多样性。
```python
results = rds.similarity_search("foo")
print(results[0].page_content)
```
```output
foo
```
```python
# 返回元数据
results = rds.similarity_search("foo", k=3)
meta = results[1].metadata
print("Redis 中文档的键：", meta.pop("id"))
print("文档的元数据：", meta)
```
```output
Redis 中文档的键： doc:users:a70ca43b3a4e4168bae57c78753a200f
文档的元数据： {'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'}
```
```python
# 带有分数（距离）的查询结果
results = rds.similarity_search_with_score("foo", k=5)
for result in results:
    print(f"内容：{result[0].page_content} --- 分数：{result[1]}")
```
```output
内容：foo --- 分数：0.0
内容：foo --- 分数：0.0
内容：foo --- 分数：0.0
内容：bar --- 分数：0.1566
内容：bar --- 分数：0.1566
```
```python
# 限制返回的向量距离
results = rds.similarity_search_with_score("foo", k=5, distance_threshold=0.1)
for result in results:
    print(f"内容：{result[0].page_content} --- 分数：{result[1]}")
```
```output
内容：foo --- 分数：0.0
内容：foo --- 分数：0.0
内容：foo --- 分数：0.0
```
```python
# 带有分数的查询结果
results = rds.similarity_search_with_relevance_scores("foo", k=5)
for result in results:
    print(f"内容：{result[0].page_content} --- 相似度：{result[1]}")
```
```output
内容：foo --- 相似度：1.0
内容：foo --- 相似度：1.0
内容：foo --- 相似度：1.0
内容：bar --- 相似度：0.8434
内容：bar --- 相似度：0.8434
```
```python
# 限制分数（相似度必须大于0.9）
results = rds.similarity_search_with_relevance_scores("foo", k=5, score_threshold=0.9)
for result in results:
    print(f"内容：{result[0].page_content} --- 相似度：{result[1]}")
```
```output
内容：foo --- 相似度：1.0
内容：foo --- 相似度：1.0
内容：foo --- 相似度：1.0
```
```python
# 还可以按以下方式添加新文档
new_document = ["baz"]
new_metadata = [{"user": "sam", "age": 50, "job": "janitor", "credit_score": "high"}]
# 文档和元数据都必须是列表
rds.add_texts(new_document, new_metadata)
```
```output
['doc:users:b9c71d62a0a34241a37950b448dafd38']
```
```python
# 现在查询新文档
results = rds.similarity_search("baz", k=3)
print(results[0].metadata)
```
```output
{'id': 'doc:users:b9c71d62a0a34241a37950b448dafd38', 'user': 'sam', 'job': 'janitor', 'credit_score': 'high', 'age': '50'}
```
```python
# 使用最大边际相关性搜索来增加结果的多样性
results = rds.max_marginal_relevance_search("foo")
```
```python
# lambda_mult 参数控制结果的多样性，值越低，多样性越高
results = rds.max_marginal_relevance_search("foo", lambda_mult=0.1)
```
## 连接到现有索引
为了在使用 ``Redis`` 的 VectorStore 时具有相同的索引元数据，您需要将相同的 ``index_schema`` 作为路径到 yaml 文件或字典传递。以下是如何从索引中获取模式并连接到现有索引的示例。
```python
# 将模式写入 yaml 文件
rds.write_schema("redis_schema.yaml")
```
此示例的模式文件应如下所示：
```yaml
numeric:
- name: age
  no_index: false
  sortable: false
text:
- name: user
  no_index: false
  no_stem: false
  sortable: false
  weight: 1
  withsuffixtrie: false
- name: job
  no_index: false
  no_stem: false
  sortable: false
  weight: 1
  withsuffixtrie: false
- name: 信用分数
  no_index: false
  no_stem: false
  sortable: false
  weight: 1
  withsuffixtrie: false
- name: 内容
  no_index: false
  no_stem: false
  sortable: false
  weight: 1
  withsuffixtrie: false
vector:
- algorithm: FLAT
  block_size: 1000
  datatype: FLOAT32
  dims: 1536
  distance_metric: COSINE
  initial_cap: 20000
  name: 内容向量
```
**注意**，这包括模式的**所有**可能字段。您可以删除不需要的任何字段。
```python
# 现在我们可以连接到现有的索引如下
new_rds = Redis.from_existing_index(
    embeddings,
    index_name="users",
    redis_url="redis://localhost:6379",
    schema="redis_schema.yaml",
)
results = new_rds.similarity_search("foo", k=3)
print(results[0].metadata)
```
```output
{'id': 'doc:users:8484c48a032d4c4cbe3cc2ed6845fabb', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}
```
```python
# 查看模式是否相同
new_rds.schema == rds.schema
```
```output
True
```
## 自定义元数据索引
在某些情况下，您可能希望控制元数据映射到哪些字段。例如，您可能希望``credit_score``字段是一个分类字段，而不是一个文本字段（这是所有字符串字段的默认行为）。在这种情况下，您可以在上述每个初始化方法中使用``index_schema``参数来指定索引的模式。自定义索引模式可以作为字典或作为指向 YAML 文件的路径传递。
模式中的所有参数都有默认值，除了名称，因此您只需指定要更改的字段。所有名称都对应于您在命令行中使用``redis-cli``或在``redis-py``中使用的参数的蛇形/小写版本。有关每个字段的参数的更多信息，请参阅[文档](https://redis.io/docs/interact/search-and-query/basic-constructs/field-and-type-options/)
以下示例显示了如何将``credit_score``字段的模式指定为标签（分类）字段，而不是文本字段。
```yaml
# index_schema.yml
tag:
    - name: credit_score
text:
    - name: user
    - name: job
numeric:
    - name: age
```
在 Python 中，这看起来像：
```python
index_schema = {
    "tag": [{"name": "credit_score"}],
    "text": [{"name": "user"}, {"name": "job"}],
    "numeric": [{"name": "age"}],
}
```
请注意，只需指定``name``字段即可。所有其他字段都有默认值。
```python
# 使用上述定义的新模式创建一个新索引
index_schema = {
    "tag": [{"name": "credit_score"}],
    "text": [{"name": "user"}, {"name": "job"}],
    "numeric": [{"name": "age"}],
}
rds, keys = Redis.from_texts_return_keys(
    texts,
    embeddings,
    metadatas=metadata,
    redis_url="redis://localhost:6379",
    index_name="users_modified",
    index_schema=index_schema,  # 传入新的索引模式
)
```
```output
`index_schema` does not match generated metadata schema.
If you meant to manually override the schema, please ignore this message.
index_schema: {'tag': [{'name': 'credit_score'}], 'text': [{'name': 'user'}, {'name': 'job'}], 'numeric': [{'name': 'age'}]}
generated_schema: {'text': [{'name': 'user'}, {'name': 'job'}, {'name': 'credit_score'}], 'numeric': [{'name': 'age'}], 'tag': []}
```
上述警告旨在在用户覆盖默认行为时通知用户。如果您有意覆盖行为，请忽略此警告。
## 混合过滤
通过内置于 LangChain 中的 Redis Filter 表达式语言，您可以创建任意长的混合过滤器链，用于过滤搜索结果。表达语言源自[RedisVL 表达式语法](https://redisvl.com)，旨在易于使用和理解。
以下是可用的过滤器类型：
- ``RedisText``：根据元数据字段进行全文搜索进行过滤。支持精确、模糊和通配符匹配。
- ``RedisNum``：根据元数据字段的数值范围进行过滤。
- ``RedisTag``：根据基于字符串的分类元数据字段的精确匹配进行过滤。可以指定多个标签，如“tag1，tag2，tag3”。
以下是利用这些过滤器的示例。
```python
from langchain_community.vectorstores.redis import RedisText, RedisNum, RedisTag
# 精确匹配
has_high_credit = RedisTag("credit_score") == "high"
does_not_have_high_credit = RedisTag("credit_score") != "low"
# 模糊匹配
job_starts_with_eng = RedisText("job") % "eng*"
job_is_engineer = RedisText("job") == "engineer"
job_is_not_engineer = RedisText("job") != "engineer"
# 数值过滤
age_is_18 = RedisNum("age") == 18
age_is_not_18 = RedisNum("age") != 18
age_is_greater_than_18 = RedisNum("age") > 18
age_is_less_than_18 = RedisNum("age") < 18
```
`RedisFilter` 类可以用于简化这些过滤器的导入，如下所示：
```python
from langchain_community.vectorstores.redis import RedisFilter
# 与上面相同的示例
has_high_credit = RedisFilter.tag("credit_score") == "high"
does_not_have_high_credit = RedisFilter.num("age") > 8
job_starts_with_eng = RedisFilter.text("job") % "eng*"
```
以下是使用混合过滤器进行搜索的示例：
```python
from langchain_community.vectorstores.redis import RedisText
is_engineer = RedisText("job") == "engineer"
results = rds.similarity_search("foo", k=3, filter=is_engineer)
print("Job:", results[0].metadata["job"])
print("Engineers in the dataset:", len(results))
```
输出结果为：
```
Job: engineer
Engineers in the dataset: 2
```
```python
# 模糊匹配
starts_with_doc = RedisText("job") % "doc*"
results = rds.similarity_search("foo", k=3, filter=starts_with_doc)
for result in results:
    print("Job:", result.metadata["job"])
print("Jobs in dataset that start with 'doc':", len(results))
```
输出结果为：
```
Job: doctor
Job: doctor
Jobs in dataset that start with 'doc': 2
```
```python
from langchain_community.vectorstores.redis import RedisNum
is_over_18 = RedisNum("age") > 18
is_under_99 = RedisNum("age") < 99
age_range = is_over_18 & is_under_99
results = rds.similarity_search("foo", filter=age_range)
for result in results:
    print("User:", result.metadata["user"], "is", result.metadata["age"])
```
输出结果为：
```
User: derrick is 45
User: nancy is 94
User: joe is 35
```
```python
# 在构造过滤器时，确保在 FilterExpressions 周围使用括号
age_range = (RedisNum("age") > 18) & (RedisNum("age") < 99)
results = rds.similarity_search("foo", filter=age_range)
for result in results:
    print("User:", result.metadata["user"], "is", result.metadata["age"])
```
输出结果为：
```
User: derrick is 45
User: nancy is 94
User: joe is 35
```
## Redis 作为检索器
下面我们将介绍使用向量存储作为检索器的不同选项。
有三种不同的搜索方法可以用于检索。默认情况下，它将使用语义相似性。
```python
query = "foo"
results = rds.similarity_search_with_score(query, k=3, return_metadata=True)
for result in results:
    print("Content:", result[0].page_content, " --- Score: ", result[1])
```
输出结果为：
```
Content: foo  --- Score:  0.0
Content: foo  --- Score:  0.0
Content: foo  --- Score:  0.0
```
```python
retriever = rds.as_retriever(search_type="similarity", search_kwargs={"k": 4})
```
```python
docs = retriever.invoke(query)
docs
```
输出结果为：
```
[Document(page_content='foo', metadata={'id': 'doc:users_modified:988ecca7574048e396756efc0e79aeca', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:009b1afeb4084cc6bdef858c7a99b48e', 'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:7087cee9be5b4eca93c30fbdd09a2731', 'user': 'nancy', 'job': 'doctor', 'credit_score': 'high', 'age': '94'}),
 Document(page_content='bar', metadata={'id': 'doc:users_modified:01ef6caac12b42c28ad870aefe574253', 'user': 'tyler', 'job': 'engineer', 'credit_score': 'high', 'age': '100'})]
```
还有 `similarity_distance_threshold` 检索器，允许用户指定向量距离：
```python
retriever = rds.as_retriever(
    search_type="similarity_distance_threshold",
    search_kwargs={"k": 4, "distance_threshold": 0.1},
)
```
```python
docs = retriever.invoke(query)
docs
```
输出结果为：
```
[Document(page_content='foo', metadata={'id': 'doc:users_modified:988ecca7574048e396756efc0e79aeca', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:009b1afeb4084cc6bdef858c7a99b48e', 'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:7087cee9be5b4eca93c30fbdd09a2731', 'user': 'nancy', 'job': 'doctor', 'credit_score': 'high', 'age': '94'})]
```
最后，`similarity_score_threshold` 允许用户定义相似文档的最小分数：
```python
retriever = rds.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"score_threshold": 0.9, "k": 10},
)
```
```python
retriever.invoke("foo")
```
输出结果为：
[Document(page_content='foo', metadata={'id': 'doc:users_modified:988ecca7574048e396756efc0e79aeca', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}),
```python
retriever = rds.as_retriever(
    search_type="mmr", search_kwargs={"fetch_k": 20, "k": 4, "lambda_mult": 0.1}
)
```
```python
retriever.invoke("foo")
```
```output
[Document(page_content='foo', metadata={'id': 'doc:users:8f6b673b390647809d510112cde01a27', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}),
 Document(page_content='bar', metadata={'id': 'doc:users:93521560735d42328b48c9c6f6418d6a', 'user': 'tyler', 'job': 'engineer', 'credit_score': 'high', 'age': '100'}),
 Document(page_content='foo', metadata={'id': 'doc:users:125ecd39d07845eabf1a699d44134a5b', 'user': 'nancy', 'job': 'doctor', 'credit_score': 'high', 'age': '94'}),
 Document(page_content='foo', metadata={'id': 'doc:users:d6200ab3764c466082fde3eaab972a2a', 'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'})]
```
## 删除键和索引
要删除条目，您必须通过它们的键来引用它们。
```python
Redis.delete(keys, redis_url="redis://localhost:6379")
```
```output
True
```
```python
# 同时删除索引
Redis.drop_index(
    index_name="users", delete_documents=True, redis_url="redis://localhost:6379"
)
Redis.drop_index(
    index_name="users_modified",
    delete_documents=True,
    redis_url="redis://localhost:6379",
)
```
```output
True
```
