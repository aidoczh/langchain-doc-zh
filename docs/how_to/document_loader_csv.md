# 如何加载 CSV 文件

[逗号分隔值（CSV）](https://zh.wikipedia.org/wiki/逗号分隔值)文件是一种使用逗号分隔值的定界文本文件。文件的每一行是一个数据记录。每个记录由一个或多个字段组成，字段之间用逗号分隔。

LangChain 实现了一个 [CSV 加载器](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.csv_loader.CSVLoader.html)，可以将 CSV 文件加载为一系列 [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html#langchain_core.documents.base.Document) 对象。CSV 文件的每一行都会被翻译为一个文档。

```python
from langchain_community.document_loaders.csv_loader import CSVLoader
file_path = (
    "../../../docs/integrations/document_loaders/example_data/mlb_teams_2012.csv"
)
loader = CSVLoader(file_path=file_path)
data = loader.load()
for record in data[:2]:
    print(record)
```

```output
page_content='Team: Nationals\n"Payroll (millions)": 81.34\n"Wins": 98' metadata={'source': '../../../docs/integrations/document_loaders/example_data/mlb_teams_2012.csv', 'row': 0}
page_content='Team: Reds\n"Payroll (millions)": 82.20\n"Wins": 97' metadata={'source': '../../../docs/integrations/document_loaders/example_data/mlb_teams_2012.csv', 'row': 1}
```

## 自定义 CSV 解析和加载

`CSVLoader` 接受一个 `csv_args` 关键字参数，用于自定义传递给 Python 的 `csv.DictReader` 的参数。有关支持的 csv 参数的更多信息，请参阅 [csv 模块](https://docs.python.org/zh-cn/3/library/csv.html) 文档。

```python
loader = CSVLoader(
    file_path=file_path,
    csv_args={
        "delimiter": ",",
        "quotechar": '"',
        "fieldnames": ["MLB Team", "Payroll in millions", "Wins"],
    },
)
data = loader.load()
for record in data[:2]:
    print(record)
```

```output
page_content='MLB Team: Team\nPayroll in millions: "Payroll (millions)"\nWins: "Wins"' metadata={'source': '../../../docs/integrations/document_loaders/example_data/mlb_teams_2012.csv', 'row': 0}
page_content='MLB Team: Nationals\nPayroll in millions: 81.34\nWins: 98' metadata={'source': '../../../docs/integrations/document_loaders/example_data/mlb_teams_2012.csv', 'row': 1}
```

## 指定用于标识文档来源的列

[Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html#langchain_core.documents.base.Document) 元数据中的 `"source"` 键可以使用 CSV 的某一列进行设置。使用 `source_column` 参数指定从每一行创建的文档的来源。否则，`file_path` 将用作从 CSV 文件创建的所有文档的来源。

当使用从 CSV 文件加载的文档回答问题的链时，这非常有用。

```python
loader = CSVLoader(file_path=file_path, source_column="Team")
data = loader.load()
for record in data[:2]:
    print(record)
```

```output
page_content='Team: Nationals\n"Payroll (millions)": 81.34\n"Wins": 98' metadata={'source': 'Nationals', 'row': 0}
page_content='Team: Reds\n"Payroll (millions)": 82.20\n"Wins": 97' metadata={'source': 'Reds', 'row': 1}
```

## 从字符串加载

当直接使用 CSV 字符串时，可以使用 Python 的 `tempfile`。

```python
import tempfile
from io import StringIO
string_data = """
"Team", "Payroll (millions)", "Wins"
"Nationals",     81.34, 98
"Reds",          82.20, 97
"Yankees",      197.96, 95
"Giants",       117.62, 94
""".strip()
with tempfile.NamedTemporaryFile(delete=False, mode="w+") as temp_file:
    temp_file.write(string_data)
    temp_file_path = temp_file.name
loader = CSVLoader(file_path=temp_file_path)
loader.load()
for record in data[:2]:
    print(record)
```

```output
page_content='Team: Nationals\n"Payroll (millions)": 81.34\n"Wins": 98' metadata={'source': 'Nationals', 'row': 0}
page_content='Team: Reds\n"Payroll (millions)": 82.20\n"Wins": 97' metadata={'source': 'Reds', 'row': 1}
```