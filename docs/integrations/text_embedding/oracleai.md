# Oracle AI 矢量搜索：生成嵌入

Oracle AI 矢量搜索专为人工智能（AI）工作负载而设计，允许您基于语义而不是关键词查询数据。

Oracle AI 矢量搜索的最大优势之一是，可以将非结构化数据的语义搜索与业务数据的关系搜索结合在一个单一系统中。

这不仅强大，而且更加有效，因为您无需添加专门的矢量数据库，消除了在多个系统之间的数据碎片化带来的痛苦。

此外，您的矢量可以受益于 Oracle Database 的所有最强大的功能，例如以下内容：

- [分区支持](https://www.oracle.com/database/technologies/partitioning.html)

- [实时应用集群可伸缩性](https://www.oracle.com/database/real-application-clusters/)

- [Exadata 智能扫描](https://www.oracle.com/database/technologies/exadata/software/smartscan/)

- [在地理分布式数据库之间的分片处理](https://www.oracle.com/database/distributed-database/)

- [事务](https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/transactions.html)

- [并行 SQL](https://docs.oracle.com/en/database/oracle/oracle-database/21/vldbg/parallel-exec-intro.html#GUID-D28717E4-0F77-44F5-BB4E-234C31D4E4BA)

- [灾难恢复](https://www.oracle.com/database/data-guard/)

- [安全性](https://www.oracle.com/security/database-security/)

- [Oracle 机器学习](https://www.oracle.com/artificial-intelligence/database-machine-learning/)

- [Oracle 图数据库](https://www.oracle.com/database/integrated-graph-database/)

- [Oracle 空间和图形](https://www.oracle.com/database/spatial/)

- [Oracle 区块链](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_blockchain_table.html#GUID-B469E277-978E-4378-A8C1-26D3FF96C9A6)

- [JSON](https://docs.oracle.com/en/database/oracle/oracle-database/23/adjsn/json-in-oracle-database.html)

该指南演示了如何使用 Oracle AI 矢量搜索中的嵌入功能，使用 OracleEmbeddings 为您的文档生成嵌入。

如果您刚开始使用 Oracle Database，请考虑探索[免费的 Oracle 23 AI](https://www.oracle.com/database/free/#resources)，它提供了设置数据库环境的绝佳介绍。在使用数据库时，通常建议避免默认使用系统用户；相反，您可以为增强安全性和定制性创建自己的用户。有关用户创建的详细步骤，请参阅我们的[端到端指南](https://github.com/langchain-ai/langchain/blob/master/cookbook/oracleai_demo.ipynb)，该指南还展示了如何在 Oracle 中设置用户。此外，了解用户权限对于有效管理数据库安全至关重要。您可以在官方的[Oracle 指南](https://docs.oracle.com/en/database/oracle/oracle-database/19/admqs/administering-user-accounts-and-security.html#GUID-36B21D72-1BBB-46C9-A0C9-F0D2A8591B8D)中了解更多关于这个主题的信息。

### 先决条件

确保已安装 Oracle Python 客户端驱动程序，以便与 Oracle AI 矢量搜索集成。

```python
# pip install oracledb
```

### 连接到 Oracle 数据库

以下示例代码将展示如何连接到 Oracle 数据库。默认情况下，python-oracledb 以“Thin”模式运行，直接连接到 Oracle 数据库。此模式不需要 Oracle 客户端库。但是，当 python-oracledb 使用它们时，会提供一些额外的功能。当 python-oracledb 使用 Oracle 客户端库时，它被称为“Thick”模式。这两种模式都具有全面的功能，支持 Python 数据库 API v2.0 规范。请参阅以下[指南](https://python-oracledb.readthedocs.io/en/latest/user_guide/appendix_a.html#featuresummary)，了解每种模式支持的功能。如果无法使用“Thin”模式，您可能希望切换到“Thick”模式。

```python
import sys
import oracledb
# 使用您的 Oracle 数据库凭据和连接详细信息更新以下变量
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

对于嵌入生成，用户可以选择多种提供商选项，包括在数据库内进行嵌入生成以及第三方服务，如 OcigenAI、Hugging Face 和 OpenAI。选择第三方提供商的用户必须建立包括必要认证信息在内的凭据。另外，如果用户选择“数据库”作为提供商，他们需要将一个 ONNX 模型加载到 Oracle 数据库中以便生成嵌入。

### 加载 ONNX 模型

Oracle 支持多种嵌入提供商，使用户可以在专有数据库解决方案和第三方服务（如 OCIGENAI 和 HuggingFace）之间进行选择。此选择决定了生成和管理嵌入的方法。

***重要***：如果用户选择数据库选项，他们必须将一个 ONNX 模型上传到 Oracle 数据库。相反，如果选择第三方提供商进行嵌入生成，则不需要将 ONNX 模型上传到 Oracle 数据库。

直接在 Oracle 中使用 ONNX 模型的一个重要优势是通过消除向外部传输数据的需求，提供了增强的安全性和性能。此外，这种方法避免了通常与网络或 REST API 调用相关的延迟。

以下是将 ONNX 模型上传到 Oracle 数据库的示例代码：

```python
from langchain_community.embeddings.oracleai import OracleEmbeddings
# 更新您的 ONNX 模型的目录和文件名
# 确保系统中有 ONNX 文件
onnx_dir = "DEMO_DIR"
onnx_file = "tinybert.onnx"
model_name = "demo_model"
try:
    OracleEmbeddings.load_onnx_model(conn, onnx_dir, onnx_file, model_name)
    print("ONNX 模型已加载。")
except Exception as e:
    print("ONNX 模型加载失败！")
    sys.exit(1)
```

### 创建凭据

当选择第三方提供商生成嵌入时，用户需要建立凭据以安全访问提供商的端点。

***重要***：选择“数据库”提供商生成嵌入时不需要凭据。但是，如果用户决定使用第三方提供商，则必须创建特定于所选提供商的凭据。

以下是一个说明性示例：

```python
try:
    cursor = conn.cursor()
    cursor.execute(
        """
       declare
           jo json_object_t;
       begin
           -- HuggingFace
           dbms_vector_chain.drop_credential(credential_name  => 'HF_CRED');
           jo := json_object_t();
           jo.put('access_token', '<access_token>');
           dbms_vector_chain.create_credential(
               credential_name   =>  'HF_CRED',
               params            => json(jo.to_string));
           -- OCIGENAI
           dbms_vector_chain.drop_credential(credential_name  => 'OCI_CRED');
           jo := json_object_t();
           jo.put('user_ocid','<user_ocid>');
           jo.put('tenancy_ocid','<tenancy_ocid>');
           jo.put('compartment_ocid','<compartment_ocid>');
           jo.put('private_key','<private_key>');
           jo.put('fingerprint','<fingerprint>');
           dbms_vector_chain.create_credential(
               credential_name   => 'OCI_CRED',
               params            => json(jo.to_string));
       end;
       """
    )
    cursor.close()
    print("凭据已创建。")
except Exception as ex:
    cursor.close()
    raise
```

### 生成嵌入

Oracle AI Vector Search 提供了多种生成嵌入的方法，可以利用本地托管的 ONNX 模型或第三方 API。有关配置这些替代方案的详细说明，请参阅 [Oracle AI Vector Search 指南](https://docs.oracle.com/en/database/oracle/oracle-database/23/arpls/dbms_vector_chain1.html#GUID-C6439E94-4E86-4ECD-954E-4B73D53579DE)。

***注意***：目前，OracleEmbeddings 通过单独调用 REST 端点处理每个嵌入生成请求，而不是批处理。这种方法可能导致超出某些提供商设置的每分钟最大请求配额。但是，我们正在积极努力改进这一流程，通过实现请求批处理来优化我们对提供商资源的使用，并遵守其请求限制。预计很快将推出此更新，消除当前的限制。

***注意***：用户可能需要配置代理以利用第三方嵌入生成提供商，但“数据库”提供商使用了一个 ONNX 模型，因此不需要配置代理。

以下是一个示例代码，演示如何生成嵌入：

```python
from langchain_community.embeddings.oracleai import OracleEmbeddings
from langchain_core.documents import Document
# 使用 OCIGENAI
embedder_params = {
    "provider": "ocigenai",
    "credential_name": "OCI_CRED",
    "url": "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/embedText",
    "model": "cohere.embed-english-light-v3.0",
}
# 使用 HuggingFace
embedder_params = {
    "provider": "huggingface", 
    "credential_name": "HF_CRED", 
    "url": "https://api-inference.huggingface.co/pipeline/feature-extraction/", 
    "model": "sentence-transformers/all-MiniLM-L6-v2", 
    "wait_for_model": "true"
}
# 使用加载到 Oracle 数据库的 ONNX 模型
embedder_params = {"provider": "database", "model": "demo_model"}
# 如果您的环境不需要代理，可以省略下面的 'proxy' 参数
embedder = OracleEmbeddings(conn=conn, params=embedder_params, proxy=proxy)
embed = embedder.embed_query("Hello World!")
# 验证
print(f"OracleEmbeddings 生成的嵌入：{embed}")
```

### 一揽子演示

请参考我们的完整演示指南 [Oracle AI Vector Search 一揽子演示指南](https://github.com/langchain-ai/langchain/tree/master/cookbook/oracleai_demo.ipynb)，以借助 Oracle AI Vector Search 构建一揽子 RAG 管道。