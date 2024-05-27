# Oracle AI Vector Search: 生成摘要

Oracle AI Vector Search 旨在处理人工智能（AI）工作负载，使您能够基于语义而非关键词查询数据。

Oracle AI Vector Search 最大的好处之一是，可以在一个系统中将对非结构化数据的语义搜索与对业务数据的关系搜索结合起来。

这不仅强大，而且效果显著，因为您无需添加专门的向量数据库，消除了在多个系统之间数据碎片化的烦恼。

此外，您的向量可以受益于 Oracle Database 的所有最强大功能，如以下内容：

- [分区支持](https://www.oracle.com/database/technologies/partitioning.html)

- [实时应用集群可伸缩性](https://www.oracle.com/database/real-application-clusters/)

- [Exadata 智能扫描](https://www.oracle.com/database/technologies/exadata/software/smartscan/)

- [跨地理分布式数据库的分片处理](https://www.oracle.com/database/distributed-database/)

- [事务](https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/transactions.html)

- [并行 SQL](https://docs.oracle.com/en/database/oracle/oracle-database/21/vldbg/parallel-exec-intro.html#GUID-D28717E4-0F77-44F5-BB4E-234C31D4E4BA)

- [灾难恢复](https://www.oracle.com/database/data-guard/)

- [安全性](https://www.oracle.com/security/database-security/)

- [Oracle 机器学习](https://www.oracle.com/artificial-intelligence/database-machine-learning/)

- [Oracle 图数据库](https://www.oracle.com/database/integrated-graph-database/)

- [Oracle 空间和图形](https://www.oracle.com/database/spatial/)

- [Oracle 区块链](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_blockchain_table.html#GUID-B469E277-978E-4378-A8C1-26D3FF96C9A6)

- [JSON](https://docs.oracle.com/en/database/oracle/oracle-database/23/adjsn/json-in-oracle-database.html)

该指南演示了如何使用 Oracle AI Vector Search 中的摘要功能来使用 OracleSummary 为您的文档生成摘要。

如果您刚开始使用 Oracle Database，请考虑探索[免费的 Oracle 23 AI](https://www.oracle.com/database/free/#resources)，它为设置数据库环境提供了很好的介绍。在使用数据库时，通常建议不要默认使用系统用户；相反，您可以为增强安全性和定制性创建自己的用户。有关用户创建的详细步骤，请参阅我们的[端到端指南](https://github.com/langchain-ai/langchain/blob/master/cookbook/oracleai_demo.ipynb)，该指南还展示了如何在 Oracle 中设置用户。此外，了解用户权限对有效管理数据库安全至关重要。您可以在官方[Oracle指南](https://docs.oracle.com/en/database/oracle/oracle-database/19/admqs/administering-user-accounts-and-security.html#GUID-36B21D72-1BBB-46C9-A0C9-F0D2A8591B8D)中了解更多关于此主题的信息。

### 先决条件

请安装 Oracle Python 客户端驱动程序以使用 Langchain 与 Oracle AI Vector Search。

```python
# pip install oracledb
```

### 连接到 Oracle Database

以下示例代码将展示如何连接到 Oracle Database。默认情况下，python-oracledb 以“Thin”模式运行，直接连接到 Oracle Database。此模式不需要 Oracle 客户端库。但是，当 python-oracledb 使用它们时，会提供一些额外功能。当 Oracle 客户端库被使用时，python-oracledb 被称为“Thick”模式。这两种模式都具有全面的功能，支持 Python 数据库 API v2.0 规范。请参阅以下[指南](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_a.html#featuresummary)，其中介绍了每种模式支持的功能。如果无法使用 Thin 模式，您可能希望切换到 Thick 模式。

```python
import sys
import oracledb
# 请更新您的用户名、密码、主机名和服务名
username = "<用户名>"
password = "<密码>"
dsn = "<主机名>/<服务名>"
try:
    conn = oracledb.connect(user=username, password=password, dsn=dsn)
    print("连接成功！")
except Exception as e:
    print("连接失败！")
    sys.exit(1)
```

### 生成摘要

Oracle AI Vector Search Langchain库提供了一套专为文档摘要设计的API。它支持多个摘要提供者，如Database、OCIGENAI、HuggingFace等，允许用户选择最符合其需求的提供者。为利用这些功能，用户必须按照指定的方式配置摘要参数。有关这些参数的详细信息，请参阅[Oracle AI Vector Search指南](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_vector_chain1.html#GUID-EC9DDB58-6A15-4B36-BA66-ECBA20D2CE57)。

***注：*** 如果用户想要使用除Oracle内部和默认提供者“database”之外的第三方摘要生成提供者，可能需要设置代理。如果您没有代理，请在实例化OracleSummary时删除代理参数。

```python
# 在实例化摘要和嵌入对象时要使用的代理
proxy = "<proxy>"
```

以下示例代码将展示如何生成摘要：

```python
from langchain_community.utilities.oracleai import OracleSummary
from langchain_core.documents import Document
# 使用'ocigenai'提供者
summary_params = {
    "provider": "ocigenai",
    "credential_name": "OCI_CRED",
    "url": "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/summarizeText",
    "model": "cohere.command",
}
# 使用'huggingface'提供者
summary_params = {
    "provider": "huggingface",
    "credential_name": "HF_CRED",
    "url": "https://api-inference.huggingface.co/models/",
    "model": "facebook/bart-large-cnn",
    "wait_for_model": "true"
}
# 使用'database'提供者
summary_params = {
    "provider": "database",
    "glevel": "S",
    "numParagraphs": 1,
    "language": "english",
}
# 获取摘要实例
# 如果不需要代理，请删除代理
summ = OracleSummary(conn=conn, params=summary_params, proxy=proxy)
summary = summ.get_summary(
    "In the heart of the forest, "
    + "a lone fox ventured out at dusk, seeking a lost treasure. "
    + "With each step, memories flooded back, guiding its path. "
    + "As the moon rose high, illuminating the night, the fox unearthed "
    + "not gold, but a forgotten friendship, worth more than any riches."
)
print(f"OracleSummary生成的摘要：{summary}")
```

### 端到端演示

请参阅我们的完整演示指南[Oracle AI Vector Search端到端演示指南](https://github.com/langchain-ai/langchain/tree/master/cookbook/oracleai_demo.ipynb)，以借助Oracle AI Vector Search构建端到端RAG管道。