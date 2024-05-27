# DuckDB

[DuckDB](https://duckdb.org/) 是一种内置的 SQL OLAP 数据库管理系统。

使用以下代码将一个文档加载到 `DuckDB` 查询中，每行一个文档。

```python
%pip install --upgrade --quiet  duckdb
```

```python
from langchain_community.document_loaders import DuckDBLoader
```

```python
%%file example.csv
Team,Payroll
Nationals,81.34
Reds,82.20
```

```output
Writing example.csv
```

```python
loader = DuckDBLoader("SELECT * FROM read_csv_auto('example.csv')")
data = loader.load()
```

```python
print(data)
```

```output
[Document(page_content='Team: Nationals\nPayroll: 81.34', metadata={}), Document(page_content='Team: Reds\nPayroll: 82.2', metadata={})]
```

## 指定哪些列是内容，哪些列是元数据

```python
loader = DuckDBLoader(
    "SELECT * FROM read_csv_auto('example.csv')",
    page_content_columns=["Team"],
    metadata_columns=["Payroll"],
)
data = loader.load()
```

```python
print(data)
```

```output
[Document(page_content='Team: Nationals', metadata={'Payroll': 81.34}), Document(page_content='Team: Reds', metadata={'Payroll': 82.2})]
```

## 将来源添加到元数据中

```python
loader = DuckDBLoader(
    "SELECT Team, Payroll, Team As source FROM read_csv_auto('example.csv')",
    metadata_columns=["source"],
)
data = loader.load()
```

```python
print(data)
```

```output
[Document(page_content='Team: Nationals\nPayroll: 81.34\nsource: Nationals', metadata={'source': 'Nationals'}), Document(page_content='Team: Reds\nPayroll: 82.2\nsource: Reds', metadata={'source': 'Reds'})]
```