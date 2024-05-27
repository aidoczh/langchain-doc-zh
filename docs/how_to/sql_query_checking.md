# 如何在 SQL 问答中进行查询验证

也许任何 SQL 链或代理中最容易出错的部分就是编写有效和安全的 SQL 查询。在本指南中，我们将介绍一些验证查询和处理无效查询的策略。

我们将涵盖：

1. 将“查询验证器”步骤附加到查询生成中；

2. 提示工程以减少错误的发生。

## 设置

首先，获取所需的软件包并设置环境变量：

```python
%pip install --upgrade --quiet  langchain langchain-community langchain-openai
```
```python
# 取消下面的注释以使用 LangSmith。不是必需的。
# import os
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

下面的示例将使用带有 Chinook 数据库的 SQLite 连接。按照[这些安装步骤](https://database.guide/2-sample-databases-sqlite/)在与此笔记本相同的目录中创建 `Chinook.db`：

- 将[此文件](https://raw.githubusercontent.com/lerocha/chinook-database/master/ChinookDatabase/DataSources/Chinook_Sqlite.sql)保存为 `Chinook_Sqlite.sql`

- 运行 `sqlite3 Chinook.db`

- 运行 `.read Chinook_Sqlite.sql`

- 测试 `SELECT * FROM Artist LIMIT 10;`

现在，`Chinhook.db` 在我们的目录中，我们可以使用基于 SQLAlchemy 的 `SQLDatabase` 类与其进行交互：

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

## 查询检查器

也许最简单的策略是要求模型本身检查原始查询中的常见错误。假设我们有以下 SQL 查询链：

```python
from langchain.chains import create_sql_query_chain
chain = create_sql_query_chain(llm, db)
```

我们想要验证其输出。我们可以通过在链中添加第二个提示和模型调用来实现：

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
system = """Double check the user's {dialect} query for common mistakes, including:
- Using NOT IN with NULL values
- Using UNION when UNION ALL should have been used
- Using BETWEEN for exclusive ranges
- Data type mismatch in predicates
- Properly quoting identifiers
- Using the correct number of arguments for functions
- Casting to the correct data type
- Using the proper columns for joins
If there are any of the above mistakes, rewrite the query.
If there are no mistakes, just reproduce the original query with no further commentary.
Output the final SQL query only."""
prompt = ChatPromptTemplate.from_messages(
    [("system", system), ("human", "{query}")]
).partial(dialect=db.dialect)
validation_chain = prompt | llm | StrOutputParser()
full_chain = {"query": chain} | validation_chain
```
```python
query = full_chain.invoke(
    {
        "question": "What's the average Invoice from an American customer whose Fax is missing since 2003 but before 2010"
    }
)
print(query)
```
```output
SELECT AVG(i.Total) AS AverageInvoice
FROM Invoice i
JOIN Customer c ON i.CustomerId = c.CustomerId
WHERE c.Country = 'USA'
AND c.Fax IS NULL
AND i.InvoiceDate >= '2003-01-01' 
AND i.InvoiceDate < '2010-01-01'
```

请注意，我们可以在[Langsmith 追踪](https://smith.langchain.com/public/8a743295-a57c-4e4c-8625-bc7e36af9d74/r)中看到链的两个步骤。

```python
db.run(query)
```
```output
'[(6.632999999999998,)]'
```

这种方法的明显缺点是我们需要进行两次模型调用而不是一次来生成我们的查询。为了解决这个问题，我们可以尝试在单个模型调用中执行查询生成和查询检查：

```python
system = """You are a {dialect} expert. Given an input question, creat a syntactically correct {dialect} query to run.
Unless the user specifies in the question a specific number of examples to obtain, query for at most {top_k} results using the LIMIT clause as per {dialect}. You can order the results to return the most informative data in the database.
Never query for all columns from a table. You must query only the columns that are needed to answer the question. Wrap each column name in double quotes (") to denote them as delimited identifiers.
Pay attention to use only the column names you can see in the tables below. Be careful to not query for columns that do not exist. Also, pay attention to which column is in which table.
Pay attention to use date('now') function to get the current date, if the question involves "today".
Only use the following tables:
{table_info}
Write an initial draft of the query. Then double check the {dialect} query for common mistakes, including:
- Using NOT IN with NULL values
- Using UNION when UNION ALL should have been used
- Using BETWEEN for exclusive ranges
- Data type mismatch in predicates
- Properly quoting identifiers
- Using the correct number of arguments for functions
- Casting to the correct data type
- Using the proper columns for joins
Use format:
First draft: <<FIRST_DRAFT_QUERY>>
Final answer: <<FINAL_ANSWER_QUERY>>
"""
prompt = ChatPromptTemplate.from_messages(
    [("system", system), ("human", "{input}")]
).partial(dialect=db.dialect)
def parse_final_answer(output: str) -> str:
    return output.split("Final answer: ")[1]
chain = create_sql_query_chain(llm, db, prompt=prompt) | parse_final_answer
prompt.pretty_print()
```
```markdown
## 系统消息
你是一位{方言}专家。根据输入问题，创建一个符合语法规范的{方言}查询来运行。
除非用户在问题中指定要获取的特定示例数量，否则使用 LIMIT 子句查询最多{top_k}个结果。你可以对结果进行排序，以返回数据库中最有信息量的数据。
永远不要从表中查询所有列。你必须只查询回答问题所需的列。将每个列名用双引号(")括起来，以将它们表示为分隔标识符。
注意只使用你可以在下表中看到的列名。小心不要查询不存在的列。还要注意哪个列在哪个表中。
注意使用 date('now') 函数来获取当前日期，如果问题涉及"今天"的话。
只使用以下表格：
{table_info}
编写查询的初始草稿。然后仔细检查{方言}查询中的常见错误，包括：
- 在 NULL 值上使用 NOT IN
- 在应该使用 UNION ALL 的地方使用 UNION
- 在排他性范围上使用 BETWEEN
- 谓词中的数据类型不匹配
- 正确引用标识符
- 对函数使用正确数量的参数
- 转换为正确的数据类型
- 对连接使用正确的列
使用格式：
初稿：<<FIRST_DRAFT_QUERY>>
最终答案：<<FINAL_ANSWER_QUERY>>
## 人工消息
{input}
```python
query = chain.invoke(
    {
        "question": "What's the average Invoice from an American customer whose Fax is missing since 2003 but before 2010"
    }
)
print(query)
```
```markdown

```
```python
db.run(query)
```
```markdown

```
```python
'[(6.632999999999998,)]'
```
## 人在回路中
在某些情况下，我们的数据足够敏感，以至于我们永远不希望在没有人员批准的情况下执行 SQL 查询。前往[工具使用：人在回路中](/docs/how_to/tools_human)页面，了解如何向任何工具、链或代理添加人在回路。
## 错误处理
在某个时刻，模型会出错并创建一个无效的 SQL 查询。或者我们的数据库出现问题。或者模型 API 出现故障。我们希望为我们的链和代理添加一些错误处理行为，以便在这些情况下优雅地失败，甚至可能自动恢复。要了解有关工具的错误处理，请前往[工具使用：错误处理](/docs/how_to/tools_error)页面。
```