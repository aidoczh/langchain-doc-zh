# Cassandra数据库

Apache Cassandra® 是一种广泛使用的数据库，用于存储事务性应用程序数据。在大型语言模型中引入函数和工具后，为现有数据在生成式人工智能应用程序中开辟了一些令人兴奋的用例。Cassandra数据库工具包使AI工程师能够高效地将代理与Cassandra数据集成，提供以下功能：

- 通过优化查询实现快速数据访问。大多数查询应在单个数字毫秒或更短时间内运行。

- 模式内省以增强LLM推理能力

- 与各种Cassandra部署兼容，包括Apache Cassandra®、DataStax Enterprise™和DataStax Astra™

- 目前，该工具包仅限于SELECT查询和模式内省操作。（安全第一）

## 快速开始

- 安装cassio库

- 为您连接的Cassandra数据库设置环境变量

- 初始化CassandraDatabase

- 通过toolkit.get_tools()将工具传递给您的代理

- 坐下来，看着它为您完成所有工作

## 操作理论

Cassandra查询语言（CQL）是与Cassandra数据库进行交互的主要*以人为本*的方式。虽然在生成查询时提供了一些灵活性，但它需要了解Cassandra数据建模的最佳实践。LLM函数调用赋予代理推理的能力，然后选择工具来满足请求。使用LLM的代理在选择适当的工具包或工具包链时应使用特定于Cassandra的逻辑进行推理。这减少了当LLM被迫提供自上而下的解决方案时引入的随机性。您想让LLM完全无限制地访问您的数据库吗？是的。可能不。为了实现这一点，我们在为代理构建问题时提供了一个提示：

```json
您是一个Apache Cassandra专家查询分析机器人，具有以下功能和规则：
- 您将接受最终用户关于在数据库中查找特定数据的问题。
- 您将检查数据库的模式并创建查询路径。
- 您将为用户提供正确的查询，以查找他们正在寻找的数据，显示查询路径提供的步骤。
- 您将使用查询Apache Cassandra的最佳实践，包括分区键和聚簇列。
- 避免在查询中使用ALLOW FILTERING。
- 目标是找到一个查询路径，因此可能需要查询其他表才能得到最终答案。
```

以下是JSON格式中查询路径的示例：

```json
{
  "query_paths": [
    {
      "description": "Direct query to users table using email",
      "steps": [
        {
          "table": "user_credentials",
          "query": 
             "SELECT userid FROM user_credentials WHERE email = 'example@example.com';"
        },
        {
          "table": "users",
          "query": "SELECT * FROM users WHERE userid = ?;"
        }
      ]
    }
  ]
}
```

## 提供的工具

### `cassandra_db_schema`

收集连接的数据库或特定模式的所有模式信息。在确定操作时对代理至关重要。

### `cassandra_db_select_table_data`

从特定keyspace和表中选择数据。代理可以传递谓词和返回记录数量的限制。

### `cassandra_db_query`

对`cassandra_db_select_table_data`的实验性替代方案，它完全由代理形成查询字符串。*警告*：这可能导致不太有效（甚至无法工作）的异常查询。这在将来的版本中可能会被移除。如果它有一些很酷的功能，我们也想知道。你永远不知道！

## 环境设置

安装以下Python模块：

```bash
pip install ipykernel python-dotenv cassio langchain_openai langchain langchain-community langchainhub
```

### .env文件

连接通过`cassio`，使用`auto=True`参数，笔记本使用OpenAI。您应相应地创建一个`.env`文件。

对于Cassandra，设置：

```bash
CASSANDRA_CONTACT_POINTS
CASSANDRA_USERNAME
CASSANDRA_PASSWORD
CASSANDRA_KEYSPACE
```

对于Astra，设置：

```bash
ASTRA_DB_APPLICATION_TOKEN
ASTRA_DB_DATABASE_ID
ASTRA_DB_KEYSPACE
```

例如：

```bash
# 连接到Astra：
ASTRA_DB_DATABASE_ID=a1b2c3d4-...
ASTRA_DB_APPLICATION_TOKEN=AstraCS:...
ASTRA_DB_KEYSPACE=notebooks
# 也设置
OPENAI_API_KEY=sk-....
```

（您还可以修改下面的代码，直接连接到`cassio`。）

```python
from dotenv import load_dotenv
load_dotenv(override=True)
```

```python
# 导入必要的库
import os
import cassio
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_community.agent_toolkits.cassandra_database.toolkit import (
    CassandraDatabaseToolkit,
)
from langchain_community.tools.cassandra_database.prompt import QUERY_PATH_PROMPT
from langchain_community.tools.cassandra_database.tool import (
    GetSchemaCassandraDatabaseTool,
    GetTableDataCassandraDatabaseTool,
    QueryCassandraDatabaseTool,
)
from langchain_community.utilities.cassandra_database import CassandraDatabase
from langchain_openai import ChatOpenAI
```

## 连接到Cassandra数据库

```python
cassio.init(auto=True)
session = cassio.config.resolve_session()
if not session:
    raise Exception("检查环境配置或手动配置cassio连接参数")
```

```python
# 测试数据 pep
session = cassio.config.resolve_session()
session.execute("""DROP KEYSPACE IF EXISTS langchain_agent_test; """)
session.execute(
    """
CREATE KEYSPACE if not exists langchain_agent_test 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
"""
)
session.execute(
    """
CREATE TABLE IF NOT EXISTS langchain_agent_test.user_credentials (
user_email text PRIMARY KEY,
user_id UUID,
password TEXT
);
"""
)
session.execute(
    """
CREATE TABLE IF NOT EXISTS langchain_agent_test.users (
id UUID PRIMARY KEY,
name TEXT,
email TEXT
);"""
)
session.execute(
    """
CREATE TABLE IF NOT EXISTS langchain_agent_test.user_videos ( 
user_id UUID,
video_id UUID,
title TEXT,
description TEXT,
PRIMARY KEY (user_id, video_id)
);
"""
)
user_id = "522b1fe2-2e36-4cef-a667-cd4237d08b89"
video_id = "27066014-bad7-9f58-5a30-f63fe03718f6"
session.execute(
    f"""
INSERT INTO langchain_agent_test.user_credentials (user_id, user_email) 
VALUES ({user_id}, 'patrick@datastax.com');
"""
)
session.execute(
    f"""
INSERT INTO langchain_agent_test.users (id, name, email) 
VALUES ({user_id}, 'Patrick McFadin', 'patrick@datastax.com');
"""
)
session.execute(
    f"""
INSERT INTO langchain_agent_test.user_videos (user_id, video_id, title)
VALUES ({user_id}, {video_id}, 'Use Langflow to Build a LangChain LLM Application in 5 Minutes');
"""
)
session.set_keyspace("langchain_agent_test")
```

```python
# 创建一个CassandraDatabase实例
# 使用cassio会话连接到数据库
db = CassandraDatabase()
# 创建Cassandra数据库工具
query_tool = QueryCassandraDatabaseTool(db=db)
schema_tool = GetSchemaCassandraDatabaseTool(db=db)
select_data_tool = GetTableDataCassandraDatabaseTool(db=db)
```

```python
# 选择将驱动代理的LLM
# 仅有某些模型支持此功能
llm = ChatOpenAI(temperature=0, model="gpt-4-1106-preview")
toolkit = CassandraDatabaseToolkit(db=db)
tools = toolkit.get_tools()
print("可用工具:")
for tool in tools:
    print(tool.name + "\t- " + tool.description)
```

```output
可用工具:
cassandra_db_schema	- 
    输入此工具的是一个keyspace名称，输出是Apache Cassandra表的表描述。
    如果查询不正确，将返回错误消息。
    如果返回错误，请向用户报告keyspace不存在并停止。
cassandra_db_query	- 
    对数据库执行CQL查询并获取结果。
    如果查询不正确，将返回错误消息。
    如果返回错误，请重写查询，检查查询，然后重试。
cassandra_db_select_table_data	- 
    从Apache Cassandra数据库中的表中获取数据的工具。
    使用WHERE子句指定使用主键的查询的谓词。空谓词将返回所有行。如果可能，请避免使用此功能。
    使用limit指定要返回的行数。空限制将返回所有行。
```

```python
prompt = hub.pull("hwchase17/openai-tools-agent")
# 构建OpenAI Tools代理
agent = create_openai_tools_agent(llm, tools, prompt)
```

```python
input = (
    QUERY_PATH_PROMPT
    + "\n\n这是您的任务：查找使用电子邮件地址'patrick@datastax.com'的用户上传到langchain_agent_test keyspace的所有视频。"
)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
response = agent_executor.invoke({"input": input})
print(response["output"])
```

```json
{
  "query_paths": [
    {
      "description": "从 user_credentials 表中查找 user_id，然后查询 user_videos 表以检索用户上传的所有视频",
      "steps": [
        {
          "table": "user_credentials",
          "query": "SELECT user_id FROM user_credentials WHERE user_email = 'patrick@datastax.com';"
        },
        {
          "table": "user_videos",
          "query": "SELECT * FROM user_videos WHERE user_id = 522b1fe2-2e36-4cef-a667-cd4237d08b89;"
        }
      ]
    }
  ]
}
```

要查找使用电子邮件地址 'patrick@datastax.com' 的用户在 `langchain_agent_test` keyspace 中上传的所有视频，我们可以按照以下步骤进行：

1. 查询 `user_credentials` 表，找到与电子邮件 'patrick@datastax.com' 关联的 `user_id`。

2. 使用第一步获得的 `user_id` 来查询 `user_videos` 表，以检索用户上传的所有视频。

按照这个查询路径，我们发现具有用户ID `522b1fe2-2e36-4cef-a667-cd4237d08b89` 的用户至少上传了一个标题为 'DataStax Academy'，描述为 'DataStax Academy is a free resource for learning Apache Cassandra.' 的视频。该视频的 video_id 是 `27066014-bad7-9f58-5a30-f63fe03718f6`。如果有更多视频，可以使用相同的查询来检索它们，如果需要，可能需要增加限制。

> 链结束。

要深入了解如何创建 Cassandra DB 代理，请参阅[CQL代理手册](https://github.com/langchain-ai/langchain/blob/master/cookbook/cql_agent.ipynb)。

```