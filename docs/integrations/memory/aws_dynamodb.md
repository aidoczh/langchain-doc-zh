# AWS DynamoDB

[Amazon AWS DynamoDB](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/dynamodb/index.html) 是一种完全托管的 NoSQL 数据库服务，提供快速且可预测的性能，具有无缝的可伸缩性。

这篇笔记将介绍如何使用 DynamoDB 存储聊天消息历史，使用 DynamoDBChatMessageHistory 类。

## 设置

首先确保已正确配置 [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)。然后确保已安装 `langchain-community` 包，因此我们需要安装它。我们还需要安装 `boto3` 包。

```bash
pip install -U langchain-community boto3
```

设置 [LangSmith](https://smith.langchain.com/) 也很有帮助（但不是必需的），以获得最佳的可观察性。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

```python
from langchain_community.chat_message_histories import (
    DynamoDBChatMessageHistory,
)
```

## 创建表

现在，创建 `DynamoDB` 表，我们将在其中存储消息：

```python
import boto3
# 获取服务资源。
dynamodb = boto3.resource("dynamodb")
# 创建 DynamoDB 表。
table = dynamodb.create_table(
    TableName="SessionTable",
    KeySchema=[{"AttributeName": "SessionId", "KeyType": "HASH"}],
    AttributeDefinitions=[{"AttributeName": "SessionId", "AttributeType": "S"}],
    BillingMode="PAY_PER_REQUEST",
)
# 等待表存在。
table.meta.client.get_waiter("table_exists").wait(TableName="SessionTable")
# 打印表的一些数据。
print(table.item_count)
```

```output
0
```

## DynamoDBChatMessageHistory

```python
history = DynamoDBChatMessageHistory(table_name="SessionTable", session_id="0")
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```

```output
[HumanMessage(content='hi!'), AIMessage(content='whats up?')]
```

## 具有自定义端点 URL 的 DynamoDBChatMessageHistory

有时，指定连接到 AWS 端点的 URL 是有用的，例如在本地运行时针对 [Localstack](https://localstack.cloud/)。对于这些情况，您可以通过构造函数中的 `endpoint_url` 参数指定 URL。

```python
history = DynamoDBChatMessageHistory(
    table_name="SessionTable",
    session_id="0",
    endpoint_url="http://localhost.localstack.cloud:4566",
)
```

## 具有复合键的 DynamoDBChatMessageHistory

DynamoDBChatMessageHistory 的默认键是 `{"SessionId": self.session_id}`，但您可以修改它以匹配您的表设计。

### 主键名称

您可以通过在构造函数中传递 `primary_key_name` 值来修改主键，从而得到以下结果：`{self.primary_key_name: self.session_id}`。

### 复合键

在使用现有的 DynamoDB 表时，您可能需要将键结构从默认值修改为包含排序键的某些内容。为此，您可以使用 `key` 参数。

通过为键传递一个值将覆盖 `primary_key` 参数，结果键结构将是传递的值。

```python
composite_table = dynamodb.create_table(
    TableName="CompositeTable",
    KeySchema=[
        {"AttributeName": "PK", "KeyType": "HASH"},
        {"AttributeName": "SK", "KeyType": "RANGE"},
    ],
    AttributeDefinitions=[
        {"AttributeName": "PK", "AttributeType": "S"},
        {"AttributeName": "SK", "AttributeType": "S"},
    ],
    BillingMode="PAY_PER_REQUEST",
)
# 等待表存在。
composite_table.meta.client.get_waiter("table_exists").wait(TableName="CompositeTable")
# 打印表的一些数据。
print(composite_table.item_count)
```

```output
0
```

```python
my_key = {
    "PK": "session_id::0",
    "SK": "langchain_history",
}
composite_key_history = DynamoDBChatMessageHistory(
    table_name="CompositeTable",
    session_id="0",
    endpoint_url="http://localhost.localstack.cloud:4566",
    key=my_key,
)
composite_key_history.add_user_message("hello, composite dynamodb table!")
composite_key_history.messages
```

```output
[HumanMessage(content='hello, composite dynamodb table!')]
```

## 链接

我们可以轻松地将此消息历史类与 [LCEL Runnables](/docs/how_to/message_history) 结合使用。

为此，我们将使用 OpenAI，因此我们需要安装它。

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)
chain = prompt | ChatOpenAI()
```

```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: DynamoDBChatMessageHistory(
        table_name="SessionTable", session_id=session_id
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

```python
# 这里是配置会话 ID 的地方
config = {"configurable": {"session_id": "<SESSION_ID>"}}
```

```python
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
```

```output
AIMessage(content='Hello Bob! How can I assist you today?')
```

```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```

```output
AIMessage(content='Your name is Bob! Is there anything specific you would like assistance with, Bob?')
```