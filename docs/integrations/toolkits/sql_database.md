# SQL数据库
本笔记展示了一个与 `SQL` 数据库进行交互的代理程序。
它旨在回答有关数据库的更一般性问题，并且能够从错误中恢复。
请注意，由于该代理程序正在积极开发中，所有答案可能不正确。
此外，并不能保证代理程序在特定问题下不会对您的数据库执行 DML 语句。
在运行时请小心处理敏感数据！
这里使用了示例 `Chinook` 数据库。
要设置它，请按照[这些说明](https://database.guide/2-sample-databases-sqlite/)进行操作，并将 .db 文件放在此存储库根目录下的 notebooks 文件夹中。
```python
%pip install --upgrade --quiet  langchain langchain-community langchain-experimental
```
```output
警告：检查 pip 的最新版本时出现错误。
注意：您可能需要重新启动内核以使用更新后的软件包。
```
## 初始化我们的数据库
```python
from langchain_community.utilities.sql_database import SQLDatabase
db = SQLDatabase.from_uri("sqlite:///Chinook.db")
```
## 快速入门
根据[SQL 使用案例文档](/docs/tutorials/agents)，我们可以使用 `create_sql_agent` 辅助工具。
```python
from langchain_community.agent_toolkits import create_sql_agent
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
agent_executor = create_sql_agent(llm, db=db, agent_type="openai-tools", verbose=True)
```
```python
agent_executor.invoke(
    "List the total sales per country. Which country's customers spent the most?"
)
```
```output
> 进入新的 AgentExecutor 链...
调用: `sql_db_list_tables` 参数为 `{}`
Album, Artist, Customer, Employee, Genre, Invoice, InvoiceLine, MediaType, Playlist, PlaylistTrack, Track
调用: `sql_db_schema` 参数为 `Invoice,Customer`
创建 "Customer" 表格如下：
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
/*
来自 Customer 表的 3 行数据：
CustomerId	FirstName	LastName	Company	Address	City	State	Country	PostalCode	Phone	Fax	Email	SupportRepId
1	Luís	Gonçalves	Embraer - Empresa Brasileira de Aeronáutica S.A.	Av. Brigadeiro Faria Lima, 2170	São José dos Campos	SP	Brazil	12227-000	+55 (12) 3923-5555	+55 (12) 3923-5566	luisg@embraer.com.br	3
2	Leonie	Köhler	None	Theodor-Heuss-Straße 34	Stuttgart	None	Germany	70174	+49 0711 2842222	None	leonekohler@surfeu.de	5
3	François	Tremblay	None	1498 rue Bélanger	Montréal	QC	Canada	H2G 1A7	+1 (514) 721-4711	None	ftremblay@gmail.com	3
*/
创建 "Invoice" 表格如下：
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
/*
来自 Invoice 表的 3 行数据：
InvoiceId	CustomerId	InvoiceDate	BillingAddress	BillingCity	BillingState	BillingCountry	BillingPostalCode	Total
1	2	2009-01-01 00:00:00	Theodor-Heuss-Straße 34	Stuttgart	None	Germany	70174	1.98
2	4	2009-01-02 00:00:00	Ullevålsveien 14	Oslo	None	Norway	0171	3.96
3	8	2009-01-03 00:00:00	Grétrystraat 63	Brussels	None	Belgium	1000	5.94
*/
调用: `sql_db_query` 参数为 `SELECT c.Country, SUM(i.Total) AS TotalSales FROM Invoice i JOIN Customer c ON i.CustomerId = c.CustomerId GROUP BY c.Country ORDER BY TotalSales DESC LIMIT 10;`
回答：要列出每个国家的总销售额，我可以查询 "Invoice" 和 "Customer" 表。我将在 "CustomerId" 列上连接这些表，并按 "BillingCountry" 列对结果进行分组。然后，我将计算 "Total" 列的总和以获得每个国家的总销售额。最后，我将按总销售额降序排序结果。
以下是 SQL 查询：
```sql
SELECT c.Country, SUM(i.Total) AS TotalSales
FROM Invoice i
JOIN Customer c ON i.CustomerId = c.CustomerId
GROUP BY c.Country
ORDER BY TotalSales DESC
LIMIT 10;
```
现在，我将执行此查询以获取每个国家的总销售额。
[('USA', 523.0600000000003), ('Canada', 303.9599999999999), ('France', 195.09999999999994), ('Brazil', 190.09999999999997), ('Germany', 156.48), ('United Kingdom', 112.85999999999999), ('Czech Republic', 90.24000000000001), ('Portugal', 77.23999999999998), ('India', 75.25999999999999), ('Chile', 46.62)]每个国家的总销售额如下：
1. 美国：$523.06
2. 加拿大：$303.96
3. 法国：$195.10
4. 巴西：$190.10
5. 德国：$156.48
6. 英国：$112.86
7. 捷克共和国：$90.24
8. 葡萄牙：$77.24
9. 印度：$75.26
10. 智利：$46.62
回答第二个问题，客户消费最多的国家是美国，总销售额为 $523.06。
> 链结束。
```
## 工具包
我们可以查看上面的`create_sql_agent`助手下的运行情况。
我们还可以强调工具包如何与特定的代理一起使用。
```python
from langchain_community.agent_toolkits import SQLDatabaseToolkit
from langchain_openai import ChatOpenAI
toolkit = SQLDatabaseToolkit(db=db, llm=ChatOpenAI(temperature=0))
context = toolkit.get_context()
tools = toolkit.get_tools()
```
### 在代理中使用SQLDatabaseToolkit
```python
from langchain_community.agent_toolkits.sql.prompt import SQL_FUNCTIONS_SUFFIX
from langchain_core.messages import AIMessage, SystemMessage
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    MessagesPlaceholder,
)
messages = [
    HumanMessagePromptTemplate.from_template("{input}"),
    AIMessage(content=SQL_FUNCTIONS_SUFFIX),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
]
prompt = ChatPromptTemplate.from_messages(messages)
prompt = prompt.partial(**context)
```
```python
from langchain.agents import create_openai_tools_agent
from langchain.agents.agent import AgentExecutor
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
agent = create_openai_tools_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=toolkit.get_tools(),
    verbose=True,
)
```
## 免责声明 ⚠️
查询链可能会生成插入/更新/删除查询。当不希望出现这种情况时，请使用自定义提示或创建一个没有写权限的SQL用户。
最终用户可能会通过提出一个简单的问题，如“运行最大的查询”，来过载您的SQL数据库。生成的查询可能如下所示：
```sql
SELECT * FROM "public"."users"
    JOIN "public"."user_permissions" ON "public"."users".id = "public"."user_permissions".user_id
    JOIN "public"."projects" ON "public"."users".id = "public"."projects".user_id
    JOIN "public"."events" ON "public"."projects".id = "public"."events".project_id;
```
对于事务性SQL数据库，如果上述任一表包含数百万行数据，该查询可能会对使用同一数据库的其他应用程序造成麻烦。
大多数面向数据仓库的数据库支持用户级配额，用于限制资源使用。
## 示例：描述一个表
```python
agent_executor.invoke({"input": "描述playlist表的模式"})
```
```output
> 进入新的AgentExecutor链...
调用: `sql_db_list_tables`，参数为``
Album, Artist, Customer, Employee, Genre, Invoice, InvoiceLine, MediaType, Playlist, PlaylistTrack, Track
调用: `sql_db_schema`，参数为`Playlist`
CREATE TABLE "Playlist" (
	"PlaylistId" INTEGER NOT NULL, 
	"Name" NVARCHAR(120), 
	PRIMARY KEY ("PlaylistId")
)
/*
Playlist表的3行数据:
PlaylistId	Name
1	Music
2	Movies
3	TV Shows
*/
"Playlist"表的模式如下：
- PlaylistId: INTEGER（主键）
- Name: NVARCHAR(120)
以下是"Playlist"表的三行示例数据：
| PlaylistId | Name      ||------------|-----------|| 1          | Music     || 2          | Movies    || 3          | TV Shows  |
> 完成链。
```
## 示例：描述一个表，从错误中恢复
在此示例中，代理尝试搜索一个不存在的表`playlists`，但找到了下一个最佳结果。
```python
agent_executor.invoke({"input": "描述playlists表"})
```
```output
> 进入新的AgentExecutor链...
调用: `sql_db_list_tables`，参数为``
Album, Artist, Customer, Employee, Genre, Invoice, InvoiceLine, MediaType, Playlist, PlaylistTrack, Track
调用: `sql_db_schema`，参数为`Playlist`
CREATE TABLE "Playlist" (
	"PlaylistId" INTEGER NOT NULL, 
	"Name" NVARCHAR(120), 
	PRIMARY KEY ("PlaylistId")
)
/*
Playlist表的3行数据:
PlaylistId	Name
1	Music
2	Movies
3	TV Shows
*/
"Playlists"表的模式如下：
- PlaylistId: INTEGER（主键）
- Name: NVARCHAR(120)
```
以下是“播放列表”表中的三个示例行：
| 播放列表编号 | 名称      |
|------------|-----------|
| 1          | 音乐      |
| 2          | 电影      |
| 3          | 电视节目  |
> 链条完成。
```output
{'input': '描述播放列表表',
 'output': '“播放列表”表具有以下架构：\n\n- 播放列表编号：整数（主键）\n- 名称：NVARCHAR（120）\n\n以下是“播放列表”表中的三个示例行：\n\n| 播放列表编号 | 名称      |\n|------------|-----------|\n| 1          | 音乐      |\n| 2          | 电影      |\n| 3          | 电视节目  |'}
```
## 示例：运行查询
```python
agent_executor.invoke(
    {
        "input": "列出每个国家的总销售额。哪个国家的顾客花费最多？"
    }
)
```
```output
> 进入新的 AgentExecutor 链...
调用：`sql_db_list_tables`，参数为``
Album, Artist, Customer, Employee, Genre, Invoice, InvoiceLine, MediaType, Playlist, PlaylistTrack, Track
调用：`sql_db_schema`，参数为`Invoice, Customer`
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
1	2	2009-01-01 00:00:00	Theodor-Heuss-Straße 34	Stuttgart	None	Germany	70174	1.98
2	4	2009-01-02 00:00:00	Ullevålsveien 14	Oslo	None	Norway	0171	3.96
3	8	2009-01-03 00:00:00	Grétrystraat 63	Brussels	None	Belgium	1000	5.94
*/
调用：`sql_db_query`，参数为`SELECT c.Country, SUM(i.Total) AS TotalSales FROM Customer c JOIN Invoice i ON c.CustomerId = i.CustomerId GROUP BY c.Country ORDER BY TotalSales DESC LIMIT 1;`
回复：根据“Customer”和“Invoice”表的架构，我们可以在“CustomerId”列上连接这两个表以获得每位顾客的总销售额。然后，我们可以按“Country”列对结果进行分组以获得每个国家的总销售额。最后，我们可以按总销售额降序排序结果，并选择总销售额最高的国家。
以下是执行此操作的 SQL 查询：
```sql
SELECT c.Country, SUM(i.Total) AS TotalSales
FROM Customer c
JOIN Invoice i ON c.CustomerId = i.CustomerId
GROUP BY c.Country
ORDER BY TotalSales DESC
LIMIT 1;
```
让我执行此查询，找出哪个国家的顾客花费最多。
[('USA', 523.0600000000003)]顾客花费最多的国家是美国，总销售额为523.06美元。
> 链条完成。
```
```output
{'input': '列出每个国家的总销售额。哪个国家的顾客花费最多？',
 'output': '顾客花费最多的国家是美国，总销售额为523.06美元。'}
```
## 从错误中恢复
在这个例子中，代理在最初尝试访问一个不存在的属性（`Track.ArtistId`）后能够从错误中恢复。
```python
agent_executor.invoke({"input": "谁是销量最好的前三位艺术家？"})
```
```output
> 进入新的 AgentExecutor 链...
调用：`sql_db_list_tables`，参数为``
Album, Artist, Customer, Employee, Genre, Invoice, InvoiceLine, MediaType, Playlist, PlaylistTrack, Track
调用：`sql_db_schema`，参数为`InvoiceLine`
CREATE TABLE "InvoiceLine" (
	"InvoiceLineId" INTEGER NOT NULL, 
	"InvoiceId" INTEGER NOT NULL, 
	"TrackId" INTEGER NOT NULL, 
	"UnitPrice" NUMERIC(10, 2) NOT NULL, 
	"Quantity" INTEGER NOT NULL, 
	PRIMARY KEY ("InvoiceLineId"), 
```markdown
外键("TrackId") 参考 "Track" 表中的 ("TrackId") 列，外键("InvoiceId") 参考 "Invoice" 表中的 ("InvoiceId") 列
)
/*
InvoiceLine 表中的 3 行数据:
InvoiceLineId	InvoiceId	TrackId	UnitPrice	Quantity
1	1	2	0.99	1
2	1	4	0.99	1
3	2	6	0.99	1
*/
调用: 使用 `Artist` 参数执行 `sql_db_schema`
CREATE TABLE "Artist" (
	"ArtistId" INTEGER NOT NULL, 
	"Name" NVARCHAR(120), 
	PRIMARY KEY ("ArtistId")
)
/*
Artist 表中的 3 行数据:
ArtistId	Name
1	AC/DC
2	Accept
3	Aerosmith
*/
调用: 使用 `SELECT Artist.Name, SUM(InvoiceLine.Quantity) AS TotalQuantity FROM Artist JOIN Album ON Artist.ArtistId = Album.ArtistId JOIN Track ON Album.AlbumId = Track.AlbumId JOIN InvoiceLine ON Track.TrackId = InvoiceLine.TrackId GROUP BY Artist.Name ORDER BY TotalQuantity DESC LIMIT 3` 参数执行 `sql_db_query`
[('Iron Maiden', 140), ('U2', 107), ('Metallica', 91)]销量最好的前 3 位艺术家是:
1. Iron Maiden - 总销量 140
2. U2 - 总销量 107
3. Metallica - 总销量 91
> 链结束.
```
```output
{'input': 'Who are the top 3 best selling artists?',
 'output': '销量最好的前 3 位艺术家是:\n\n1. Iron Maiden - 总销量 140\n2. U2 - 总销量 107\n3. Metallica - 总销量 91'}
```