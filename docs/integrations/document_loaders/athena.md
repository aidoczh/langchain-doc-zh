# Athena

[Amazon Athena](https://aws.amazon.com/athena/) 是一个基于开源框架构建的无服务器交互式分析服务，支持开放表格和文件格式。`Athena` 提供了一种简化、灵活的方式来分析存储在数据湖中的千兆字节数据。您可以使用 SQL 或 Python 分析数据或构建应用程序，这些数据可以来自 Amazon Simple Storage Service (S3) 数据湖和 30 个数据源，包括本地数据源或其他云系统。`Athena` 基于开源的 `Trino` 和 `Presto` 引擎以及 `Apache Spark` 框架构建，无需进行任何配置或配置工作。

本笔记介绍了如何从 `AWS Athena` 加载文档。

## 设置

请按照[设置 AWS 账户的说明](https://docs.aws.amazon.com/athena/latest/ug/setting-up.html)进行操作。

安装 Python 库：

```python
! pip install boto3
```

## 示例

```python
from langchain_community.document_loaders.athena import AthenaLoader
```

```python
database_name = "my_database"
s3_output_path = "s3://my_bucket/query_results/"
query = "SELECT * FROM my_table"
profile_name = "my_profile"
loader = AthenaLoader(
    query=query,
    database=database_name,
    s3_output_uri=s3_output_path,
    profile_name=profile_name,
)
documents = loader.load()
print(documents)
```

带有元数据列的示例

```python
database_name = "my_database"
s3_output_path = "s3://my_bucket/query_results/"
query = "SELECT * FROM my_table"
profile_name = "my_profile"
metadata_columns = ["_row", "_created_at"]
loader = AthenaLoader(
    query=query,
    database=database_name,
    s3_output_uri=s3_output_path,
    profile_name=profile_name,
    metadata_columns=metadata_columns,
)
documents = loader.load()
print(documents)
```