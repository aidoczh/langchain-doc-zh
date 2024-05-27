# 如何在 CSV 文件上进行问答
LLM 在构建各种数据源上的问答系统方面表现出色。在本节中，我们将介绍如何在存储在 CSV 文件中的数据上构建问答系统。与使用 SQL 数据库类似，处理 CSV 文件的关键是让 LLM 能够访问工具以查询和与数据交互。实现这一目标的两种主要方法是：
* **推荐**：将 CSV 文件加载到 SQL 数据库中，并使用 [SQL 教程](/docs/tutorials/sql_qa) 中概述的方法。
* 让 LLM 访问 Python 环境，使其能够使用诸如 Pandas 等库与数据交互。
本指南将涵盖这两种方法。
## ⚠️ 安全提示 ⚠️
上述两种方法都存在重大风险。使用 SQL 需要执行模型生成的 SQL 查询。使用诸如 Pandas 的库需要让模型执行 Python 代码。由于限制 SQL 连接权限和清理 SQL 查询比隔离 Python 环境更容易，**我们强烈建议通过 SQL 与 CSV 数据交互**。有关一般安全最佳实践，请参阅[此处](/docs/security)。
## 设置
本指南的依赖项：
```python
%pip install -qU langchain langchain-openai langchain-community langchain-experimental pandas
```
设置所需的环境变量：
```python
# 推荐使用 LangSmith，但不是必需的。取消下面的注释以使用。
# import os
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```
如果尚未拥有，下载 [泰坦尼克号数据集](https://www.kaggle.com/datasets/yasserh/titanic-dataset)：
```python
!wget https://web.stanford.edu/class/archive/cs/cs109/cs109.1166/stuff/titanic.csv -O titanic.csv
```
```python
import pandas as pd
df = pd.read_csv("titanic.csv")
print(df.shape)
print(df.columns.tolist())
```
```output
(887, 8)
['Survived', 'Pclass', 'Name', 'Sex', 'Age', 'Siblings/Spouses Aboard', 'Parents/Children Aboard', 'Fare']
```
## SQL
使用 SQL 与 CSV 数据交互是推荐的方法，因为限制权限和清理查询比使用任意 Python 更容易。
大多数 SQL 数据库都可以轻松将 CSV 文件加载为表（[DuckDB](https://duckdb.org/docs/data/csv/overview.html)、[SQLite](https://www.sqlite.org/csv.html) 等）。加载完成后，您可以使用 [SQL 教程](/docs/tutorials/sql_qa) 中概述的所有链和代理创建技术。以下是我们如何在 SQLite 中执行此操作的快速示例：
```python
from langchain_community.utilities import SQLDatabase
from sqlalchemy import create_engine
engine = create_engine("sqlite:///titanic.db")
df.to_sql("titanic", engine, index=False)
```
```output
887
```
```python
db = SQLDatabase(engine=engine)
print(db.dialect)
print(db.get_usable_table_names())
print(db.run("SELECT * FROM titanic WHERE Age < 2;"))
```
```output
sqlite
['titanic']
[(1, 2, 'Master. Alden Gates Caldwell', 'male', 0.83, 0, 2, 29.0), (0, 3, 'Master. Eino Viljami Panula', 'male', 1.0, 4, 1, 39.6875), (1, 3, 'Miss. Eleanor Ileen Johnson', 'female', 1.0, 1, 1, 11.1333), (1, 2, 'Master. Richard F Becker', 'male', 1.0, 2, 1, 39.0), (1, 1, 'Master. Hudson Trevor Allison', 'male', 0.92, 1, 2, 151.55), (1, 3, 'Miss. Maria Nakid', 'female', 1.0, 0, 2, 15.7417), (0, 3, 'Master. Sidney Leonard Goodwin', 'male', 1.0, 5, 2, 46.9), (1, 3, 'Miss. Helene Barbara Baclini', 'female', 0.75, 2, 1, 19.2583), (1, 3, 'Miss. Eugenie Baclini', 'female', 0.75, 2, 1, 19.2583), (1, 2, 'Master. Viljo Hamalainen', 'male', 0.67, 1, 1, 14.5), (1, 3, 'Master. Bertram Vere Dean', 'male', 1.0, 1, 2, 20.575), (1, 3, 'Master. Assad Alexander Thomas', 'male', 0.42, 0, 1, 8.5167), (1, 2, 'Master. Andre Mallet', 'male', 1.0, 0, 2, 37.0042), (1, 2, 'Master. George Sibley Richards', 'male', 0.83, 1, 1, 18.75)]
```
创建一个 [SQL 代理](/docs/tutorials/sql_qa) 与之交互：
```python
import ChatModelTabs from "@theme/ChatModelTabs";
<ChatModelTabs customVarName="llm" />
from langchain_community.agent_toolkits import create_sql_agent
agent_executor = create_sql_agent(llm, db=db, agent_type="openai-tools", verbose=True)
```
```python
agent_executor.invoke({"input": "what's the average age of survivors"})
```
```output
> 进入新的 SQL 代理执行链...
调用：`sql_db_list_tables`，参数为 `{}`
titanic
调用：`sql_db_schema`，参数为 `{'table_names': 'titanic'}`
CREATE TABLE titanic (
	"Survived" BIGINT, 
	"Pclass" BIGINT, 
	"Name" TEXT, 
	"Sex" TEXT, 
	"Age" FLOAT, 
	"Siblings/Spouses Aboard" BIGINT, 
	"Parents/Children Aboard" BIGINT, 
	"Fare" FLOAT
)
/*
来自泰坦尼克号表的 3 行数据：
Survived	Pclass	Name	Sex	Age	Siblings/Spouses Aboard	Parents/Children Aboard	Fare
0	3	Mr. Owen Harris Braund	male	22.0	1	0	7.25
1	1	Mrs. John Bradley (Florence Briggs Thayer) Cumings	female	38.0	1	0	71.2833
1	3	Miss. Laina Heikkinen	female	26.0	0	0	7.925
*/
调用：`sql_db_query`，参数为 `{'query': 'SELECT AVG(Age) AS Average_Age FROM titanic WHERE Survived = 1'}`
[(28.408391812865496,)]
泰坦尼克号数据集中幸存者的平均年龄约为 28.41 岁。
> 链结束。
```
## Pandas
除了使用 SQL 外，我们还可以使用像 pandas 这样的数据分析库和 LLM 的代码生成能力来处理 CSV 数据。同样，**除非你有充分的保护措施，否则这种方法不适用于生产环境**。因此，我们的代码执行工具和构造函数位于 `langchain-experimental` 包中。
### Chain
大多数 LLM 都经过了足够多的 pandas Python 代码训练，只要被要求，它们就可以生成这些代码：
```python
ai_msg = llm.invoke(
    "我有一个名为 'df' 的 pandas DataFrame，它有 'Age' 和 'Fare' 两列。请编写代码计算这两列之间的相关性。返回一个 Python 代码片段的 Markdown，不要返回其他内容。"
)
print(ai_msg.content)
```
```python
correlation = df['Age'].corr(df['Fare'])
correlation
```
我们可以将这种能力与 Python 执行工具结合起来，创建一个简单的数据分析链。首先，我们需要将 CSV 表加载为一个 dataframe，并让工具可以访问这个 dataframe：
```python
import pandas as pd
from langchain_core.prompts import ChatPromptTemplate
from langchain_experimental.tools import PythonAstREPLTool
df = pd.read_csv("titanic.csv")
tool = PythonAstREPLTool(locals={"df": df})
tool.invoke("df['Fare'].mean()")
```
```output
32.30542018038331
```
为了帮助强制使用我们的 Python 工具，我们将使用 [tool calling](/docs/how_to/tool_calling/)：
```python
llm_with_tools = llm.bind_tools([tool], tool_choice=tool.name)
response = llm_with_tools.invoke(
    "我有一个名为 'df' 的 dataframe，想知道 'Age' 和 'Fare' 两列之间的相关性"
)
response
```
```output
AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_SBrK246yUbdnJemXFC8Iod05', 'function': {'arguments': '{"query":"df.corr()[\'Age\'][\'Fare\']"}', 'name': 'python_repl_ast'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 13, 'prompt_tokens': 125, 'total_tokens': 138}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-1fd332ba-fa72-4351-8182-d464e7368311-0', tool_calls=[{'name': 'python_repl_ast', 'args': {'query': "df.corr()['Age']['Fare']"}, 'id': 'call_SBrK246yUbdnJemXFC8Iod05'}])
```
```python
response.tool_calls
```
```output
[{'name': 'python_repl_ast',
  'args': {'query': "df.corr()['Age']['Fare']"},
  'id': 'call_SBrK246yUbdnJemXFC8Iod05'}]
```
我们将添加一个工具输出解析器，将函数调用提取为一个字典：
```python
from langchain_core.output_parsers.openai_tools import JsonOutputKeyToolsParser
parser = JsonOutputKeyToolsParser(key_name=tool.name, first_tool_only=True)
(llm_with_tools | parser).invoke(
    "我有一个名为 'df' 的 dataframe，想知道 'Age' 和 'Fare' 两列之间的相关性"
)
```
```output
{'query': "df[['Age', 'Fare']].corr()"}
```
并与提示结合起来，这样我们只需指定一个问题，而不需要在每次调用时指定 dataframe 信息：
```python
system = f"""你可以访问一个名为 `df` 的 pandas dataframe。这是 `df.head().to_markdown()` 的输出：
```
{df.head().to_markdown()}
```
给定一个用户问题，编写 Python 代码来回答它。只返回有效的 Python 代码，不要返回其他内容。不要假设你可以访问除了内置的 Python 库和 pandas 之外的任何库。"""
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", "{question}")])
code_chain = prompt | llm_with_tools | parser
code_chain.invoke({"question": "年龄和票价之间的相关性是多少"})
```
```output
{'query': "df[['Age', 'Fare']].corr()"}
```
最后，我们将添加我们的 Python 工具，以便实际执行生成的代码：
```python
chain = prompt | llm_with_tools | parser | tool  # noqa
chain.invoke({"question": "年龄和票价之间的相关性是多少"})
```
```output
0.11232863699941621
```
就这样，我们就有了一个简单的数据分析链。我们可以通过查看 LangSmith 的跟踪来查看中间步骤：https://smith.langchain.com/public/b1309290-7212-49b7-bde2-75b39a32b49a/r
我们可以在最后添加一个 LLM 调用，生成一个对话式回复，这样我们不仅仅是用工具的输出进行回复。为此，我们需要在提示中添加一个聊天历史 `MessagesPlaceholder`：
```python
from operator import itemgetter
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import ToolMessage
from langchain_core.messages import Tool
df_1 = df[["Age", "Fare"]]
df_2 = df[["Fare", "Survived"]]
tool = PythonAstREPLTool(locals={"df_1": df_1, "df_2": df_2})
llm_with_tool = llm.bind_tools(tools=[tool], tool_choice=tool.name)
df_template = """```python
{df_name}.head().to_markdown()
>>> {df_head}
```"""
df_context = "\n\n".join(
    df_template.format(df_head=_df.head().to_markdown(), df_name=df_name)
    for _df, df_name in [(df_1, "df_1"), (df_2, "df_2")]
)
system = f"""你可以访问多个 pandas 数据框。\
以下是每个数据框的样本行和生成样本的 Python 代码：
{df_context}
```
给定一个关于数据框的用户问题，编写Python代码来回答它。不要假设你可以访问除了内置的Python库和pandas之外的任何库。
确保只参考上面提到的变量。
```python
import pandas as pd
# 假设你已经有了名为df的数据框
# 计算年龄和票价之间的相关性
correlation_age_fare = df['age'].corr(df['fare'])
# 计算票价和生存之间的相关性
correlation_fare_survival = df['fare'].corr(df['survival'])
# 计算两个相关性之间的差异
correlation_difference = correlation_age_fare - correlation_fare_survival
correlation_difference
```
这里是这次运行的LangSmith跟踪：[链接](https://smith.langchain.com/public/cc2a7d7f-7c5a-4e77-a10c-7b5420fcd07f/r)
### 沙盒代码执行
有许多工具，如[E2B](/docs/integrations/tools/e2b_data_analysis)和[Bearly](/docs/integrations/tools/bearly)，提供用于Python代码执行的沙盒环境，以实现更安全的代码执行链和代理。
## 下一步
对于更高级的数据分析应用程序，我们建议查看以下内容：
- [SQL教程](/docs/tutorials/sql_qa)：许多与SQL数据库和CSV文件一起工作的挑战对于任何结构化数据类型都是通用的，因此即使您使用Pandas进行CSV数据分析，阅读SQL技术也很有用。
- [工具使用](/docs/how_to/tool_calling)：在调用工具的链和代理中使用的一般最佳实践指南
- [代理](/docs/tutorials/agents)：了解构建LLM代理的基本原理。
- 集成：像[E2B](/docs/integrations/tools/e2b_data_analysis)和[Bearly](/docs/integrations/tools/bearly)这样的沙盒环境，像[SQLDatabase](https://api.python.langchain.com/en/latest/utilities/langchain_community.utilities.sql_database.SQLDatabase.html#langchain_community.utilities.sql_database.SQLDatabase)这样的实用工具，以及像[Spark DataFrame agent](/docs/integrations/toolkits/spark)这样的相关代理。