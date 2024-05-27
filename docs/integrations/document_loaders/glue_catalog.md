# Glue 目录

[**AWS Glue 数据目录**](https://docs.aws.amazon.com/en_en/glue/latest/dg/catalog-and-crawler.html) 是一个集中的元数据存储库，允许您管理、访问和共享存储在 AWS 中的数据的元数据。它充当您的数据资产的元数据存储，使各种 AWS 服务和您的应用程序能够高效地查询和连接到它们所需的数据。

当您在 AWS Glue 中定义数据源、转换和目标时，有关这些元素的元数据存储在数据目录中。这包括有关数据位置、模式定义、运行时指标等的信息。它支持各种数据存储类型，如 Amazon S3、Amazon RDS、Amazon Redshift 和与 JDBC 兼容的外部数据库。它还直接集成了 Amazon Athena、Amazon Redshift Spectrum 和 Amazon EMR，使这些服务能够直接访问和查询数据。

Langchain GlueCatalogLoader 将以与 Pandas dtype 相同的格式获取给定 Glue 数据库中所有表的模式。

## 设置

- 按照[设置 AWS 账户的说明](https://docs.aws.amazon.com/athena/latest/ug/setting-up.html)。

- 安装 boto3 库：`pip install boto3`

## 示例

```python
from langchain_community.document_loaders.glue_catalog import GlueCatalogLoader
```

```python
database_name = "my_database"
profile_name = "my_profile"
loader = GlueCatalogLoader(
    database=database_name,
    profile_name=profile_name,
)
schemas = loader.load()
print(schemas)
```

## 带表过滤的示例

表过滤允许您有选择性地检索 Glue 数据库中特定子集表的模式信息。您可以使用 `table_filter` 参数来指定您感兴趣的确切表。

```python
from langchain_community.document_loaders.glue_catalog import GlueCatalogLoader
```

```python
database_name = "my_database"
profile_name = "my_profile"
table_filter = ["table1", "table2", "table3"]
loader = GlueCatalogLoader(
    database=database_name, profile_name=profile_name, table_filter=table_filter
)
schemas = loader.load()
print(schemas)
```