# SQL数据库

::: {.callout-note}

`SQLDatabase` 适配器实用程序是数据库连接的包装器。

用于与SQL数据库通信时，它使用 [SQLAlchemy] 核心 API。

:::

本笔记本展示了如何使用该实用程序访问SQLite数据库。

它使用了 [Chinook Database] 示例，并演示了以下功能：

- 使用SQL查询

- 使用SQLAlchemy可选择的查询

- 获取模式 `cursor`，`all` 和 `one`

- 绑定查询参数

[Chinook Database]: https://github.com/lerocha/chinook-database

[SQLAlchemy]: https://www.sqlalchemy.org/

您可以使用 `Tool` 或 `@tool` 装饰器从此实用程序创建工具。

::: {.callout-caution}

如果从 SQLDatbase 实用程序创建工具并将其与 LLM 结合使用或向最终用户公开，请记住遵循良好的安全实践。

查看安全信息：https://python.langchain.com/docs/security

:::

```python
!wget 'https://github.com/lerocha/chinook-database/releases/download/v1.4.2/Chinook_Sqlite.sql'
```

```python
!sqlite3 -bail -cmd '.read Chinook_Sqlite.sql' -cmd 'SELECT * FROM Artist LIMIT 12;' -cmd '.quit'
```

```output
1|AC/DC
2|Accept
3|Aerosmith
4|Alanis Morissette
5|Alice In Chains
6|Antônio Carlos Jobim
7|Apocalyptica
8|Audioslave
9|BackBeat
10|Billy Cobham
11|Black Label Society
12|Black Sabbath
```

```python
!sqlite3 -bail -cmd '.read Chinook_Sqlite.sql' -cmd '.save Chinook.db' -cmd '.quit'
```

## 初始化数据库

```python
from pprint import pprint
import sqlalchemy as sa
from langchain_community.utilities import SQLDatabase
db = SQLDatabase.from_uri("sqlite:///Chinook.db")
```

## 作为游标查询

获取模式 `cursor` 将结果作为 SQLAlchemy 的 `CursorResult` 实例返回。

```python
result = db.run("SELECT * FROM Artist LIMIT 12;", fetch="cursor")
print(type(result))
pprint(list(result.mappings()))
```

## 作为字符串载荷查询

获取模式 `all` 和 `one` 以字符串格式返回结果。

```python
result = db.run("SELECT * FROM Artist LIMIT 12;", fetch="all")
print(type(result))
print(result)
```

```python
result = db.run("SELECT * FROM Artist LIMIT 12;", fetch="one")
print(type(result))
print(result)
```

## 带参数查询

为了绑定查询参数，使用可选的 `parameters` 参数。

```python
result = db.run(
    "SELECT * FROM Artist WHERE Name LIKE :search;",
    parameters={"search": "p%"},
    fetch="cursor",
)
pprint(list(result.mappings()))
```

## 使用SQLAlchemy可选择的查询

除了纯文本SQL语句外，该适配器还接受SQLAlchemy可选择的查询。

```python
# 为了在SA的核心API上构建可选择的查询，您需要一个表定义。
metadata = sa.MetaData()
artist = sa.Table(
    "Artist",
    metadata,
    sa.Column("ArtistId", sa.INTEGER, primary_key=True),
    sa.Column("Name", sa.TEXT),
)
# 使用与最近查询相同的语义构建可选择的查询。
query = sa.select(artist).where(artist.c.Name.like("p%"))
result = db.run(query, fetch="cursor")
pprint(list(result.mappings()))
```

## 使用执行选项进行查询

可以通过自定义执行选项来增强语句的调用。

例如，当应用模式名称翻译时，后续的语句将会失败，因为它们试图访问一个不存在的表。

```python
query = sa.select(artist).where(artist.c.Name.like("p%"))
db.run(query, fetch="cursor", execution_options={"schema_translate_map": {None: "bar"}})
```

![图片描述](https://example.com/image.png)