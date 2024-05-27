# Oracle AI Vector Search: 文档处理

Oracle AI Vector Search 专为人工智能（AI）工作负载设计，允许您基于语义而不是关键词查询数据。

Oracle AI Vector Search 最大的好处之一是，可以在一个单一系统中将非结构化数据的语义搜索与业务数据的关系搜索相结合。

这不仅强大，而且更加有效，因为您无需添加专门的向量数据库，消除了在多个系统之间的数据碎片化的痛苦。

此外，您的向量可以受益于 Oracle Database 的所有最强大的功能，比如以下内容：

* [分区支持](https://www.oracle.com/database/technologies/partitioning.html)

* [实时应用集群可扩展性](https://www.oracle.com/database/real-application-clusters/)

* [Exadata 智能扫描](https://www.oracle.com/database/technologies/exadata/software/smartscan/)

* [在地理分布式数据库之间进行分片处理](https://www.oracle.com/database/distributed-database/)

* [事务](https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/transactions.html)

* [并行 SQL](https://docs.oracle.com/en/database/oracle/oracle-database/21/vldbg/parallel-exec-intro.html#GUID-D28717E4-0F77-44F5-BB4E-234C31D4E4BA)

* [灾难恢复](https://www.oracle.com/database/data-guard/)

* [安全性](https://www.oracle.com/security/database-security/)

* [Oracle 机器学习](https://www.oracle.com/artificial-intelligence/database-machine-learning/)

* [Oracle 图数据库](https://www.oracle.com/database/integrated-graph-database/)

* [Oracle 空间和图形](https://www.oracle.com/database/spatial/)

* [Oracle 区块链](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_blockchain_table.html#GUID-B469E277-978E-4378-A8C1-26D3FF96C9A6)

* [JSON](https://docs.oracle.com/en/database/oracle/oracle-database/23/adjsn/json-in-oracle-database.html)

该指南演示了如何在 Oracle AI Vector Search 中使用文档处理功能，使用 OracleDocLoader 和 OracleTextSplitter 分别加载和分块文档。

如果您刚开始使用 Oracle Database，请考虑探索[免费的 Oracle 23 AI](https://www.oracle.com/database/free/#resources)，它提供了一个很好的介绍，帮助您设置数据库环境。在使用数据库时，通常建议不要默认使用系统用户；相反，您可以创建自己的用户以增强安全性和定制性。有关用户创建的详细步骤，请参阅我们的[端到端指南](https://github.com/langchain-ai/langchain/blob/master/cookbook/oracleai_demo.ipynb)，该指南还展示了如何在 Oracle 中设置用户。此外，了解用户权限对于有效管理数据库安全至关重要。您可以在官方的[Oracle指南](https://docs.oracle.com/en/database/oracle/oracle-database/19/admqs/administering-user-accounts-and-security.html#GUID-36B21D72-1BBB-46C9-A0C9-F0D2A8591B8D)中了解更多关于这个主题的内容。

### 先决条件

请安装 Oracle Python 客户端驱动程序以在 Oracle AI Vector Search 中使用 Langchain。 

```python
# pip install oracledb
```

### 连接到 Oracle Database

以下示例代码将展示如何连接到 Oracle Database。默认情况下，python-oracledb 以“Thin”模式运行，直接连接到 Oracle Database。此模式不需要 Oracle 客户端库。但是，当 python-oracledb 使用它们时，可以使用一些附加功能。当 Python-oracledb 使用 Oracle 客户端库时，它被称为“Thick”模式。这两种模式都具有全面的功能，支持 Python 数据库 API v2.0 规范。请参阅以下[指南](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_a.html#featuresummary)，了解每种模式支持的功能。如果无法使用“Thin”模式，您可能希望切换到“Thick”模式。

```python
import sys
import oracledb
# 请更新为您的用户名、密码、主机名和服务名
username = "<username>"
password = "<password>"
dsn = "<hostname>/<service_name>"
try:
    conn = oracledb.connect(user=username, password=password, dsn=dsn)
    print("连接成功！")
except Exception as e:
    print("连接失败！")
    sys.exit(1)
```

现在让我们创建一个表并插入一些示例文档进行测试。

```python
try:
    cursor = conn.cursor()
    drop_table_sql = """drop table if exists demo_tab"""
    cursor.execute(drop_table_sql)
    create_table_sql = """create table demo_tab (id number, data clob)"""
    cursor.execute(create_table_sql)
    insert_row_sql = """insert into demo_tab values (:1, :2)"""
    rows_to_insert = [
        (
            1,
            "如果对任何前面的问题的答案是肯定的，则数据库会停止搜索并从指定的表空间分配空间；否则，空间将从数据库默认的共享临时表空间分配。",
        ),
        (
            2,
            "只要数据库是打开的，表空间就可以是在线的（可访问的）或离线的（不可访问的）。\n通常情况下，表空间是在线的，以便其数据对用户可用。SYSTEM 表空间和临时表空间不能被脱机。",
        ),
        (
            3,
            "数据库以与其他数据类型不同的方式存储 LOB。创建 LOB 列隐式地创建了一个 LOB 段和一个 LOB 索引。包含 LOB 段和 LOB 索引的表空间总是存储在一起，可能与包含表的表空间不同。\n有时，数据库可以将少量 LOB 数据存储在表本身而不是在单独的 LOB 段中。",
        ),
    ]
    cursor.executemany(insert_row_sql, rows_to_insert)
    conn.commit()
    print("表已创建并填充。")
    cursor.close()
except Exception as e:
    print("表创建失败。")
    cursor.close()
    conn.close()
    sys.exit(1)
### 加载文档
用户可以通过适当配置加载器参数，灵活地从 Oracle 数据库、文件系统或两者同时加载文档。有关这些参数的详细信息，请参阅[Oracle AI 矢量搜索指南](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_vector_chain1.html#GUID-73397E89-92FB-48ED-94BB-1AD960C4EA1F)。
利用 OracleDocLoader 的一个重要优势是其能够处理超过 150 种不同的文件格式，消除了针对不同文档类型使用多个加载器的需要。有关支持格式的完整列表，请参阅[Oracle Text 支持的文档格式](https://docs.oracle.com/en/database/oracle/oracle-database/23/ccref/oracle-text-supported-document-formats.html)。
以下是一个演示如何使用 OracleDocLoader 的示例代码片段
```python

from langchain_community.document_loaders.oracleai import OracleDocLoader

from langchain_core.documents import Document

# 从本地文件加载

loader_params = {}

loader_params["file"] = "<file>"

# 从本地目录加载

loader_params = {}

loader_params["dir"] = "<directory>"

# 从 Oracle 数据库表加载

loader_params = {

    "owner": "<owner>",

    "tablename": "demo_tab",

    "colname": "data",

}

# 加载文档

loader = OracleDocLoader(conn=conn, params=loader_params)

docs = loader.load()

# 验证

print(f"加载的文档数量: {len(docs)}")

# print(f"文档-0: {docs[0].page_content}") # 内容

```
### 分割文档
文档的大小可能不同，从小到非常大不等。用户通常希望将文档分成较小的部分，以便生成嵌入。这个分割过程提供了广泛的定制选项。有关这些参数的详细信息，请参阅[Oracle AI 矢量搜索指南](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_vector_chain1.html#GUID-4E145629-7098-4C7C-804F-FC85D1F24240)。
以下是一个演示如何实现这一过程的示例代码
```python
from langchain_community.document_loaders.oracleai import OracleTextSplitter
from langchain_core.documents import Document
# 一些示例
# 按字符分割，最大 500 个字符
splitter_params = {"split": "chars", "max": 500, "normalize": "all"}
# 按单词分割，最大 100 个单词
splitter_params = {"split": "words", "max": 100, "normalize": "all"}
# 按句子分割，最大 20 个句子
splitter_params = {"split": "sentence", "max": 20, "normalize": "all"}
# 按默认参数分割
splitter_params = {"normalize": "all"}
# 获取分割器实例
splitter = OracleTextSplitter(conn=conn, params=splitter_params)
list_chunks = []
for doc in docs:
    chunks = splitter.split_text(doc.page_content)
    list_chunks.extend(chunks)
# 验证
print(f"分割后的块数: {len(list_chunks)}")
# print(f"块-0: {list_chunks[0]}") # 内容
```

### 全流程演示

请参阅我们的完整演示指南[Oracle AI 矢量搜索全流程演示指南](https://github.com/langchain-ai/langchain/tree/master/cookbook/oracleai_demo.ipynb)，以借助 Oracle AI 矢量搜索构建端到端 RAG 流水线。