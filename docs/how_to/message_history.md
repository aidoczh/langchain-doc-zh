

# 如何添加消息历史

:::info 先决条件

本指南假定您熟悉以下概念：

- [LangChain 表达语言 (LCEL)](/docs/concepts/#langchain-expression-language)

- [链接可运行项](/docs/how_to/sequence/)

- [在运行时配置链参数](/docs/how_to/configure)

- [提示模板](/docs/concepts/#prompt-templates)

- [聊天消息](/docs/concepts/#message-types)

:::

在构建聊天机器人时，将对话状态传递到链中以及从链中传出对话状态至关重要。[`RunnableWithMessageHistory`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html#langchain_core.runnables.history.RunnableWithMessageHistory) 类让我们能够向某些类型的链中添加消息历史。它包装另一个 Runnable 并管理其聊天消息历史。

具体来说，它可用于任何接受以下之一作为输入的 Runnable：

* 一系列 [`BaseMessages`](/docs/concepts/#message-types)

* 具有以序列 `BaseMessages` 作为值的键的字典

* 具有以字符串或序列 `BaseMessages` 作为最新消息的值的键和一个接受历史消息的单独键的字典

并将以下之一作为输出返回：

* 可视为 `AIMessage` 内容的字符串

* 一系列 `BaseMessage`

* 具有包含一系列 `BaseMessage` 的键的字典

让我们看一些示例以了解其工作原理。首先，我们构建一个 Runnable（此处接受字典作为输入并返回消息作为输出）：

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai.chat_models import ChatOpenAI
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You're an assistant who's good at {ability}. Respond in 20 words or fewer",
        ),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{input}"),
    ]
)
runnable = prompt | model
```

要管理消息历史，我们需要：

1. 此 Runnable；

2. 一个返回 `BaseChatMessageHistory` 实例的可调用对象。

请查看 [memory integrations](https://integrations.langchain.com/memory) 页面，了解使用 Redis 和其他提供程序实现聊天消息历史的方法。这里我们演示使用内存中的 `ChatMessageHistory` 以及使用 `RedisChatMessageHistory` 进行更持久存储。

## 内存中

下面我们展示一个简单的示例，其中聊天历史保存在内存中，此处通过全局 Python 字典实现。

我们构建一个名为 `get_session_history` 的可调用对象，引用此字典以返回 `ChatMessageHistory` 实例。通过在运行时向 `RunnableWithMessageHistory` 传递配置，可以指定可调用对象的参数。默认情况下，期望配置参数是一个字符串 `session_id`。可以通过 `history_factory_config` 关键字参数进行调整。

使用单参数默认值：

```python
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
store = {}
def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]
with_message_history = RunnableWithMessageHistory(
    runnable,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history",
)
```

请注意，我们已指定了 `input_messages_key`（要视为最新输入消息的键）和 `history_messages_key`（要添加历史消息的键）。

在调用此新 Runnable 时，我们通过配置参数指定相应的聊天历史：

```python
with_message_history.invoke(
    {"ability": "math", "input": "What does cosine mean?"},
    config={"configurable": {"session_id": "abc123"}},
)
```
```output
AIMessage(content='Cosine is a trigonometric function that represents the ratio of the adjacent side to the hypotenuse of a right triangle.', response_metadata={'id': 'msg_017rAM9qrBTSdJ5i1rwhB7bT', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 32, 'output_tokens': 31}}, id='run-65e94a5e-a804-40de-ba88-d01b6cd06864-0')
```
```python
# 记住
with_message_history.invoke(
    {"ability": "math", "input": "What?"},
    config={"configurable": {"session_id": "abc123"}},
)
```
```output
AIMessage(content='Cosine is a trigonometric function that represents the ratio of the adjacent side to the hypotenuse of a right triangle.', response_metadata={'id': 'msg_017hK1Q63ganeQZ9wdeqruLP', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 68, 'output_tokens': 31}}, id='run-a42177ef-b04a-4968-8606-446fb465b943-0')
```python
# 新的 session_id --> 不记得了。
with_message_history.invoke(
    {"ability": "math", "input": "What?"},
    config={"configurable": {"session_id": "def234"}},
)
```
```output

AIMessage(content="我是一位擅长数学的 AI 助手。您有什么数学相关的问题需要帮忙吗？", response_metadata={'id': 'msg_01AYwfQ6SH5qz8ZQMW3nYtGU', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 28, 'output_tokens': 24}}, id='run-c57d93e3-305f-4c0e-bdb9-ef82f5b49f61-0')

```
我们可以通过向 `history_factory_config` 参数传递一个 `ConfigurableFieldSpec` 对象列表来自定义跟踪消息历史的配置参数。下面我们使用了两个参数：`user_id` 和 `conversation_id`。
```python
from langchain_core.runnables import ConfigurableFieldSpec
store = {}
def get_session_history(user_id: str, conversation_id: str) -> BaseChatMessageHistory:
    if (user_id, conversation_id) not in store:
        store[(user_id, conversation_id)] = ChatMessageHistory()
    return store[(user_id, conversation_id)]
with_message_history = RunnableWithMessageHistory(
    runnable,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history",
    history_factory_config=[
        ConfigurableFieldSpec(
            id="user_id",
            annotation=str,
            name="User ID",
            description="用户的唯一标识符。",
            default="",
            is_shared=True,
        ),
        ConfigurableFieldSpec(
            id="conversation_id",
            annotation=str,
            name="Conversation ID",
            description="对话的唯一标识符。",
            default="",
            is_shared=True,
        ),
    ],
)
with_message_history.invoke(
    {"ability": "math", "input": "Hello"},
    config={"configurable": {"user_id": "123", "conversation_id": "1"}},
)
```
```output

AIMessage(content='你好！有什么数学问题我可以帮忙解决吗？', response_metadata={'id': 'msg_01UdhnwghuSE7oRM57STFhHL', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 27, 'output_tokens': 14}}, id='run-3d53f67a-4ea7-4d78-8e67-37db43d4af5d-0')

```
### 不同签名的可运行示例
上述可运行对象以字典作为输入，并返回一个 BaseMessage。下面我们展示一些替代方法。
#### 输入为消息，输出为字典
```python
from langchain_core.messages import HumanMessage
from langchain_core.runnables import RunnableParallel
chain = RunnableParallel({"output_message": model})
def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]
with_message_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    output_messages_key="output_message",
)
with_message_history.invoke(
    [HumanMessage(content="What did Simone de Beauvoir believe about free will")],
    config={"configurable": {"session_id": "baz"}},
)
```
```output

{'output_message': AIMessage(content='西蒙娜·德·波伏娃是一位著名的法国存在主义哲学家，她对自由意志有一些重要观点：

1. 激进的自由：德·波伏娃认为人类拥有激进的自由——通过自己的行为选择和定义自己。她反对决定论，认为我们不仅仅是生物学、成长环境或社会环境的产物。

2. 人类境况的模棱两可：然而，德·波伏娃也意识到人类境况的模棱两可性。虽然我们拥有激进的自由，但我们也是受限于我们的事实性（我们的特定环境和限制）的存在。这在人类经验中造成了张力和痛苦。

3. 责任和虚伪：伴随着这种激进的自由而来的是巨大的责任。德·波伏娃批评了“虚伪”——人们倾向于通过找借口或隐藏在社会角色和规范背后来否认他们的自由和责任。

4. 道德参与：对于德·波伏娃来说，真正的自由和真实性需要与世界和他人进行道德参与。我们必须对自己的选择及其对他人的影响负责。

总的来说，德·波伏娃将自由意志视为人类境况的核心特征，但这一特征充满了困难和模棱两可性。她的哲学强调了拥有自由并用其在道德上塑造我们的生活和世界的重要性。', response_metadata={'id': 'msg_01A78LdxxsCm6uR8vcAdMQBt', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 20, 'output_tokens': 293}}, id='run-9447a229-5d17-4b20-a48b-7507b78b225a-0')}

```
```
```python
import asyncio
import aioredis
async def get_session_history(session_id: str):
    redis = await aioredis.create_redis_pool('redis://localhost')
    history = await redis.lrange(f"history:{session_id}", 0, -1)
    return [json.loads(msg) for msg in history]
async def save_message(session_id: str, message: Dict):
    redis = await aioredis.create_redis_pool('redis://localhost')
    await redis.rpush(f"history:{session_id}", json.dumps(message))
async def clear_history(session_id: str):
    redis = await aioredis.create_redis_pool('redis://localhost')
    await redis.delete(f"history:{session_id}")
```
```output
```

## 持久存储

在许多情况下，持久化对话历史是可取的。`RunnableWithMessageHistory` 对于 `get_session_history` 可调用如何检索其聊天消息历史是中立的。请参见[这里](https://github.com/langchain-ai/langserve/blob/main/examples/chat_with_persistence_and_user/server.py) ，这是一个使用本地文件系统的示例。下面我们演示如何使用 Redis。请查看[内存集成](https://integrations.langchain.com/memory) 页面，以获取使用其他提供程序的聊天消息历史的实现。

### 设置

如果尚未安装 Redis，我们需要安装它：

```python
%pip install --upgrade --quiet redis
```

如果我们没有现有的 Redis 部署可以连接，可以启动本地 Redis Stack 服务器：

```bash
docker run -d -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```
```python
REDIS_URL = "redis://localhost:6379/0"
```

### [LangSmith](https://docs.smith.langchain.com)

LangSmith 在诸如消息历史注入之类的情况下特别有用，否则很难理解链条中各部分的输入是什么。

请注意，LangSmith 不是必需的，但它很有帮助。

如果您想使用 LangSmith，在上面的链接注册后，请确保取消下面的注释并设置您的环境变量以开始记录跟踪：

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

更新消息历史实现只需要我们定义一个新的可调用对象，这次返回一个 `RedisChatMessageHistory` 实例：

```python
from langchain_community.chat_message_histories import RedisChatMessageHistory
def get_message_history(session_id: str) -> RedisChatMessageHistory:
    return RedisChatMessageHistory(session_id, url=REDIS_URL)
with_message_history = RunnableWithMessageHistory(
    runnable,
    get_message_history,
    input_messages_key="input",
    history_messages_key="history",
)
```

我们可以像以前一样调用：

```python
with_message_history.invoke(
    {"ability": "math", "input": "What does cosine mean?"},
    config={"configurable": {"session_id": "foobar"}},
)
```
```output
AIMessage(content='Cosine is a trigonometric function that represents the ratio of the adjacent side to the hypotenuse in a right triangle.')
```
```python
with_message_history.invoke(
    {"ability": "math", "input": "What's its inverse"},
    config={"configurable": {"session_id": "foobar"}},
)
```
```output
AIMessage(content='The inverse of cosine is the arccosine function, denoted as acos or cos^-1, which gives the angle corresponding to a given cosine value.')
```

:::tip

[Langsmith trace](https://smith.langchain.com/public/bd73e122-6ec1-48b2-82df-e6483dc9cb63/r)

:::

查看第二次调用的 Langsmith 追踪，我们可以看到在构建提示时，已注入了一个“history”变量，其中包含两条消息（我们的第一个输入和第一个输出）。

## 下一步

您现在已经学会了一种管理可运行对象消息历史的方法。

要了解更多信息，请参阅本节中有关可运行对象的其他操作指南。

```