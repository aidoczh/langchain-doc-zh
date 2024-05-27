# Kinetica SqlAssist LLM 演示

本文档演示了如何使用 Kinetica 将自然语言转换为 SQL，并简化数据检索过程。此演示旨在展示创建和使用链的机制，而不是 LLM 的功能。

## 概述

使用 Kinetica LLM 工作流，您可以在数据库中创建一个 LLM 上下文，该上下文提供了推理所需的信息，包括表、注释、规则和样本。调用 `ChatKinetica.load_messages_from_context()` 将从数据库中检索上下文信息，以便用于创建聊天提示。

聊天提示由 `SystemMessage` 和一对 `HumanMessage`/`AIMessage` 组成，其中包含问题/SQL 对的样本。您可以将样本对附加到此列表中，但它不适用于典型的自然语言对话。

当您从聊天提示创建链并执行它时，Kinetica LLM 将从输入生成 SQL。可选地，您可以使用 `KineticaSqlOutputParser` 执行 SQL 并将结果作为数据帧返回。

目前，支持生成 SQL 的 LLM 有两种：

1. **Kinetica SQL-GPT**：此 LLM 基于 OpenAI ChatGPT API。

2. **Kinetica SqlAssist**：此 LLM 是专为与 Kinetica 数据库集成而构建的，并且可以在安全的客户端运行。

在本演示中，我们将使用 **SqlAssist**。有关更多信息，请参阅[Kinetica 文档站点](https://docs.kinetica.com/7.1/sql-gpt/concepts/)。

## 先决条件

要开始使用，您需要一个 Kinetica DB 实例。如果没有，可以获取[免费的开发实例](https://cloud.kinetica.com/trynow)。

您需要安装以下软件包...

```python
# 安装 Langchain 社区和核心软件包
%pip install --upgrade --quiet langchain-core langchain-community
# 安装 Kineitca DB 连接软件包
%pip install --upgrade --quiet gpudb typeguard
# 安装本教程所需的软件包
%pip install --upgrade --quiet faker
```

## 数据库连接

您必须在以下环境变量中设置数据库连接。如果使用虚拟环境，可以在项目的 `.env` 文件中设置它们：

- `KINETICA_URL`：数据库连接 URL

- `KINETICA_USER`：数据库用户

- `KINETICA_PASSWD`：安全密码。

如果您可以创建 `KineticaChatLLM` 的实例，则表示已成功连接。

```python
from langchain_community.chat_models.kinetica import ChatKinetica
kinetica_llm = ChatKinetica()
# 我们将创建的测试表
table_name = "demo.user_profiles"
# 我们将创建的 LLM 上下文
kinetica_ctx = "demo.test_llm_ctx"
```

## 创建测试数据

在我们生成 SQL 之前，我们需要创建一个 Kinetica 表和一个可以推理该表的 LLM 上下文。

### 创建一些虚假用户配置文件

我们将使用 `faker` 包创建一个包含 100 个虚假配置文件的数据帧。

```python
from typing import Generator
import pandas as pd
from faker import Faker
Faker.seed(5467)
faker = Faker(locale="en-US")
def profile_gen(count: int) -> Generator:
    for id in range(0, count):
        rec = dict(id=id, **faker.simple_profile())
        rec["birthdate"] = pd.Timestamp(rec["birthdate"])
        yield rec
load_df = pd.DataFrame.from_records(data=profile_gen(100), index="id")
print(load_df.head())
```

```output
         username             name sex  \
id                                       
0       eduardo69       Haley Beck   F   
1        lbarrera  Joshua Stephens   M   
2         bburton     Paula Kaiser   F   
3       melissa49      Wendy Reese   F   
4   melissacarter      Manuel Rios   M   
                                              address                    mail  \
id                                                                              
0   59836 Carla Causeway Suite 939\nPort Eugene, I...  meltondenise@yahoo.com   
1   3108 Christina Forges\nPort Timothychester, KY...     erica80@hotmail.com   
2                    Unit 7405 Box 3052\nDPO AE 09858  timothypotts@gmail.com   
3   6408 Christopher Hill Apt. 459\nNew Benjamin, ...        dadams@gmail.com   
4    2241 Bell Gardens Suite 723\nScottside, CA 38463  williamayala@gmail.com   
    birthdate  
id             
0  1997-12-01  
1  1924-07-27  
2  1933-11-28  
3  1988-10-19  
4  1931-03-12
```

### 从数据帧创建一个 Kinetica 表

```python
from gpudb import GPUdbTable
gpudb_table = GPUdbTable.from_df(
    load_df,
    db=kinetica_llm.kdbc,
    table_name=table_name,
    clear_table=True,
    load_data=True,
)
# 查看 Kinetica 列类型
print(gpudb_table.type_as_df())
```

```output
        name    type   properties
0   username  string     [char32]
1       name  string     [char32]
2        sex  string      [char2]
3    address  string     [char64]
4       mail  string     [char32]
5  birthdate    long  [timestamp]
```

### 创建LLM上下文

您可以使用 Kinetica Workbench UI 创建LLM上下文，也可以使用 `CREATE OR REPLACE CONTEXT` 语法手动创建。

这里我们使用SQL语法创建一个上下文，引用我们创建的表。

```python
# 为表创建LLM上下文。
sql = f"""
CREATE OR REPLACE CONTEXT {kinetica_ctx}
(
    TABLE = {table_name}
    COMMENT = '包含用户档案。'
),
(
    SAMPLES = (
    '有多少男性用户？' = 
    'select count(1) as num_users
    from {table_name}
    where sex = ''M'';')
)
"""
count_affected = kinetica_llm.kdbc.execute(sql)
count_affected
```

```output
1
```

## 使用Langchain进行推理

在下面的示例中，我们将从先前创建的表和LLM上下文创建一个链。该链将生成SQL并将结果数据作为数据框返回。

### 从Kinetica DB加载聊天提示

`load_messages_from_context()` 函数将从数据库中检索上下文，并将其转换为我们用来创建 ``ChatPromptTemplate`` 的聊天消息列表。

```python
from langchain_core.prompts import ChatPromptTemplate
# 从数据库加载上下文
ctx_messages = kinetica_llm.load_messages_from_context(kinetica_ctx)
# 添加输入提示。这是将替换输入问题的地方。
ctx_messages.append(("human", "{input}"))
# 创建提示模板。
prompt_template = ChatPromptTemplate.from_messages(ctx_messages)
prompt_template.pretty_print()
```

```output
================================ 系统消息 ================================
CREATE TABLE demo.user_profiles AS
(
   username VARCHAR (32) NOT NULL,
   name VARCHAR (32) NOT NULL,
   sex VARCHAR (2) NOT NULL,
   address VARCHAR (64) NOT NULL,
   mail VARCHAR (32) NOT NULL,
   birthdate TIMESTAMP NOT NULL
);
COMMENT ON TABLE demo.user_profiles IS '包含用户档案。';
================================ 人类消息 =================================
有多少男性用户？
================================== AI消息 ==================================
select count(1) as num_users
    from demo.user_profiles
    where sex = 'M';
================================ 人类消息 =================================
{input}
```

### 创建链

这个链的最后一个元素是 `KineticaSqlOutputParser`，它将执行SQL并返回一个数据框。这是可选的，如果我们省略它，那么只会返回SQL。

```python
from langchain_community.chat_models.kinetica import (
    KineticaSqlOutputParser,
    KineticaSqlResponse,
)
chain = prompt_template | kinetica_llm | KineticaSqlOutputParser(kdbc=kinetica_llm.kdbc)
```

### 生成SQL

我们创建的链将以问题作为输入，并返回包含生成的SQL和数据的 ``KineticaSqlResponse``。问题必须与我们用来创建提示的LLM上下文相关。

```python
# 在这里，您必须提出与提示模板中提供的LLM上下文相关的问题。
response: KineticaSqlResponse = chain.invoke(
    {"input": "女性用户按用户名排序是什么？"}
)
print(f"SQL: {response.sql}")
print(response.dataframe.head())
```

```output
SQL: SELECT username, name
    FROM demo.user_profiles
    WHERE sex = 'F'
    ORDER BY username;
      username               name
0  alexander40       Tina Ramirez
1      bburton       Paula Kaiser
2      brian12  Stefanie Williams
3    brownanna      Jennifer Rowe
4       carl19       Amanda Potts
```