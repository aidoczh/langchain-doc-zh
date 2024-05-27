# 在 SQL 数据上构建问答系统

使一个 LLM 系统能够查询结构化数据可能与非结构化文本数据有质的不同。在后者中，通常会生成可针对向量数据库进行搜索的文本，而对于结构化数据的方法通常是让 LLM 编写并执行 DSL（如 SQL）中的查询。在本指南中，我们将介绍在数据库中的表格数据上创建问答系统的基本方法。我们将涵盖使用链和代理的实现。这些系统将允许我们提出关于数据库中数据的问题，并获得自然语言回答。两者之间的主要区别在于我们的代理可以在循环中查询数据库，以便多次回答问题。

## ⚠️ 安全提示 ⚠️

构建 SQL 数据库的问答系统需要执行模型生成的 SQL 查询。这样做存在固有风险。确保您的数据库连接权限始终尽可能地针对链/代理的需求进行了范围限定。这将减轻但不会消除构建模型驱动系统的风险。有关一般安全最佳实践的更多信息，请参阅[此处](/docs/security)。

## 架构

在高层次上，这些系统的步骤如下：

1. **将问题转换为 DSL 查询**：模型将用户输入转换为 SQL 查询。

2. **执行 SQL 查询**：执行查询。

3. **回答问题**：模型使用查询结果回应用户输入的问题。

请注意，查询 CSV 数据可以采用类似的方法。有关更多细节，请参阅我们的[操作指南](/docs/how_to/sql_csv)中关于在 CSV 数据上进行问答的内容。

![sql_usecase.png](../../static/img/sql_usecase.png)

## 设置

首先，获取所需的软件包并设置环境变量：

```python
%pip install --upgrade --quiet  langchain langchain-community langchain-openai
```

我们将在本指南中使用 OpenAI 模型。

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
# 取消下面的注释以使用 LangSmith。非必需。
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

下面的示例将使用 SQLite 连接和 Chinook 数据库。按照[这些安装步骤](https://database.guide/2-sample-databases-sqlite/)在与此笔记本相同的目录中创建 `Chinook.db`：

- 将[此文件](https://raw.githubusercontent.com/lerocha/chinook-database/master/ChinookDatabase/DataSources/Chinook_Sqlite.sql)保存为 `Chinook.sql`

- 运行 `sqlite3 Chinook.db`

- 运行 `.read Chinook.sql`

- 测试 `SELECT * FROM Artist LIMIT 10;`

现在，`Chinook.db` 在我们的目录中，我们可以使用基于 SQLAlchemy 的 `SQLDatabase` 类与其进行交互：

```python
from langchain_community.utilities import SQLDatabase
db = SQLDatabase.from_uri("sqlite:///Chinook.db")
print(db.dialect)
print(db.get_usable_table_names())
db.run("SELECT * FROM Artist LIMIT 10;")
```

```output
sqlite
['Album', 'Artist', 'Customer', 'Employee', 'Genre', 'Invoice', 'InvoiceLine', 'MediaType', 'Playlist', 'PlaylistTrack', 'Track']
```

```output
"[(1, 'AC/DC'), (2, 'Accept'), (3, 'Aerosmith'), (4, 'Alanis Morissette'), (5, 'Alice In Chains'), (6, 'Antônio Carlos Jobim'), (7, 'Apocalyptica'), (8, 'Audioslave'), (9, 'BackBeat'), (10, 'Billy Cobham')]"
```

太棒了！我们有一个可以查询的 SQL 数据库。现在让我们尝试将其连接到一个 LLM。

## 链 {#chains}

链（即 LangChain [Runnables](/docs/concepts#langchain-expression-language) 的组合）支持其步骤可预测的应用程序。我们可以创建一个简单的链，它接受一个问题并执行以下操作：

- 将问题转换为 SQL 查询；

- 执行查询；

- 使用结果回答原始问题。

这种安排不支持某些场景。例如，该系统将为任何用户输入执行 SQL 查询，甚至是“你好”。重要的是，正如我们将在下面看到的，有些问题需要多个查询才能回答。我们将在代理部分解决这些场景。

### 将问题转换为 SQL 查询

SQL 链或代理的第一步是接受用户输入并将其转换为 SQL 查询。LangChain 自带了一个用于此目的的内置链：[create_sql_query_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.sql_database.query.create_sql_query_chain)。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

```python
from langchain.chains import create_sql_query_chain
chain = create_sql_query_chain(llm, db)
response = chain.invoke({"question": "有多少名员工"})
response
```

```output
'SELECT COUNT("EmployeeId") AS "TotalEmployees" FROM "Employee"\nLIMIT 1;'
```

我们可以执行查询以确保其有效：

```python
db.run(response)
```

```output
'[(8,)]'
```

我们可以查看 [LangSmith trace](https://smith.langchain.com/public/c8fa52ea-be46-4829-bde2-52894970b830/r) 来更好地了解这个链正在做什么。我们也可以直接检查链的提示。从下面的提示中，我们可以看到它是：

- 方言特定。在这种情况下，它明确引用了 SQLite。

- 对所有可用表格进行了定义。

- 每个表格有三个示例行。

这种技术受到了像 [这篇论文](https://arxiv.org/pdf/2204.00498.pdf) 这样的论文的启发，该论文建议展示示例行并明确表格可以提高性能。我们还可以这样检查完整的提示：

```python
chain.get_prompts()[0].pretty_print()
```

```output
你是一个 SQLite 专家。给定一个输入问题，首先创建一个语法正确的 SQLite 查询来运行，然后查看查询的结果并返回输入问题的答案。
除非用户在问题中指定了要获取的示例数量，否则使用 LIMIT 子句最多查询 5 个结果，根据 SQLite 的规定。您可以对结果进行排序，以返回数据库中最具信息量的数据。
永远不要从表格中查询所有列。您必须只查询需要回答问题的列。将每个列名用双引号 (") 括起来，以将它们标记为分隔标识符。
注意只使用您在下面表格中看到的列名。小心不要查询不存在的列。还要注意哪个列在哪个表格中。
注意使用 date('now') 函数来获取当前日期，如果问题涉及“今天”。
使用以下格式：
问题：问题在这里
SQL查询：要运行的 SQL 查询
SQL结果：SQL查询的结果
答案：最终答案在这里
只使用以下表格：
{table_info}
问题：{input}
```

### 执行 SQL 查询

现在我们已经生成了一个 SQL 查询，我们希望执行它。**这是创建 SQL 链中最危险的部分。**请仔细考虑是否可以在数据上运行自动查询。尽量最小化数据库连接权限。在查询执行之前考虑添加人工批准步骤（见下文）。

我们可以使用 `QuerySQLDatabaseTool` 轻松地将查询执行添加到我们的链中：

```python
from langchain_community.tools.sql_database.tool import QuerySQLDataBaseTool
execute_query = QuerySQLDataBaseTool(db=db)
write_query = create_sql_query_chain(llm, db)
chain = write_query | execute_query
chain.invoke({"question": "有多少员工"})
```

```output
'[(8,)]'
```

### 回答问题

现在我们已经有了自动生成和执行查询的方法，我们只需要将原始问题和 SQL 查询结果结合起来生成最终答案。我们可以通过再次将问题和结果传递给 LLM 来实现这一点：

```python
from operator import itemgetter
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnablePassthrough
answer_prompt = PromptTemplate.from_template(
    """给定以下用户问题，相应的 SQL 查询和 SQL 结果，回答用户问题。
问题：{question}
SQL 查询：{query}
SQL 结果：{result}
答案："""
)
chain = (
    RunnablePassthrough.assign(query=write_query).assign(
        result=itemgetter("query") | execute_query
    )
    | answer_prompt
    | llm
    | StrOutputParser()
)
chain.invoke({"question": "有多少员工"})
```

```output
'一共有 8 名员工。'
```

让我们回顾一下上述 LCEL 中发生了什么。假设调用了这个链。

- 在第一个 `RunnablePassthrough.assign` 之后，我们有一个包含两个元素的可运行对象：

  `{"question": question, "query": write_query.invoke(question)}`

  其中 `write_query` 将生成一个用于回答问题的 SQL 查询。

- 在第二个 `RunnablePassthrough.assign` 之后，我们添加了一个包含 `execute_query.invoke(query)` 的第三个元素 `"result"`，其中 `query` 是在上一步中计算的。

- 这三个输入被格式化为提示，并传递给 LLM。

- `StrOutputParser()` 提取出输出消息的字符串内容。

请注意，我们正在组合 LLM、工具、提示和其他链，但由于每个都实现了 Runnable 接口，它们的输入和输出可以以合理的方式相互关联。

### 下一步

对于更复杂的查询生成，我们可能希望创建 few-shot prompts 或添加查询检查步骤。有关此类高级技术和更多信息，请查看：

- [提示策略](/docs/how_to/sql_prompting)：高级提示工程技术。

- [查询检查](/docs/how_to/sql_query_checking)：添加查询验证和错误处理。

* [大型数据库](/docs/how_to/sql_large_db)：处理大型数据库的技术。

## 代理 {#agents}

LangChain拥有一个SQL代理，它提供了一种比链式交互更灵活的方式来与SQL数据库进行交互。使用SQL代理的主要优势包括：

- 它可以根据数据库的模式和内容（比如描述特定表）来回答问题。

- 它可以通过运行生成的查询、捕获回溯并正确地重新生成来从错误中恢复。

- 它可以查询数据库多次以回答用户问题。

- 它将通过仅从相关表中检索模式来节省令牌。

要初始化代理，我们将使用`SQLDatabaseToolkit`来创建一系列工具：

* 创建和执行查询

* 检查查询语法

* 检索表描述

* ...等等

```python
from langchain_community.agent_toolkits import SQLDatabaseToolkit
toolkit = SQLDatabaseToolkit(db=db, llm=llm)
tools = toolkit.get_tools()
tools
```

```output
[QuerySQLDataBaseTool(description="输入到此工具的是详细且正确的SQL查询，输出是来自数据库的结果。如果查询不正确，将返回错误消息。如果返回错误，请重新编写查询，检查查询，然后重试。如果遇到“Unknown column 'xxxx' in 'field list'”的问题，请使用sql_db_schema查询正确的表字段。", db=<langchain_community.utilities.sql_database.SQLDatabase object at 0x113403b50>),
 InfoSQLDatabaseTool(description='输入到此工具的是逗号分隔的表列表，输出是这些表的模式和示例行。确保通过调用sql_db_list_tables来检查表是否实际存在！示例输入：table1, table2, table3', db=<langchain_community.utilities.sql_database.SQLDatabase object at 0x113403b50>),
 ListSQLDatabaseTool(db=<langchain_community.utilities.sql_database.SQLDatabase object at 0x113403b50>),
 QuerySQLCheckerTool(description='在执行查询之前，使用此工具来双重检查查询是否正确。始终在使用sql_db_query执行查询之前使用此工具！', db=<langchain_community.utilities.sql_database.SQLDatabase object at 0x113403b50>, llm=ChatOpenAI(client=<openai.resources.chat.completions.Completions object at 0x115b7e890>, async_client=<openai.resources.chat.completions.AsyncCompletions object at 0x115457e10>, temperature=0.0, openai_api_key=SecretStr('**********'), openai_proxy=''), llm_chain=LLMChain(prompt=PromptTemplate(input_variables=['dialect', 'query'], template='\n{query}\nDouble check the {dialect} query above for common mistakes, including:\n- Using NOT IN with NULL values\n- Using UNION when UNION ALL should have been used\n- Using BETWEEN for exclusive ranges\n- Data type mismatch in predicates\n- Properly quoting identifiers\n- Using the correct number of arguments for functions\n- Casting to the correct data type\n- Using the proper columns for joins\n\nIf there are any of the above mistakes, rewrite the query. If there are no mistakes, just reproduce the original query.\n\nOutput the final SQL query only.\n\nSQL Query: '), llm=ChatOpenAI(client=<openai.resources.chat.completions.Completions object at 0x115b7e890>, async_client=<openai.resources.chat.completions.AsyncCompletions object at 0x115457e10>, temperature=0.0, openai_api_key=SecretStr('**********'), openai_proxy='')))]
```

### 系统提示

我们还需要为我们的代理创建一个系统提示。这将包括行为指示。

```python
from langchain_core.messages import SystemMessage
SQL_PREFIX = """您是一个专门与SQL数据库交互的代理。
给定一个输入问题，创建一个语法正确的SQLite查询来运行，然后查看查询的结果并返回答案。
除非用户指定他们希望获得的特定数量的示例，否则始终将查询限制在最多5个结果。
您可以按相关列对结果进行排序，以返回数据库中最有趣的示例。
永远不要查询特定表的所有列，只根据问题要求的相关列进行查询。
您可以使用与数据库交互的工具。
只使用以下工具。只使用以下工具返回的信息来构建您的最终答案。
在执行查询之前，您必须仔细检查您的查询。如果在执行查询时出现错误，请重新编写查询并重试。
不要对数据库进行任何DML语句（INSERT、UPDATE、DELETE、DROP等）。
首先，您应该始终查看数据库中的表，以了解您可以查询的内容。
不要跳过此步骤。
然后，您应该查询最相关表的模式。"""
system_message = SystemMessage(content=SQL_PREFIX)
```

### 初始化代理

首先，获取所需的包**LangGraph**

```python
%pip install --upgrade --quiet langgraph
```

我们将使用一个预构建的 [LangGraph](/docs/concepts/#langgraph) 代理来构建我们的代理。

```python
from langchain_core.messages import HumanMessage
from langgraph.prebuilt import chat_agent_executor
agent_executor = chat_agent_executor.create_tool_calling_executor(
    llm, tools, messages_modifier=system_message
)
```

考虑代理如何回答以下问题：

```python
for s in agent_executor.stream(
    {"messages": [HumanMessage(content="Which country's customers spent the most?")]}
):
    print(s)
    print("----")
```

```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_vnHKe3oul1xbpX0Vrb2vsamZ', 'function': {'arguments': '{"query":"SELECT c.Country, SUM(i.Total) AS Total_Spent FROM customers c JOIN invoices i ON c.CustomerId = i.CustomerId GROUP BY c.Country ORDER BY Total_Spent DESC LIMIT 1"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 53, 'prompt_tokens': 557, 'total_tokens': 610}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-da250593-06b5-414c-a9d9-3fc77036dd9c-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT c.Country, SUM(i.Total) AS Total_Spent FROM customers c JOIN invoices i ON c.CustomerId = i.CustomerId GROUP BY c.Country ORDER BY Total_Spent DESC LIMIT 1'}, 'id': 'call_vnHKe3oul1xbpX0Vrb2vsamZ'}])]}}
----
{'action': {'messages': [ToolMessage(content='Error: (sqlite3.OperationalError) no such table: customers\n[SQL: SELECT c.Country, SUM(i.Total) AS Total_Spent FROM customers c JOIN invoices i ON c.CustomerId = i.CustomerId GROUP BY c.Country ORDER BY Total_Spent DESC LIMIT 1]\n(Background on this error at: https://sqlalche.me/e/20/e3q8)', name='sql_db_query', id='1a5c85d4-1b30-4af3-ab9b-325cbce3b2b4', tool_call_id='call_vnHKe3oul1xbpX0Vrb2vsamZ')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_pp3BBD1hwpdwskUj63G3tgaQ', 'function': {'arguments': '{}', 'name': 'sql_db_list_tables'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 12, 'prompt_tokens': 699, 'total_tokens': 711}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-04cf0e05-61d0-4673-b5dc-1a9b5fd71fff-0', tool_calls=[{'name': 'sql_db_list_tables', 'args': {}, 'id': 'call_pp3BBD1hwpdwskUj63G3tgaQ'}])]}}
----
{'action': {'messages': [ToolMessage(content='Album, Artist, Customer, Employee, Genre, Invoice, InvoiceLine, MediaType, Playlist, PlaylistTrack, Track', name='sql_db_list_tables', id='c2668450-4d73-4d32-8d75-8aac8fa153fd', tool_call_id='call_pp3BBD1hwpdwskUj63G3tgaQ')]}}
----
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_22Asbqgdx26YyEvJxBuANVdY', 'function': {'arguments': '{"query":"SELECT c.Country, SUM(i.Total) AS Total_Spent FROM Customer c JOIN Invoice i ON c.CustomerId = i.CustomerId GROUP BY c.Country ORDER BY Total_Spent DESC LIMIT 1"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 53, 'prompt_tokens': 744, 'total_tokens': 797}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-bdd94241-ca49-4f15-b31a-b7c728a34ea8-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': 'SELECT c.Country, SUM(i.Total) AS Total_Spent FROM Customer c JOIN Invoice i ON c.CustomerId = i.CustomerId GROUP BY c.Country ORDER BY Total_Spent DESC LIMIT 1'}, 'id': 'call_22Asbqgdx26YyEvJxBuANVdY'}])]}}
----
{'action': {'messages': [ToolMessage(content="[('USA', 523.0600000000003)]", name='sql_db_query', id='f647e606-8362-40ab-8d34-612ff166dbe1', tool_call_id='call_22Asbqgdx26YyEvJxBuANVdY')]}}
----
{'agent': {'messages': [AIMessage(content='Customers from the USA spent the most, with a total amount spent of $523.06.', response_metadata={'token_usage': {'completion_tokens': 20, 'prompt_tokens': 819, 'total_tokens': 839}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-92e88de0-ff62-41da-8181-053fb5632af4-0')]}}
----
```

请注意，代理执行多个查询，直到获得所需的信息：

1. 列出可用的表；

2. 检索三个表的模式；

3. 通过连接操作查询多个表。

然后代理能够使用最终查询的结果来生成对原始问题的回答。

代理也能够处理类似的定性问题：

```python
for s in agent_executor.stream(
    {"messages": [HumanMessage(content="Describe the playlisttrack table")]}
):
    print(s)
    print("----")
```

```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_WN0N3mm8WFvPXYlK9P7KvIEr', 'function': {'arguments': '{"table_names":"playlisttrack"}', 'name': 'sql_db_schema'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 17, 'prompt_tokens': 554, 'total_tokens': 571}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-be278326-4115-4c67-91a0-6dc97e7bffa4-0', tool_calls=[{'name': 'sql_db_schema', 'args': {'table_names': 'playlisttrack'}, 'id': 'call_WN0N3mm8WFvPXYlK9P7KvIEr'}])]}}
----
{'action': {'messages': [ToolMessage(content="Error: table_names {'playlisttrack'} not found in database", name='sql_db_schema', id='fe32b3d3-a40f-4802-a6b8-87a2453af8c2', tool_call_id='call_WN0N3mm8WFvPXYlK9P7KvIEr')]}}
----
{'agent': {'messages': [AIMessage(content='I apologize for the error. Let me first check the available tables in the database.', additional_kwargs={'tool_calls': [{'id': 'call_CzHt30847ql2MmnGxgYeVSL2', 'function': {'arguments': '{}', 'name': 'sql_db_list_tables'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 30, 'prompt_tokens': 592, 'total_tokens': 622}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-f6c107bb-e945-4848-a83c-f57daec1144e-0', tool_calls=[{'name': 'sql_db_list_tables', 'args': {}, 'id': 'call_CzHt30847ql2MmnGxgYeVSL2'}])]}}
----
{'action': {'messages': [ToolMessage(content='Album, Artist, Customer, Employee, Genre, Invoice, InvoiceLine, MediaType, Playlist, PlaylistTrack, Track', name='sql_db_list_tables', id='a4950f74-a0ad-4558-ba54-7bcf99539a02', tool_call_id='call_CzHt30847ql2MmnGxgYeVSL2')]}}
----
{'agent': {'messages': [AIMessage(content='The database contains a table named "PlaylistTrack". Let me retrieve the schema and sample rows from the "PlaylistTrack" table.', additional_kwargs={'tool_calls': [{'id': 'call_wX9IjHLgRBUmxlfCthprABRO', 'function': {'arguments': '{"table_names":"PlaylistTrack"}', 'name': 'sql_db_schema'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 44, 'prompt_tokens': 658, 'total_tokens': 702}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-e8d34372-1159-4654-a185-1e7d0cb70269-0', tool_calls=[{'name': 'sql_db_schema', 'args': {'table_names': 'PlaylistTrack'}, 'id': 'call_wX9IjHLgRBUmxlfCthprABRO'}])]}}
----
{'action': {'messages': [ToolMessage(content='\nCREATE TABLE "PlaylistTrack" (\n\t"PlaylistId" INTEGER NOT NULL, \n\t"TrackId" INTEGER NOT NULL, \n\tPRIMARY KEY ("PlaylistId", "TrackId"), \n\tFOREIGN KEY("TrackId") REFERENCES "Track" ("TrackId"), \n\tFOREIGN KEY("PlaylistId") REFERENCES "Playlist" ("PlaylistId")\n)\n\n/*\n3 rows from PlaylistTrack table:\nPlaylistId\tTrackId\n1\t3402\n1\t3389\n1\t3390\n*/', name='sql_db_schema', id='f6ffc37a-188a-4690-b84e-c9f2c78b1e49', tool_call_id='call_wX9IjHLgRBUmxlfCthprABRO')]}}
----
{'agent': {'messages': [AIMessage(content='The "PlaylistTrack" table has the following schema:\n- PlaylistId: INTEGER (NOT NULL)\n- TrackId: INTEGER (NOT NULL)\n- Primary Key: (PlaylistId, TrackId)\n- Foreign Key: TrackId references Track(TrackId)\n- Foreign Key: PlaylistId references Playlist(PlaylistId)\n\nHere are 3 sample rows from the "PlaylistTrack" table:\n1. PlaylistId: 1, TrackId: 3402\n2. PlaylistId: 1, TrackId: 3389\n3. PlaylistId: 1, TrackId: 3390\n\nIf you have any specific questions or queries regarding the "PlaylistTrack" table, feel free to ask!', response_metadata={'token_usage': {'completion_tokens': 145, 'prompt_tokens': 818, 'total_tokens': 963}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-961a4552-3cbd-4d28-b338-4d2f1ac40ea0-0')]}
----
```

### 处理高基数列

为了过滤包含专有名词（如地址、歌曲名称或艺术家）的列，我们首先需要仔细检查拼写，以便正确地过滤数据。

我们可以通过创建一个向量存储器，其中包含数据库中存在的所有不同的专有名词，来实现这一点。然后，每当用户在问题中包含专有名词时，代理可以查询该向量存储器，以找到该词的正确拼写。通过这种方式，代理可以确保在构建目标查询之前理解用户所指的实体。

首先，我们需要每个实体的唯一值，为此我们定义一个函数，将结果解析为元素列表：

```python
import ast
import re
def query_as_list(db, query):
    res = db.run(query)
    res = [el for sub in ast.literal_eval(res) for el in sub if el]
    res = [re.sub(r"\b\d+\b", "", string).strip() for string in res]
    return list(set(res))
artists = query_as_list(db, "SELECT Name FROM Artist")
albums = query_as_list(db, "SELECT Title FROM Album")
albums[:5]
```

```output
['Big Ones',
 'Cidade Negra - Hits',
 'In Step',
 'Use Your Illusion I',
 'Voodoo Lounge']
```

使用这个函数，我们可以创建一个**检索工具**，代理可以自行执行。

```python
from langchain.agents.agent_toolkits import create_retriever_tool
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
vector_db = FAISS.from_texts(artists + albums, OpenAIEmbeddings())
retriever = vector_db.as_retriever(search_kwargs={"k": 5})
description = """用于查找要过滤的值。输入是专有名词的近似拼写，输出是有效的专有名词。使用与搜索最相似的名词。"""
retriever_tool = create_retriever_tool(
    retriever,
    name="search_proper_nouns",
    description=description,
)
```

让我们试一试：

```python
print(retriever_tool.invoke("Alice Chains"))
```

```output
Alice In Chains
Alanis Morissette
Pearl Jam
Pearl Jam
Audioslave
```

这样，如果代理确定需要基于类似于 "Alice Chains" 的艺术家编写过滤器，它可以首先使用检索工具观察列的相关值。

将这些组合起来：

```python
system = """您是一个设计用于与 SQL 数据库交互的代理。
给定一个输入问题，创建一个语法正确的 SQLite 查询来运行，然后查看查询的结果并返回答案。
除非用户指定希望获得的特定示例数量，否则始终将查询限制为最多 5 个结果。
您可以按相关列对结果进行排序，以返回数据库中最有趣的示例。
从特定表中不要查询所有列，只需根据问题请求相关的列。
您可以使用与数据库交互的工具。
只使用给定的工具。只使用工具返回的信息来构建最终答案。
在执行查询之前，您必须仔细检查您的查询。如果在执行查询时出现错误，请重新编写查询并重试。
不要对数据库进行任何 DML 语句（INSERT、UPDATE、DELETE、DROP 等）。
您可以访问以下表格：{table_names}
如果需要对专有名词进行过滤，您必须始终使用“search_proper_nouns”工具查找过滤值！
不要试图猜测专有名词的正确名称 - 使用此功能查找相似的名称。""".format(
    table_names=db.get_usable_table_names()
)
system_message = SystemMessage(content=system)
agent = chat_agent_executor.create_tool_calling_executor(
    llm, tools, messages_modifier=system_message
)
```

```python
for s in agent.stream(
    {"messages": [HumanMessage(content="Alice Chains有多少张专辑？")]}
):
    print(s)
    print("----")
```

```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_r5UlSwHKQcWDHx6LrttnqE56', 'function': {'arguments': '{"query":"SELECT COUNT(*) AS album_count FROM Album WHERE ArtistId IN (SELECT ArtistId FROM Artist WHERE Name = \'Alice In Chains\')"}', 'name': 'sql_db_query'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 40, 'prompt_tokens': 612, 'total_tokens': 652}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-548353fd-b06c-45bf-beab-46f81eb434df-0', tool_calls=[{'name': 'sql_db_query', 'args': {'query': "SELECT COUNT(*) AS album_count FROM Album WHERE ArtistId IN (SELECT ArtistId FROM Artist WHERE Name = 'Alice In Chains')"}, 'id': 'call_r5UlSwHKQcWDHx6LrttnqE56'}])]}
----
```

正如我们所看到的，代理使用了 `search_proper_nouns` 工具来检查如何正确地查询数据库以获取这位特定艺术家的信息。
