# 如何在进行 SQL 问答时处理大型数据库

为了对数据库编写有效的查询，我们需要向模型提供表名、表结构和要查询的特征值。当存在许多表、列和/或高基数列时，我们无法在每个提示中转储关于数据库的完整信息。相反，我们必须找到方法，只动态插入最相关的信息到提示中。

在本指南中，我们将演示识别此类相关信息的方法，并将其馈送到查询生成步骤中。我们将涵盖以下内容：

1. 识别相关的表子集；

2. 识别相关的列值子集。

## 设置

首先，获取所需的软件包并设置环境变量：

```python
%pip install --upgrade --quiet langchain langchain-community langchain-openai
```
```python
# Uncomment the below to use LangSmith. Not required.
# import os
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

下面的示例将使用一个带有 Chinook 数据库的 SQLite 连接。按照[这些安装步骤](https://database.guide/2-sample-databases-sqlite/)在与此笔记本相同的目录中创建 `Chinook.db`：

- 将[此文件](https://raw.githubusercontent.com/lerocha/chinook-database/master/ChinookDatabase/DataSources/Chinook_Sqlite.sql)保存为 `Chinook_Sqlite.sql`

- 运行 `sqlite3 Chinook.db`

- 运行 `.read Chinook_Sqlite.sql`

- 测试 `SELECT * FROM Artist LIMIT 10;`

现在，`Chinhook.db` 在我们的目录中，我们可以使用基于 SQLAlchemy 的 [SQLDatabase](https://api.python.langchain.com/en/latest/utilities/langchain_community.utilities.sql_database.SQLDatabase.html) 类与其进行交互：

```python
from langchain_community.utilities import SQLDatabase
db = SQLDatabase.from_uri("sqlite:///Chinook.db")
print(db.dialect)
print(db.get_usable_table_names())
print(db.run("SELECT * FROM Artist LIMIT 10;"))
```
```output
sqlite
['Album', 'Artist', 'Customer', 'Employee', 'Genre', 'Invoice', 'InvoiceLine', 'MediaType', 'Playlist', 'PlaylistTrack', 'Track']
[(1, 'AC/DC'), (2, 'Accept'), (3, 'Aerosmith'), (4, 'Alanis Morissette'), (5, 'Alice In Chains'), (6, 'Antônio Carlos Jobim'), (7, 'Apocalyptica'), (8, 'Audioslave'), (9, 'BackBeat'), (10, 'Billy Cobham')]
```

## 多个表

我们在提示中需要包含的主要信息之一是相关表的结构。当我们有很多表时，无法将所有的结构都放在一个提示中。在这种情况下，我们可以先提取与用户输入相关的表的名称，然后只包含它们的结构。

一种简单可靠的方法是使用 [tool-calling](/docs/how_to/tool_calling)。下面，我们展示如何使用此功能来获取符合所需格式的输出（在本例中为表名列表）。我们使用聊天模型的 `.bind_tools` 方法来绑定一个以 Pydantic 格式的工具，并将其馈送到输出解析器中，以从模型的响应中重构对象。

```python
from langchain_core.output_parsers.openai_tools import PydanticToolsParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field
class Table(BaseModel):
    """SQL 数据库中的表。"""
    name: str = Field(description="SQL 数据库中的表名。")
table_names = "\n".join(db.get_usable_table_names())
system = f"""返回与用户问题可能相关的所有 SQL 表的名称。
这些表是：
{table_names}
请记住包括所有可能相关的表，即使您不确定是否需要它们。"""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{input}"),
    ]
)
llm_with_tools = llm.bind_tools([Table])
output_parser = PydanticToolsParser(tools=[Table])
table_chain = prompt | llm_with_tools | output_parser
table_chain.invoke({"input": "Alanis Morisette 的所有流派是什么"})
```
```output
[Table(name='Genre')]
```

这个方法效果不错！除了，正如我们将在下面看到的，我们实际上还需要一些其他的表。仅凭用户的问题，模型很难知道这一点。在这种情况下，我们可以通过将表分组来简化模型的工作。我们只需要求模型在“音乐”和“商业”之间进行选择，然后负责从中选择所有相关的表：

```python
system = """返回与用户问题相关的任何 SQL 表的名称。
这些表是：
音乐
商业
"""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{input}"),
    ]
)
category_chain = prompt | llm_with_tools | output_parser
category_chain.invoke({"input": "Alanis Morisette 的所有流派是什么"})
```
```

```
```python
from typing import List
def get_tables(categories: List[Table]) -> List[str]:
    tables = []
    for category in categories:
        if category.name == "Music":
            tables.extend(
                [
                    "Album",
                    "Artist",
                    "Genre",
                    "MediaType",
                    "Playlist",
                    "PlaylistTrack",
                    "Track",
                ]
            )
        elif category.name == "Business":
            tables.extend(["Customer", "Employee", "Invoice", "InvoiceLine"])
    return tables
table_chain = category_chain | get_tables  # noqa
table_chain.invoke({"input": "What are all the genres of Alanis Morisette songs"})
```

现在我们已经有了一个可以输出任何查询相关表格的链条，我们可以将其与我们的[create_sql_query_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.sql_database.query.create_sql_query_chain.html)结合起来，该链条可以接受一个`table_names_to_use`列表，以确定在提示中包含哪些表模式：

```python
from operator import itemgetter
from langchain.chains import create_sql_query_chain
from langchain_core.runnables import RunnablePassthrough
query_chain = create_sql_query_chain(llm, db)
# 将“question”键转换为当前table_chain所期望的“input”键。
table_chain = {"input": itemgetter("question")} | table_chain
# 使用table_chain设置table_names_to_use。
full_chain = RunnablePassthrough.assign(table_names_to_use=table_chain) | query_chain
```
```python
query = full_chain.invoke(
    {"question": "What are all the genres of Alanis Morisette songs"}
)
print(query)
```
```python
db.run(query)
```

我们可以在此处查看此运行的LangSmith跟踪[链接](https://smith.langchain.com/public/4fbad408-3554-4f33-ab47-1e510a1b52a3/r)。

我们已经看到了如何在链条中动态包含一组表模式以在提示中使用。解决这个问题的另一个可能方法是让代理自行决定何时查找表格，方法是给它一个工具来执行。您可以在[SQL: Agents](/docs/tutorials/agents)指南中看到这种方法的示例。

## 高基数列

为了过滤包含地址、歌曲名称或艺术家等专有名词的列，我们首先需要仔细检查拼写，以便正确过滤数据。

一个简单的策略是创建一个包含数据库中所有不同专有名词的向量存储。然后，我们可以查询该向量存储每个用户输入，并将最相关的专有名词注入到提示中。

首先，我们需要每个实体的唯一值，为此我们定义一个函数，将结果解析为元素列表：

```python
import ast
import re
def query_as_list(db, query):
    res = db.run(query)
    res = [el for sub in ast.literal_eval(res) for el in sub if el]
    res = [re.sub(r"\b\d+\b", "", string).strip() for string in res]
    return res
proper_nouns = query_as_list(db, "SELECT Name FROM Artist")
proper_nouns += query_as_list(db, "SELECT Title FROM Album")
proper_nouns += query_as_list(db, "SELECT Name FROM Genre")
len(proper_nouns)
proper_nouns[:5]
```

现在我们可以将所有值嵌入和存储在一个向量数据库中：

```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
vector_db = FAISS.from_texts(proper_nouns, OpenAIEmbeddings())
retriever = vector_db.as_retriever(search_kwargs={"k": 15})
```

并组合一个查询构建链，首先从数据库中检索值并将其插入到提示中：

```python
from operator import itemgetter
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
system = """You are a SQLite expert. Given an input question, create a syntactically
correct SQLite query to run. Unless otherwise specificed, do not return more than
{top_k} rows.
Only return the SQL query with no markup or explanation.
Here is the relevant table info: {table_info}
Here is a non-exhaustive list of possible feature values. If filtering on a feature
value make sure to check its spelling against this list first:
```

{proper_nouns}

"""

## 使用链式查询

我们可以使用链式查询来尝试一下。首先，我们来看看在没有检索的情况下，过滤"elenis moriset"（Alanis Morissette的拼写错误）会发生什么情况：

```python
# 没有检索
query = query_chain.invoke(
    {"question": "elenis moriset的所有歌曲的流派是什么", "proper_nouns": ""}
)
print(query)
db.run(query)
```
```output
SELECT DISTINCT g.Name 
FROM Track t
JOIN Album a ON t.AlbumId = a.AlbumId
JOIN Artist ar ON a.ArtistId = ar.ArtistId
JOIN Genre g ON t.GenreId = g.GenreId
WHERE ar.Name = 'Elenis Moriset';
```
```output
''
```

我们可以看到，在没有检索的情况下，我们无法得到有效的结果。

接下来，我们来看看在有检索的情况下，过滤"elenis moriset"会发生什么情况：

```python
# 有检索
query = chain.invoke({"question": "elenis moriset的所有歌曲的流派是什么"})
print(query)
db.run(query)
```
```output
SELECT DISTINCT g.Name
FROM Genre g
JOIN Track t ON g.GenreId = t.GenreId
JOIN Album a ON t.AlbumId = a.AlbumId
JOIN Artist ar ON a.ArtistId = ar.ArtistId
WHERE ar.Name = 'Alanis Morissette';
```
```output
"[('Rock',)]"
```

我们可以看到，在有检索的情况下，我们能够将"elenis moriset"的拼写纠正为"Alanis Morissette"，并得到一个有效的结果。

解决这个问题的另一种可能方法是让一个Agent自己决定何时查找专有名词。您可以在[SQL: Agents](/docs/tutorials/agents)指南中看到这个问题的一个示例。