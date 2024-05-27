# 如何在 SQL 问题回答中更好地提示

在本指南中，我们将介绍一些提示策略，以改进使用 [create_sql_query_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.sql_database.query.create_sql_query_chain.html) 生成 SQL 查询的方法。我们将主要关注如何在提示中获取相关的特定于数据库的信息。

我们将涵盖以下内容：

- LangChain 的 [SQLDatabase](https://api.python.langchain.com/en/latest/utilities/langchain_community.utilities.sql_database.SQLDatabase.html) 方言如何影响链的提示；

- 如何使用 `SQLDatabase.get_context` 将模式信息格式化到提示中；

- 如何构建和选择少量示例来辅助模型。

## 设置

首先，获取所需的包并设置环境变量：

```python
%pip install --upgrade --quiet  langchain langchain-community langchain-experimental langchain-openai
```

```python
# 取消下面的注释以使用 LangSmith。非必需。
# import os
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

下面的示例将使用 Chinook 数据库的 SQLite 连接。按照 [这些安装步骤](https://database.guide/2-sample-databases-sqlite/) 在与此笔记本相同的目录中创建 `Chinook.db`：

- 将 [此文件](https://raw.githubusercontent.com/lerocha/chinook-database/master/ChinookDatabase/DataSources/Chinook_Sqlite.sql) 保存为 `Chinook_Sqlite.sql`

- 运行 `sqlite3 Chinook.db`

- 运行 `.read Chinook_Sqlite.sql`

- 测试 `SELECT * FROM Artist LIMIT 10;`

现在，`Chinhook.db` 在我们的目录中，我们可以使用基于 SQLAlchemy 的 `SQLDatabase` 类与其交互：

```python
from langchain_community.utilities import SQLDatabase
db = SQLDatabase.from_uri("sqlite:///Chinook.db", sample_rows_in_table_info=3)
print(db.dialect)
print(db.get_usable_table_names())
print(db.run("SELECT * FROM Artist LIMIT 10;"))
```

```output
sqlite
['Album', 'Artist', 'Customer', 'Employee', 'Genre', 'Invoice', 'InvoiceLine', 'MediaType', 'Playlist', 'PlaylistTrack', 'Track']
[(1, 'AC/DC'), (2, 'Accept'), (3, 'Aerosmith'), (4, 'Alanis Morissette'), (5, 'Alice In Chains'), (6, 'Antônio Carlos Jobim'), (7, 'Apocalyptica'), (8, 'Audioslave'), (9, 'BackBeat'), (10, 'Billy Cobham')]
```

## 特定于方言的提示

我们可以做的最简单的事情之一是使我们的提示特定于我们正在使用的 SQL 方言。当使用内置的 [create_sql_query_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.sql_database.query.create_sql_query_chain.html) 和 [SQLDatabase](https://api.python.langchain.com/en/latest/utilities/langchain_community.utilities.sql_database.SQLDatabase.html) 时，这对于以下任何方言都会为您处理：

```python
from langchain.chains.sql_database.prompt import SQL_PROMPTS
list(SQL_PROMPTS)
```

```output
['crate', 'duckdb', 'googlesql', 'mssql', 'mysql', 'mariadb', 'oracle', 'postgresql', 'sqlite', 'clickhouse', 'prestodb']
```

例如，使用我们当前的数据库，我们可以看到我们将获得一个特定于 SQLite 的提示。

```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="llm" />
from langchain.chains import create_sql_query_chain
chain = create_sql_query_chain(llm, db)
chain.get_prompts()[0].pretty_print()
```

```output
You are a SQLite expert. Given an input question, first create a syntactically correct SQLite query to run, then look at the results of the query and return the answer to the input question.
Unless the user specifies in the question a specific number of examples to obtain, query for at most 5 results using the LIMIT clause as per SQLite. You can order the results to return the most informative data in the database.
Never query for all columns from a table. You must query only the columns that are needed to answer the question. Wrap each column name in double quotes (") to denote them as delimited identifiers.
Pay attention to use only the column names you can see in the tables below. Be careful to not query for columns that do not exist. Also, pay attention to which column is in which table.
Pay attention to use date('now') function to get the current date, if the question involves "today".
Use the following format:
Question: Question here
SQLQuery: SQL Query to run
SQLResult: Result of the SQLQuery
Answer: Final answer here
Only use the following tables:
{table_info}
Question: {input}
```

## 表定义和示例行

在大多数 SQL 链中，我们需要向模型提供至少部分数据库模式。如果没有这个，它将无法编写有效的查询。我们的数据库带有一些方便的方法，可以为我们提供相关的上下文。具体来说，我们可以获取表名、它们的模式以及每个表的一部分行的示例。

在这里，我们将使用 `SQLDatabase.get_context`，该函数提供了可用表格及其模式：

```python
context = db.get_context()
print(list(context))
print(context["table_info"])
```

```output
['table_info', 'table_names']
CREATE TABLE "Album" (
	"AlbumId" INTEGER NOT NULL, 
	"Title" NVARCHAR(160) NOT NULL, 
	"ArtistId" INTEGER NOT NULL, 
	PRIMARY KEY ("AlbumId"), 
	FOREIGN KEY("ArtistId") REFERENCES "Artist" ("ArtistId")
)
/*
Album 表中的 3 行数据：
AlbumId	Title	ArtistId
1	For Those About To Rock We Salute You	1
2	Balls to the Wall	2
3	Restless and Wild	2
*/
CREATE TABLE "Artist" (
	"ArtistId" INTEGER NOT NULL, 
	"Name" NVARCHAR(120), 
	PRIMARY KEY ("ArtistId")
)
/*
Artist 表中的 3 行数据：
ArtistId	Name
1	AC/DC
2	Accept
3	Aerosmith
*/
CREATE TABLE "Customer" (
	"CustomerId" INTEGER NOT NULL, 
	"FirstName" NVARCHAR(40) NOT NULL, 
	"LastName" NVARCHAR(20) NOT NULL, 
	"Company" NVARCHAR(80), 
	"Address" NVARCHAR(70), 
	"City" NVARCHAR(40), 
	"State" NVARCHAR(40), 
	"Country" NVARCHAR(40), 
	"PostalCode" NVARCHAR(10), 
	"Phone" NVARCHAR(24), 
	"Fax" NVARCHAR(24), 
	"Email" NVARCHAR(60) NOT NULL, 
	"SupportRepId" INTEGER, 
	PRIMARY KEY ("CustomerId"), 
	FOREIGN KEY("SupportRepId") REFERENCES "Employee" ("EmployeeId")
)
/*
Customer 表中的 3 行数据：
CustomerId	FirstName	LastName	Company	Address	City	State	Country	PostalCode	Phone	Fax	Email	SupportRepId
1	Luís	Gonçalves	Embraer - Empresa Brasileira de Aeronáutica S.A.	Av. Brigadeiro Faria Lima, 2170	São José dos Campos	SP	Brazil	12227-000	+55 (12) 3923-5555	+55 (12) 3923-5566	luisg@embraer.com.br	3
2	Leonie	Köhler	None	Theodor-Heuss-Straße 34	Stuttgart	None	Germany	70174	+49 0711 2842222	None	leonekohler@surfeu.de	5
3	François	Tremblay	None	1498 rue Bélanger	Montréal	QC	Canada	H2G 1A7	+1 (514) 721-4711	None	ftremblay@gmail.com	3
*/
CREATE TABLE "Employee" (
	"EmployeeId" INTEGER NOT NULL, 
	"LastName" NVARCHAR(20) NOT NULL, 
	"FirstName" NVARCHAR(20) NOT NULL, 
	"Title" NVARCHAR(30), 
	"ReportsTo" INTEGER, 
	"BirthDate" DATETIME, 
	"HireDate" DATETIME, 
	"Address" NVARCHAR(70), 
	"City" NVARCHAR(40), 
	"State" NVARCHAR(40), 
	"Country" NVARCHAR(40), 
	"PostalCode" NVARCHAR(10), 
	"Phone" NVARCHAR(24), 
	"Fax" NVARCHAR(24), 
	"Email" NVARCHAR(60), 
	PRIMARY KEY ("EmployeeId"), 
	FOREIGN KEY("ReportsTo") REFERENCES "Employee" ("EmployeeId")
)
/*
Employee 表中的 3 行数据：
EmployeeId	LastName	FirstName	Title	ReportsTo	BirthDate	HireDate	Address	City	State	Country	PostalCode	Phone	Fax	Email
1	Adams	Andrew	General Manager	None	1962-02-18 00:00:00	2002-08-14 00:00:00	11120 Jasper Ave NW	Edmonton	AB	Canada	T5K 2N1	+1 (780) 428-9482	+1 (780) 428-3457	andrew@chinookcorp.com
2	Edwards	Nancy	Sales Manager	1	1958-12-08 00:00:00	2002-05-01 00:00:00	825 8 Ave SW	Calgary	AB	Canada	T2P 2T3	+1 (403) 262-3443	+1 (403) 262-3322	nancy@chinookcorp.com
3	Peacock	Jane	Sales Support Agent	2	1973-08-29 00:00:00	2002-04-01 00:00:00	1111 6 Ave SW	Calgary	AB	Canada	T2P 5M5	+1 (403) 262-3443	+1 (403) 262-6712	jane@chinookcorp.com
*/
CREATE TABLE "Genre" (
	"GenreId" INTEGER NOT NULL, 
	"Name" NVARCHAR(120), 
	PRIMARY KEY ("GenreId")
)
/*
Genre 表中的 3 行数据：
GenreId	Name
1	Rock
2	Jazz
3	Metal
*/
CREATE TABLE "Invoice" (
	"InvoiceId" INTEGER NOT NULL, 
	"CustomerId" INTEGER NOT NULL, 
	"InvoiceDate" DATETIME NOT NULL, 
	"BillingAddress" NVARCHAR(70), 
	"BillingCity" NVARCHAR(40), 
	"BillingState" NVARCHAR(40), 
	"BillingCountry" NVARCHAR(40), 
	"BillingPostalCode" NVARCHAR(10), 
	"Total" NUMERIC(10, 2) NOT NULL, 
	PRIMARY KEY ("InvoiceId"), 
	FOREIGN KEY("CustomerId") REFERENCES "Customer" ("CustomerId")
)
/*
Invoice 表中的 3 行数据：
InvoiceId	CustomerId	InvoiceDate	BillingAddress	BillingCity	BillingState	BillingCountry	BillingPostalCode	Total
1	2	2021-01-01 00:00:00	Theodor-Heuss-Straße 34	Stuttgart	None	Germany	70174	1.98
2	4	2021-01-02 00:00:00	Ullevålsveien 14	Oslo	None	Norway	0171	3.96
3	8	2021-01-03 00:00:00	Grétrystraat 63	Brussels	None	Belgium	1000	5.94
*/
CREATE TABLE "InvoiceLine" (
	"InvoiceLineId" INTEGER NOT NULL, 
	"InvoiceId" INTEGER NOT NULL, 
	"TrackId" INTEGER NOT NULL, 
	"UnitPrice" NUMERIC(10, 2) NOT NULL, 
	"Quantity" INTEGER NOT NULL, 
	PRIMARY KEY ("InvoiceLineId"), 
	FOREIGN KEY("TrackId") REFERENCES "Track" ("TrackId"), 
	FOREIGN KEY("InvoiceId") REFERENCES "Invoice" ("InvoiceId")
)
/*
InvoiceLine 表中的 3 行数据：
InvoiceLineId	InvoiceId	TrackId	UnitPrice	Quantity
1	1	2	0.99	1
2	1	4	0.99	1
3	2	6	0.99	1
*/
CREATE TABLE "MediaType" (
	"MediaTypeId" INTEGER NOT NULL, 
	"Name" NVARCHAR(120), 
	PRIMARY KEY ("MediaTypeId")
)
/*
MediaType 表中的 3 行数据：
MediaTypeId	Name
1	MPEG audio file
2	Protected AAC audio file
3	Protected MPEG-4 video file
*/
CREATE TABLE "Playlist" (
	"PlaylistId" INTEGER NOT NULL, 
	"Name" NVARCHAR(120), 
	PRIMARY KEY ("PlaylistId")
)
/*
Playlist 表中的 3 行数据：
PlaylistId	Name
1	Music
2	Movies
3	TV Shows
*/
CREATE TABLE "PlaylistTrack" (
	"PlaylistId" INTEGER NOT NULL, 
	"TrackId" INTEGER NOT NULL, 
	PRIMARY KEY ("PlaylistId", "TrackId"), 
	FOREIGN KEY("TrackId") REFERENCES "Track" ("TrackId"), 
	FOREIGN KEY("PlaylistId") REFERENCES "Playlist" ("PlaylistId")
)
/*
PlaylistTrack 表中的 3 行数据：
PlaylistId	TrackId
1	3402
1	3389
1	3390
*/
CREATE TABLE "Track" (
	"TrackId" INTEGER NOT NULL, 
	"Name" NVARCHAR(200) NOT NULL, 
	"AlbumId" INTEGER, 
	"MediaTypeId" INTEGER NOT NULL, 
	"GenreId" INTEGER, 
	"Composer" NVARCHAR(220), 
	"Milliseconds" INTEGER NOT NULL, 
	"Bytes" INTEGER, 
	"UnitPrice" NUMERIC(10, 2) NOT NULL, 
	PRIMARY KEY ("TrackId"), 
	FOREIGN KEY("MediaTypeId") REFERENCES "MediaType" ("MediaTypeId"), 
	FOREIGN KEY("GenreId") REFERENCES "Genre" ("GenreId"), 
	FOREIGN KEY("AlbumId") REFERENCES "Album" ("AlbumId")
)
/*
Track 表中的 3 行数据：
TrackId	Name	AlbumId	MediaTypeId	GenreId	Composer	Milliseconds	Bytes	UnitPrice
1	For Those About To Rock (We Salute You)	1	1	1	Angus Young, Malcolm Young, Brian Johnson	343719	11170334	0.99
2	Balls to the Wall	2	2	1	U. Dirkschneider, W. Hoffmann, H. Frank, P. Baltes, S. Kaufmann, G. Hoffmann	342562	5510424	0.99
3	Fast As a Shark	3	2	1	F. Baltes, S. Kaufman, U. Dirkscneider & W. Hoffman	230619	3990994	0.99
```

当我们没有太多或太宽的表格时，我们可以直接在提示中插入所有这些信息：

```python
prompt_with_context = chain.get_prompts()[0].partial(table_info=context["table_info"])
print(prompt_with_context.pretty_repr()[:1500])
```

```output
你是一个 SQLite 专家。给定一个输入问题，首先创建一个语法正确的 SQLite 查询来运行，然后查看查询结果并返回输入问题的答案。
除非用户在问题中指定要获取的特定示例数量，否则使用 LIMIT 子句最多查询 5 个结果，按照 SQLite 的规定。您可以对结果进行排序，以返回数据库中最具信息量的数据。
永远不要从表中查询所有列。您必须仅查询回答问题所需的列。将每个列名用双引号 (") 括起来，以将它们标记为分隔标识符。
注意只使用您可以在下面表格中看到的列名。小心不要查询不存在的列。还要注意哪个列在哪个表中。
注意使用 date('now') 函数来获取当前日期，如果问题涉及"今天"的话。
使用以下格式：
问题：这里是问题
SQL查询：要运行的 SQL 查询
SQL结果：SQL查询的结果
答案：这里是最终答案
只使用以下表格：
CREATE TABLE "Album" (
	"AlbumId" INTEGER NOT NULL, 
	"Title" NVARCHAR(160) NOT NULL, 
	"ArtistId" INTEGER NOT NULL, 
	PRIMARY KEY ("AlbumId"), 
	FOREIGN KEY("ArtistId") REFERENCES "Artist" ("ArtistId")
)
/*
Album 表中的 3 行：
AlbumId	Title	ArtistId
1	For Those About To Rock We Salute You	1
2	Balls to the Wall	2
3	Restless and Wild	2
*/
CREATE TABLE "Artist" (
	"ArtistId" INTEGER NOT NULL, 
	"Name" NVARCHAR(120)
```

当我们有太大无法适应模型上下文窗口的数据库模式时，我们需要想办法根据用户输入仅插入相关的表定义到提示中。有关更多信息，请参阅[许多表格、宽表格、高基数特性](/docs/how_to/sql_large_db)指南。

## 少样本示例

在提示中包含将自然语言问题转换为针对数据库的有效 SQL 查询的示例，通常会提高模型性能，特别是对于复杂查询。

假设我们有以下示例：

```python
examples = [
    {"input": "列出所有艺术家。", "query": "SELECT * FROM Artist;"},
    {
        "input": "查找艺术家'AC/DC'的所有专辑。",
        "query": "SELECT * FROM Album WHERE ArtistId = (SELECT ArtistId FROM Artist WHERE Name = 'AC/DC');",
    },
    {
        "input": "列出'摇滚'流派中的所有曲目。",
        "query": "SELECT * FROM Track WHERE GenreId = (SELECT GenreId FROM Genre WHERE Name = 'Rock');",
    },
    {
        "input": "查找所有曲目的总时长。",
        "query": "SELECT SUM(Milliseconds) FROM Track;",
    },
    {
        "input": "列出所有来自加拿大的客户。",
        "query": "SELECT * FROM Customer WHERE Country = 'Canada';",
    },
    {
        "input": "ID为5的专辑中有多少曲目？",
        "query": "SELECT COUNT(*) FROM Track WHERE AlbumId = 5;",
    },
    {
        "input": "查找发票的总数。",
        "query": "SELECT COUNT(*) FROM Invoice;",
    },
    {
        "input": "列出所有时长超过5分钟的曲目。",
        "query": "SELECT * FROM Track WHERE Milliseconds > 300000;",
    },
    {
        "input": "前5位购买总额最高的客户是谁？",
        "query": "SELECT CustomerId, SUM(Total) AS TotalPurchase FROM Invoice GROUP BY CustomerId ORDER BY TotalPurchase DESC LIMIT 5;",
    },
    {
        "input": "哪些专辑来自2000年？",
        "query": "SELECT * FROM Album WHERE strftime('%Y', ReleaseDate) = '2000';",
    },
    {
        "input": "有多少名员工",
        "query": 'SELECT COUNT(*) FROM "Employee"',
    },
]
```

我们可以像这样使用它们创建少样本提示：

```python
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate
example_prompt = PromptTemplate.from_template("用户输入：{input}\nSQL查询：{query}")
prompt = FewShotPromptTemplate(
    examples=examples[:5],
    example_prompt=example_prompt,
    prefix="你是一个 SQLite 专家。给定一个输入问题，创建一个语法正确的 SQLite 查询来运行。除非另有说明，不要返回超过 {top_k} 行。\n\n这是相关的表信息：{table_info}\n\n以下是一些问题及其相应 SQL 查询的示例。",
    suffix="用户输入：{input}\nSQL查询：",
    input_variables=["input", "top_k", "table_info"],
)
```

```python
print(prompt.format(input="有多少位艺术家？", top_k=3, table_info="foo"))
```

```output
你是一位SQLite专家。根据输入的问题，创建一个语法正确的SQLite查询来运行。除非另有说明，不要返回超过3行的结果。
以下是一些问题及其相应的SQL查询的示例。
用户输入：列出所有艺术家。
SQL查询：SELECT * FROM Artist;
用户输入：找到所有属于艺术家'AC/DC'的专辑。
SQL查询：SELECT * FROM Album WHERE ArtistId = (SELECT ArtistId FROM Artist WHERE Name = 'AC/DC');
用户输入：列出所有'Rock'流派的曲目。
SQL查询：SELECT * FROM Track WHERE GenreId = (SELECT GenreId FROM Genre WHERE Name = 'Rock');
用户输入：计算所有曲目的总时长。
SQL查询：SELECT SUM(Milliseconds) FROM Track;
用户输入：列出所有来自加拿大的顾客。
SQL查询：SELECT * FROM Customer WHERE Country = 'Canada';
用户输入：有多少位艺术家？
SQL查询：
```

## 动态few-shot示例

如果我们有足够的示例，我们可能只想在提示中包含最相关的示例，要么是因为它们不适合模型的上下文窗口，要么是因为长尾示例会分散模型的注意力。具体而言，对于任何输入，我们都希望包含与该输入最相关的示例。

我们可以使用ExampleSelector来实现这一点。在这种情况下，我们将使用SemanticSimilarityExampleSelector，它将示例存储在我们选择的向量数据库中。在运行时，它将在输入和我们的示例之间执行语义相似性搜索，并返回最相似的示例。

我们在这里默认使用OpenAI嵌入，但您可以将其替换为您选择的模型提供商。

```python
from langchain_community.vectorstores import FAISS
from langchain_core.example_selectors import SemanticSimilarityExampleSelector
from langchain_openai import OpenAIEmbeddings
example_selector = SemanticSimilarityExampleSelector.from_examples(
    examples,
    OpenAIEmbeddings(),
    FAISS,
    k=5,
    input_keys=["input"],
)
```

```python
example_selector.select_examples({"input": "有多少位艺术家？"})
```

```output
[{'input': '列出所有艺术家。', 'query': 'SELECT * FROM Artist;'},
 {'input': '有多少位员工',
  'query': 'SELECT COUNT(*) FROM "Employee"'},
 {'input': '专辑ID为5的专辑中有多少首曲目？',
  'query': 'SELECT COUNT(*) FROM Track WHERE AlbumId = 5;'},
 {'input': '哪些专辑是2000年的？',
  'query': "SELECT * FROM Album WHERE strftime('%Y', ReleaseDate) = '2000';"},
 {'input': "列出所有'Rock'流派的曲目。",
  'query': "SELECT * FROM Track WHERE GenreId = (SELECT GenreId FROM Genre WHERE Name = 'Rock');"}]
```

要使用它，我们可以直接将ExampleSelector传递给FewShotPromptTemplate：

```python
prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="你是一位SQLite专家。根据输入的问题，创建一个语法正确的SQLite查询来运行。除非另有说明，不要返回超过{top_k}行的结果。\n\n以下是一些问题及其相应的SQL查询。\n\n用户输入：{input}\nSQL查询：",
    suffix="",
    input_variables=["input", "top_k"],
)
```

```python
print(prompt.format(input="有多少位艺术家？", top_k=3))
```

```output
你是一位SQLite专家。根据输入的问题，创建一个语法正确的SQLite查询来运行。除非另有说明，不要返回超过3行的结果。
以下是一些问题及其相应的SQL查询。
用户输入：列出所有艺术家。
SQL查询：SELECT * FROM Artist;
用户输入：有多少位员工
SQL查询：SELECT COUNT(*) FROM "Employee"
用户输入：专辑ID为5的专辑中有多少首曲目？
SQL查询：SELECT COUNT(*) FROM Track WHERE AlbumId = 5;
```

试一下，我们可以看到模型识别出了相关的表：

```python
chain = create_sql_query_chain(llm, db, prompt)
chain.invoke({"question": "有多少位艺术家？"})
```

```output
'SELECT COUNT(*) FROM Artist;'
```