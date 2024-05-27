# 为 Oracle 工作负载准备 Google El Carro

> Google 的 [El Carro Oracle Operator](https://github.com/GoogleCloudPlatform/elcarro-oracle-operator) 提供了一种在 Kubernetes 中运行 Oracle 数据库的方式，作为一个便携、开源、社区驱动、无供应商锁定的容器编排系统。El Carro 提供了一个强大的声明式 API，用于全面和一致的配置和部署，以及实时操作和监控。

通过利用 El Carro Langchain 集成，扩展你的 Oracle 数据库的功能，构建基于人工智能的体验。

本指南介绍了如何使用 El Carro Langchain 集成来使用 `ElCarroLoader` 和 `ElCarroDocumentSaver` [保存、加载和删除 langchain 文档](/docs/how_to#document-loaders)。该集成适用于任何 Oracle 数据库，无论其运行在何处。

在 [GitHub](https://github.com/googleapis/langchain-google-el-carro-python/) 上了解更多关于该包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-el-carro-python/blob/main/docs/document_loader.ipynb)

## 开始之前

请完成[入门指南](https://github.com/googleapis/langchain-google-el-carro-python/tree/main/README.md#getting-started)部分的内容，以设置你的 El Carro Oracle 数据库。

### 🦜🔗 库安装

该集成位于自己的 `langchain-google-el-carro` 包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-el-carro
```

## 基本用法

### 设置 Oracle 数据库连接

填写以下变量，使用你的 Oracle 数据库连接详细信息。

```python
# @title 在此处设置你的值 { display-mode: "form" }
HOST = "127.0.0.1"  # @param {type: "string"}
PORT = 3307  # @param {type: "integer"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "message_store"  # @param {type: "string"}
USER = "my-user"  # @param {type: "string"}
PASSWORD = input("请提供用于数据库用户的密码：")
```

如果你正在使用 El Carro，你可以在 El Carro Kubernetes 实例的状态中找到主机名和端口值。使用你为 PDB 创建的用户密码。

示例输出：

```
kubectl get -w instances.oracle.db.anthosapis.com -n db
NAME   DB ENGINE   VERSION   EDITION      ENDPOINT      URL                DB NAMES   BACKUP ID   READYSTATUS   READYREASON        DBREADYSTATUS   DBREADYREASON
mydb   Oracle      18c       Express      mydb-svc.db   34.71.69.25:6021   ['pdbname']            TRUE          CreateComplete     True            CreateComplete
```

### ElCarroEngine 连接池

`ElCarroEngine` 配置了一个连接池到你的 Oracle 数据库，使应用程序能够成功连接，并遵循行业最佳实践。

```python
from langchain_google_el_carro import ElCarroEngine
elcarro_engine = ElCarroEngine.from_instance(
    db_host=HOST,
    db_port=PORT,
    db_name=DATABASE,
    db_user=USER,
    db_password=PASSWORD,
)
```

### 初始化表

通过 `elcarro_engine.init_document_table(<table_name>)` 初始化一个默认模式的表。表列：

- page_content（类型：文本）

- langchain_metadata（类型：JSON）

```python
elcarro_engine.drop_document_table(TABLE_NAME)
elcarro_engine.init_document_table(
    table_name=TABLE_NAME,
)
```

### 保存文档

使用 `ElCarroDocumentSaver.add_documents(<documents>)` 保存 langchain 文档。要初始化 `ElCarroDocumentSaver` 类，你需要提供两件事情：

1. `elcarro_engine` - `ElCarroEngine` 引擎的实例。

2. `table_name` - 存储 langchain 文档的 Oracle 数据库内表的名称。

```python
from langchain_core.documents import Document
from langchain_google_el_carro import ElCarroDocumentSaver
doc = Document(
    page_content="Banana",
    metadata={"type": "fruit", "weight": 100, "organic": 1},
)
saver = ElCarroDocumentSaver(
    elcarro_engine=elcarro_engine,
    table_name=TABLE_NAME,
)
saver.add_documents([doc])
```

### 加载文档

使用 `ElCarroLoader.load()` 或 `ElCarroLoader.lazy_load()` 加载 langchain 文档。`lazy_load` 返回一个仅在迭代期间查询数据库的生成器。要初始化 `ElCarroLoader` 类，你需要提供：

1. `elcarro_engine` - `ElCarroEngine` 引擎的实例。

2. `table_name` - 存储 langchain 文档的 Oracle 数据库内表的名称。

```python
from langchain_google_el_carro import ElCarroLoader
loader = ElCarroLoader(elcarro_engine=elcarro_engine, table_name=TABLE_NAME)
docs = loader.lazy_load()
for doc in docs:
    print("加载的文档：", doc)
```

### 通过查询加载文档

除了从表中加载文档外，我们还可以选择从通过 SQL 查询生成的视图中加载文档。例如：

```python
from langchain_google_el_carro import ElCarroLoader
loader = ElCarroLoader(
    elcarro_engine=elcarro_engine,
    query=f"SELECT * FROM {TABLE_NAME} WHERE json_value(langchain_metadata, '$.organic') = '1'",
)
onedoc = loader.load()
print(onedoc)
```

通过 SQL 查询生成的视图可能具有不同的模式，与默认表不同。

在这种情况下，ElCarroLoader 的行为与从具有非默认模式的表中加载相同。请参考

[加载具有自定义文档页面内容和元数据的文档](#load-documents-with-customized-document-page-content--metadata) 部分。

### 删除文档

从 Oracle 表中删除一系列 langchain 文档

使用 `ElCarroDocumentSaver.delete(<documents>)`。

对于具有默认模式（page_content, langchain_metadata）的表，

删除条件为：

如果存在列表中的 `document`，则应删除 `row`，使得

- `document.page_content` 等于 `row[page_content]`

- `document.metadata` 等于 `row[langchain_metadata]`

```python
docs = loader.load()
print("删除前的文档:", docs)
saver.delete(onedoc)
print("删除后的文档:", loader.load())
```

## 高级用法

### 加载具有自定义文档页面内容和元数据的文档

首先，我们准备一个具有非默认模式的示例表，并用一些任意数据填充它。

```python
import sqlalchemy
create_table_query = f"""CREATE TABLE {TABLE_NAME} (
    fruit_id NUMBER GENERATED BY DEFAULT AS IDENTITY (START WITH 1),
    fruit_name VARCHAR2(100) NOT NULL,
    variety VARCHAR2(50),
    quantity_in_stock NUMBER(10) NOT NULL,
    price_per_unit NUMBER(6,2) NOT NULL,
    organic NUMBER(3) NOT NULL
)"""
elcarro_engine.drop_document_table(TABLE_NAME)
with elcarro_engine.connect() as conn:
    conn.execute(sqlalchemy.text(create_table_query))
    conn.commit()
    conn.execute(
        sqlalchemy.text(
            f"""
            INSERT INTO {TABLE_NAME} (fruit_name, variety, quantity_in_stock, price_per_unit, organic)
            VALUES ('Apple', 'Granny Smith', 150, 0.99, 1)
            """
        )
    )
    conn.execute(
        sqlalchemy.text(
            f"""
            INSERT INTO {TABLE_NAME} (fruit_name, variety, quantity_in_stock, price_per_unit, organic)
            VALUES ('Banana', 'Cavendish', 200, 0.59, 0)
            """
        )
    )
    conn.execute(
        sqlalchemy.text(
            f"""
            INSERT INTO {TABLE_NAME} (fruit_name, variety, quantity_in_stock, price_per_unit, organic)
            VALUES ('Orange', 'Navel', 80, 1.29, 1)
            """
        )
    )
    conn.commit()
```

如果我们仍然使用 `ElCarroLoader` 的默认参数从这个示例表中加载 langchain 文档，加载的文档的 `page_content` 将是表的第一列，而 `metadata` 将由所有其他列的键值对组成。

```python
loader = ElCarroLoader(
    elcarro_engine=elcarro_engine,
    table_name=TABLE_NAME,
)
loaded_docs = loader.load()
print(f"加载的文档: [{loaded_docs}]")
```

我们可以通过在初始化 `ElCarroLoader` 时设置 `content_columns` 和 `metadata_columns` 来指定要加载的内容和元数据。

1. `content_columns`: 要写入文档的 `page_content` 的列。

2. `metadata_columns`: 要写入文档的 `metadata` 的列。

例如，在这里，`content_columns` 中的列的值将被连接成一个以空格分隔的字符串，作为加载的文档的 `page_content`，而加载的文档的 `metadata` 将只包含在 `metadata_columns` 中指定的列的键值对。

```python
loader = ElCarroLoader(
    elcarro_engine=elcarro_engine,
    table_name=TABLE_NAME,
    content_columns=[
        "variety",
        "quantity_in_stock",
        "price_per_unit",
        "organic",
    ],
    metadata_columns=["fruit_id", "fruit_name"],
)
loaded_docs = loader.load()
print(f"加载的文档: [{loaded_docs}]")
```

### 保存具有自定义页面内容和元数据的文档

为了将 langchain 文档保存到具有自定义元数据字段的表中，我们需要首先通过 `ElCarroEngine.init_document_table()` 创建这样的表，并指定我们希望其具有的 `metadata_columns` 列表。在这个例子中，创建的表将具有以下列：

- content（类型：文本）：用于存储水果描述。

- type（类型：VARCHAR2(200)）：用于存储水果类型。

- weight（类型：INT）：用于存储水果重量。

- extra_json_metadata（类型：JSON）：用于存储水果的其他元数据信息。

我们可以使用以下参数

与 `elcarro_engine.init_document_table()` 一起使用来创建表：

```python
1. `table_name`：在 Oracle 数据库中存储 langchain 文档的表名。
2. `metadata_columns`：`sqlalchemy.Column` 列表，指示我们需要的元数据列的列表。
3. `content_column`：存储 langchain 文档的 `page_content` 的列名。默认值：`"page_content", "VARCHAR2(4000)"`
4. `metadata_json_column`：存储 langchain 文档额外 JSON `metadata` 的列名。默认值：`"langchain_metadata", "VARCHAR2(4000)"`。
elcarro_engine.drop_document_table(TABLE_NAME)
elcarro_engine.init_document_table(
    table_name=TABLE_NAME,
    metadata_columns=[
        sqlalchemy.Column("type", sqlalchemy.dialects.oracle.VARCHAR2(200)),
        sqlalchemy.Column("weight", sqlalchemy.INT),
    ],
    content_column="content",
    metadata_json_column="extra_json_metadata",
)
```

使用 `ElCarroDocumentSaver.add_documents(<documents>)` 保存文档。如下例所示，

- `document.page_content` 将被保存到 `content` 列中。

- `document.metadata.type` 将被保存到 `type` 列中。

- `document.metadata.weight` 将被保存到 `weight` 列中。

- `document.metadata.organic` 将以 JSON 格式保存到 `extra_json_metadata` 列中。

```python
doc = Document(
    page_content="Banana",
    metadata={"type": "fruit", "weight": 100, "organic": 1},
)
print(f"原始文档: [{doc}]")
saver = ElCarroDocumentSaver(
    elcarro_engine=elcarro_engine,
    table_name=TABLE_NAME,
    content_column="content",
    metadata_json_column="extra_json_metadata",
)
saver.add_documents([doc])
loader = ElCarroLoader(
    elcarro_engine=elcarro_engine,
    table_name=TABLE_NAME,
    content_columns=["content"],
    metadata_columns=[
        "type",
        "weight",
    ],
    metadata_json_column="extra_json_metadata",
)
loaded_docs = loader.load()
print(f"加载后的文档: [{loaded_docs[0]}]")
```

### 使用自定义页面内容和元数据删除文档

我们还可以通过 `ElCarroDocumentSaver.delete(<documents>)` 从表中删除具有自定义元数据列的文档。删除条件为：

如果存在列表中的 `document`，则应删除 `row`，使得

- `document.page_content` 等于 `row[page_content]`

- 对于 `document.metadata` 中的每个元数据字段 `k`

    - `document.metadata[k]` 等于 `row[k]` 或 `document.metadata[k]` 等于 `row[langchain_metadata][k]`

- `row` 中不存在额外的元数据字段，而不在 `document.metadata` 中。

```python
loader = ElCarroLoader(elcarro_engine=elcarro_engine, table_name=TABLE_NAME)
saver.delete(loader.load())
print(f"剩余文档数: {len(loader.load())}")
```

## 更多示例

请查看 [demo_doc_loader_basic.py](https://github.com/googleapis/langchain-google-el-carro-python/tree/main/samples/demo_doc_loader_basic.py)

和 [demo_doc_loader_advanced.py](https://github.com/googleapis/langchain-google-el-carro-python/tree/main/samples/demo_doc_loader_advanced.py)

获取完整的代码示例。

```