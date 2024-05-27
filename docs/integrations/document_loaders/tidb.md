# TiDB

> [TiDB Cloud](https://tidbcloud.com/) 是一款全面的数据库即服务（DBaaS）解决方案，提供了专用和无服务器选项。TiDB 无服务器现在正在将内置的向量搜索集成到 MySQL 环境中。通过这一增强功能，您可以在不需要新数据库或额外技术堆栈的情况下，无缝开发使用 TiDB 无服务器的人工智能应用程序。成为第一批体验者，加入私人测试版的等待列表，网址为 https://tidb.cloud/ai。

本笔记介绍如何使用 `TiDBLoader` 从 TiDB 中加载数据到 langchain 中。

## 先决条件

在使用 `TiDBLoader` 之前，我们将安装以下依赖项：

```python
%pip install --upgrade --quiet langchain
```

然后，我们将配置连接到 TiDB。在本笔记中，我们将按照 TiDB Cloud 提供的标准连接方法来建立安全高效的数据库连接。

```python
import getpass
# 从 TiDB Cloud 控制台复制，用您自己的替换
tidb_connection_string_template = "mysql+pymysql://<USER>:<PASSWORD>@<HOST>:4000/<DB>?ssl_ca=/etc/ssl/cert.pem&ssl_verify_cert=true&ssl_verify_identity=true"
tidb_password = getpass.getpass("输入您的 TiDB 密码:")
tidb_connection_string = tidb_connection_string_template.replace(
    "<PASSWORD>", tidb_password
)
```

## 从 TiDB 加载数据

以下是一些关键参数的详细说明，您可以使用这些参数来自定义 `TiDBLoader` 的行为：

- `query` (str)：这是针对 TiDB 数据库执行的 SQL 查询。查询应选择您想要加载到 `Document` 对象中的数据。例如，您可以使用类似 `"SELECT * FROM my_table"` 的查询来从 `my_table` 中获取所有数据。

- `page_content_columns` (Optional[List[str]])：指定应包含在每个 `Document` 对象的 `page_content` 中的列名列表。如果设置为 `None`（默认值），则查询返回的所有列都包含在 `page_content` 中。这使您可以根据数据的特定列定制每个文档的内容。

- `metadata_columns` (Optional[List[str]])：指定应包含在每个 `Document` 对象的 `metadata` 中的列名列表。默认情况下，此列表为空，这意味着除非显式指定，否则不会包含任何元数据。这对于包含有关每个文档的附加信息（不属于主要内容但仍对处理或分析很有价值）非常有用。

```python
from sqlalchemy import Column, Integer, MetaData, String, Table, create_engine
# 连接数据库
engine = create_engine(tidb_connection_string)
metadata = MetaData()
table_name = "test_tidb_loader"
# 创建表
test_table = Table(
    table_name,
    metadata,
    Column("id", Integer, primary_key=True),
    Column("name", String(255)),
    Column("description", String(255)),
)
metadata.create_all(engine)
with engine.connect() as connection:
    transaction = connection.begin()
    try:
        connection.execute(
            test_table.insert(),
            [
                {"name": "物品 1", "description": "物品 1 的描述"},
                {"name": "物品 2", "description": "物品 2 的描述"},
                {"name": "物品 3", "description": "物品 3 的描述"},
            ],
        )
        transaction.commit()
    except:
        transaction.rollback()
        raise
```

```python
from langchain_community.document_loaders import TiDBLoader
# 设置 TiDBLoader 来检索数据
loader = TiDBLoader(
    connection_string=tidb_connection_string,
    query=f"SELECT * FROM {table_name};",
    page_content_columns=["name", "description"],
    metadata_columns=["id"],
)
# 加载数据
documents = loader.load()
# 显示加载的文档
for doc in documents:
    print("-" * 30)
    print(f"内容: {doc.page_content}\n元数据: {doc.metadata}")
```

```output
------------------------------
内容: name: 物品 1
description: 物品 1 的描述
元数据: {'id': 1}
------------------------------
内容: name: 物品 2
description: 物品 2 的描述
元数据: {'id': 2}
------------------------------
内容: name: 物品 3
description: 物品 3 的描述
元数据: {'id': 3}
```

```python
test_table.drop(bind=engine)
```