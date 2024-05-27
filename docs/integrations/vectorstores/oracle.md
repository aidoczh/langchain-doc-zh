# Oracle AI Vector Search: Vector Store

Oracle AI Vector Search 是专为人工智能（AI）工作负载设计的，允许您基于语义而不是关键词查询数据。

Oracle AI Vector Search 最大的好处之一是，可以将非结构化数据的语义搜索与业务数据的关系搜索结合在一个单一系统中。

这不仅强大，而且更有效，因为您无需添加专门的向量数据库，消除了在多个系统之间的数据碎片化的痛苦。

此外，您的向量可以受益于 Oracle Database 的所有最强大的功能，比如以下内容：

- [分区支持](https://www.oracle.com/database/technologies/partitioning.html)

- [实时应用集群可伸缩性](https://www.oracle.com/database/real-application-clusters/)

- [Exadata 智能扫描](https://www.oracle.com/database/technologies/exadata/software/smartscan/)

- [地理分布式数据库中的 Shard 处理](https://www.oracle.com/database/distributed-database/)

- [事务](https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/transactions.html)

- [并行 SQL](https://docs.oracle.com/en/database/oracle/oracle-database/21/vldbg/parallel-exec-intro.html#GUID-D28717E4-0F77-44F5-BB4E-234C31D4E4BA)

- [灾难恢复](https://www.oracle.com/database/data-guard/)

- [安全性](https://www.oracle.com/security/database-security/)

- [Oracle 机器学习](https://www.oracle.com/artificial-intelligence/database-machine-learning/)

- [Oracle 图数据库](https://www.oracle.com/database/integrated-graph-database/)

- [Oracle 空间和图形](https://www.oracle.com/database/spatial/)

- [Oracle 区块链](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_blockchain_table.html#GUID-B469E277-978E-4378-A8C1-26D3FF96C9A6)

- [JSON](https://docs.oracle.com/en/database/oracle/oracle-database/23/adjsn/json-in-oracle-database.html)

如果您刚开始使用 Oracle Database，请考虑探索[免费的 Oracle 23 AI](https://www.oracle.com/database/free/#resources)，它提供了设置数据库环境的绝佳介绍。在使用数据库时，通常建议不要默认使用系统用户；相反，您可以为增强安全性和定制性创建自己的用户。有关用户创建的详细步骤，请参阅我们的[端到端指南](https://github.com/langchain-ai/langchain/blob/master/cookbook/oracleai_demo.ipynb)，该指南还展示了如何在 Oracle 中设置用户。此外，了解用户权限对于有效管理数据库安全至关重要。您可以在官方的[Oracle 指南](https://docs.oracle.com/en/database/oracle/oracle-database/19/admqs/administering-user-accounts-and-security.html#GUID-36B21D72-1BBB-46C9-A0C9-F0D2A8591B8D)中了解更多关于此主题的内容。

### 使用 Langchain 与 Oracle AI Vector Search 的先决条件

请安装 Oracle Python 客户端驱动程序以使用 Langchain 与 Oracle AI Vector Search。

```python
# pip install oracledb
```

### 连接到 Oracle AI Vector Search

以下示例代码将展示如何连接到 Oracle Database。默认情况下，python-oracledb 以“Thin”模式运行，直接连接到 Oracle Database。此模式不需要 Oracle 客户端库。但是，当 python-oracledb 使用它们时，会提供一些额外的功能。当 python-oracledb 使用 Oracle 客户端库时，它被称为“Thick”模式。这两种模式都具有全面的功能，支持 Python 数据库 API v2.0 规范。请参阅以下[指南](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_a.html#featuresummary)，了解每种模式支持的功能。如果无法使用 Thin 模式，您可能希望切换到 Thick 模式。

```python
import oracledb
username = "username"
password = "password"
dsn = "ipaddress:port/orclpdb1"
try:
    connection = oracledb.connect(user=username, password=password, dsn=dsn)
    print("连接成功！")
except Exception as e:
    print("连接失败！")
```

### 导入所需的依赖项以使用 Oracle AI Vector Search

```python
from langchain_community.vectorstores import oraclevs
from langchain_community.vectorstores.oraclevs import OracleVS
from langchain_community.vectorstores.utils import DistanceStrategy
from langchain_core.documents import Document
from langchain_huggingface import HuggingFaceEmbeddings
```

### 加载文档

```python
# 定义文档列表（以下示例是来自 Oracle 概念手册的 5 个随机文档）
documents_json_list = [
    {
        "id": "cncpt_15.5.3.2.2_P4",
        "text": "如果对任何前面的问题的答案是肯定的，那么数据库将停止搜索并从指定的表空间分配空间；否则，空间将从数据库默认的共享临时表空间分配。",
        "link": "https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/logical-storage-structures.html#GUID-5387D7B2-C0CA-4C1E-811B-C7EB9B636442",
    },
    {
        "id": "cncpt_15.5.5_P1",
        "text": "只要数据库是打开的，表空间就可以是在线的（可访问的）或离线的（不可访问的）。\n表空间通常是在线的，以便其数据对用户可用。SYSTEM 表空间和临时表空间不能被脱机。",
        "link": "https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/logical-storage-structures.html#GUID-D02B2220-E6F5-40D9-AFB5-BC69BCEF6CD4",
    },
    {
        "id": "cncpt_22.3.4.3.1_P2",
        "text": "数据库以与其他数据类型不同的方式存储 LOB。创建 LOB 列会隐式创建 LOB 段和 LOB 索引。包含 LOB 段和 LOB 索引的表空间（它们总是一起存储）可能与包含表的表空间不同。\n有时，数据库可以将少量 LOB 数据存储在表本身而不是在单独的 LOB 段中。",
        "link": "https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/concepts-for-database-developers.html#GUID-3C50EAB8-FC39-4BB3-B680-4EACCE49E866",
    },
    {
        "id": "cncpt_22.3.4.3.1_P3",
        "text": "LOB 段以称为块的片段存储数据。块是逻辑上连续的数据块的集合，是 LOB 的最小分配单元。表中存储一个称为 LOB 定位器的指针，它指向 LOB 索引。当查询表时，数据库使用 LOB 索引快速定位 LOB 块。",
        "link": "https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/concepts-for-database-developers.html#GUID-3C50EAB8-FC39-4BB3-B680-4EACCE49E866",
    },
]
```

```python
# 创建 Langchain 文档
documents_langchain = []
for doc in documents_json_list:
    metadata = {"id": doc["id"], "link": doc["link"]}
    doc_langchain = Document(page_content=doc["text"], metadata=metadata)
    documents_langchain.append(doc_langchain)
### 使用 AI 向量搜索创建不同距离度量的向量存储
首先，我们将创建三个具有不同距离函数的向量存储。由于我们尚未在其中创建索引，它们目前只会创建表。稍后，我们将使用这些向量存储来创建 HNSW 索引。要了解 Oracle AI 向量搜索支持的不同类型索引的更多信息，请参考以下[指南](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/manage-different-categories-vector-indexes.html)。
您可以手动连接到 Oracle 数据库，然后会看到三个表：Documents_DOT、Documents_COSINE 和 Documents_EUCLIDEAN。
然后，我们将创建三个额外的表 Documents_DOT_IVF、Documents_COSINE_IVF 和 Documents_EUCLIDEAN_IVF，这些表将用于在表上创建 IVF 索引，而不是 HNSW 索引。
```python

# 将文档摄入到 Oracle 向量存储中，使用不同的距离策略

# 在使用我们的 API 调用时，首先通过 from_documents() 初始化您的向量存储的一部分文档，然后使用 add_texts() 逐渐添加更多文档。

# 这种方法可以防止系统过载，并确保高效的文档处理。

model = HuggingFaceEmbeddings(model_name="sentence-transformers/all-mpnet-base-v2")

vector_store_dot = OracleVS.from_documents(

    documents_langchain,

    model,

    client=connection,

    table_name="Documents_DOT",

    distance_strategy=DistanceStrategy.DOT_PRODUCT,

)

vector_store_max = OracleVS.from_documents(

    documents_langchain,

    model,

    client=connection,

    table_name="Documents_COSINE",

    distance_strategy=DistanceStrategy.COSINE,

)

vector_store_euclidean = OracleVS.from_documents(

    documents_langchain,

    model,

    client=connection,

    table_name="Documents_EUCLIDEAN",

    distance_strategy=DistanceStrategy.EUCLIDEAN_DISTANCE,

)

# 使用不同的距离策略将文档摄入到 Oracle 向量存储中

vector_store_dot_ivf = OracleVS.from_documents(

    documents_langchain,

    model,

    client=connection,

    table_name="Documents_DOT_IVF",

    distance_strategy=DistanceStrategy.DOT_PRODUCT,

)

vector_store_max_ivf = OracleVS.from_documents(

    documents_langchain,

    model,

    client=connection,

    table_name="Documents_COSINE_IVF",

    distance_strategy=DistanceStrategy.COSINE,

)

vector_store_euclidean_ivf = OracleVS.from_documents(

    documents_langchain,

    model,

    client=connection,

    table_name="Documents_EUCLIDEAN_IVF",

    distance_strategy=DistanceStrategy.EUCLIDEAN_DISTANCE,

)

```
### 演示文本的添加和删除操作，以及基本的相似性搜索
```python
def manage_texts(vector_stores):
    """
    向每个向量存储添加文本，演示重复添加的错误处理，并执行文本的删除。展示每个向量存储的相似性搜索和索引创建。
    Args:
    - vector_stores (list): OracleVS 实例的列表。
    """
    texts = ["Rohan", "Shailendra"]
    metadata = [
        {"id": "100", "link": "Document Example Test 1"},
        {"id": "101", "link": "Document Example Test 2"},
    ]
    for i, vs in enumerate(vector_stores, start=1):
        # 添加文本
        try:
            vs.add_texts(texts, metadata)
            print(f"\n\n\n向量存储 {i} 的文本添加完成\n\n\n")
        except Exception as ex:
            print(f"\n\n\n向量存储 {i} 的重复添加预期错误\n\n\n")
        # 使用'id'的值删除文本
        vs.delete([metadata[0]["id"]])
        print(f"\n\n\n向量存储 {i} 的文本删除完成\n\n\n")
        # 相似性搜索
        results = vs.similarity_search("How are LOBS stored in Oracle Database", 2)
        print(f"\n\n\n向量存储 {i} 的相似性搜索结果：{results}\n\n\n")
vector_store_list = [
    vector_store_dot,
    vector_store_max,
    vector_store_euclidean,
    vector_store_dot_ivf,
    vector_store_max_ivf,
    vector_store_euclidean_ivf,
]
manage_texts(vector_store_list)
```
### 演示为每种距离策略创建具体参数的索引
```python
def create_search_indices(connection):
    """
    为向量存储创建搜索索引，每个索引都具有特定于其距离策略的特定参数。
    """
    # DOT_PRODUCT 策略的索引
    # 请注意，我们正在使用默认参数创建 HNSW 索引
```
# 创建索引
这将默认创建一个具有 8 个并行工作者并使用 Oracle AI Vector Search 默认精度的 HNSW 索引
```python
oraclevs.create_index(
    connection,
    vector_store_dot,
    params={"idx_name": "hnsw_idx1", "idx_type": "HNSW"},
)
```
用特定参数创建 COSINE 策略的索引
注意我们正在创建一个具有 16 个并行工作者和目标精度规定为 97% 的 HNSW 索引
```python
oraclevs.create_index(
    connection,
    vector_store_max,
    params={
        "idx_name": "hnsw_idx2",
        "idx_type": "HNSW",
        "accuracy": 97,
        "parallel": 16,
    },
)
```
用特定参数创建 EUCLIDEAN_DISTANCE 策略的索引
注意我们通过指定 Power User 参数（neighbors = 64 和 efConstruction = 100）来创建 HNSW 索引
```python
oraclevs.create_index(
    connection,
    vector_store_euclidean,
    params={
        "idx_name": "hnsw_idx3",
        "idx_type": "HNSW",
        "neighbors": 64,
        "efConstruction": 100,
    },
)
```
用特定参数创建 DOT_PRODUCT 策略的索引
注意我们正在创建一个具有默认参数的 IVF 索引
这将默认创建一个具有 8 个并行工作者并使用 Oracle AI Vector Search 默认精度的 IVF 索引
```python
oraclevs.create_index(
    connection,
    vector_store_dot_ivf,
    params={
        "idx_name": "ivf_idx1",
        "idx_type": "IVF",
    },
)
```
用特定参数创建 COSINE 策略的索引
注意我们正在创建一个具有 32 个并行工作者和目标精度规定为 90% 的 IVF 索引
```python
oraclevs.create_index(
    connection,
    vector_store_max_ivf,
    params={
        "idx_name": "ivf_idx2",
        "idx_type": "IVF",
        "accuracy": 90,
        "parallel": 32,
    },
)
```
用特定参数创建 EUCLIDEAN_DISTANCE 策略的索引
注意我们通过指定 Power User 参数（neighbor_part = 64）来创建 IVF 索引
```python
oraclevs.create_index(
    connection,
    vector_store_euclidean_ivf,
    params={"idx_name": "ivf_idx3", "idx_type": "IVF", "neighbor_part": 64},
)
```
打印"索引创建完成。"
创建搜索索引(connection)
```python
create_search_indices(connection)
```
### 展示在所有六个向量存储上进行高级搜索，包括有和没有属性过滤的情况 - 有过滤时，我们只选择文档 id 为 101，其他不选
```python
# 创建索引后进行高级搜索
def conduct_advanced_searches(vector_stores):
    query = "Oracle 数据库中 LOBS 是如何存储的"
    # 构建一个用于直接与文档元数据进行比较的过滤器
    # 此过滤器旨在包括其元数据'id'恰好为'2'的文档
    filter_criteria = {"id": ["101"]}  # 直接比较过滤器
    for i, vs in enumerate(vector_stores, start=1):
        print(f"\n--- 向量存储 {i} 高级搜索 ---")
        # 没有过滤器的相似度搜索
        print("\n没有过滤器的相似度搜索结果:")
        print(vs.similarity_search(query, 2))
        # 带过滤器的相似度搜索
        print("\n带过滤器的相似度搜索结果:")
        print(vs.similarity_search(query, 2, filter=filter_criteria))
        # 带相关性分数的相似度搜索
        print("\n带相关性分数的相似度搜索结果:")
        print(vs.similarity_search_with_score(query, 2))
        # 带过滤器的相关性分数的相似度搜索
        print("\n带过滤器的相关性分数的相似度搜索结果:")
        print(vs.similarity_search_with_score(query, 2, filter=filter_criteria))
        # 最大边际相关性搜索
        print("\n最大边际相关性搜索结果:")
        print(vs.max_marginal_relevance_search(query, 2, fetch_k=20, lambda_mult=0.5))
        # 带过滤器的最大边际相关性搜索结果
        print("\n带过滤器的最大边际相关性搜索结果:")
        print(
            vs.max_marginal_relevance_search(
                query, 2, fetch_k=20, lambda_mult=0.5, filter=filter_criteria
            )
        )
conduct_advanced_searches(vector_store_list)
```

### 全流程演示

请参阅我们的完整演示指南 [Oracle AI Vector Search 全流程演示指南](https://github.com/langchain-ai/langchain/tree/master/cookbook/oracleai_demo.ipynb) ，以借助 Oracle AI Vector Search 构建全流程 RAG 管道。