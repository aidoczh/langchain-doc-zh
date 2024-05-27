# 如何缓存聊天模型的响应

:::info 先决条件

本指南假定您熟悉以下概念：

- [聊天模型](/docs/concepts/#chat-models)

- [LLMs](/docs/concepts/#llms)

:::

LangChain为聊天模型提供了一个可选的缓存层。这有两个主要优点：

- 如果您经常多次请求相同的完成结果，它可以通过减少您向LLM提供程序发出的API调用次数来节省您的费用。这在应用程序开发过程中尤为有用。

- 通过减少您向LLM提供程序发出的API调用次数，可以加快应用程序的运行速度。

本指南将指导您如何在您的应用程序中启用此功能。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

```python
# <!-- ruff: noqa: F821 -->
from langchain.globals import set_llm_cache
```

## 内存缓存

这是一个临时缓存，用于在内存中存储模型调用。当您的环境重新启动时，它将被清除，并且不会跨进程共享。

```python
%%time
from langchain.cache import InMemoryCache
set_llm_cache(InMemoryCache())
# 第一次调用时，由于尚未缓存，所以需要更长的时间
llm.invoke("Tell me a joke")
```

```output
CPU times: user 645 ms, sys: 214 ms, total: 859 ms
Wall time: 829 ms
```

```output
AIMessage(content="为什么科学家不相信原子？\n\n因为它们构成了一切！", response_metadata={'token_usage': {'completion_tokens': 13, 'prompt_tokens': 11, 'total_tokens': 24}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-b6836bdd-8c30-436b-828f-0ac5fc9ab50e-0')
```

```python
%%time
# 第二次调用时，由于已经缓存，所以速度更快
llm.invoke("Tell me a joke")
```

```output
CPU times: user 822 µs, sys: 288 µs, total: 1.11 ms
Wall time: 1.06 ms
```

```output
AIMessage(content="为什么科学家不相信原子？\n\n因为它们构成了一切！", response_metadata={'token_usage': {'completion_tokens': 13, 'prompt_tokens': 11, 'total_tokens': 24}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-b6836bdd-8c30-436b-828f-0ac5fc9ab50e-0')
```

## SQLite缓存

此缓存实现使用`SQLite`数据库存储响应，并且在进程重新启动时仍然存在。

```python
!rm .langchain.db
```

```python
# 我们可以使用SQLite缓存做同样的事情
from langchain_community.cache import SQLiteCache
set_llm_cache(SQLiteCache(database_path=".langchain.db"))
```

```python
%%time
# 第一次调用时，由于尚未缓存，所以需要更长的时间
llm.invoke("Tell me a joke")
```

```output
CPU times: user 9.91 ms, sys: 7.68 ms, total: 17.6 ms
Wall time: 657 ms
```

```output
AIMessage(content='为什么稻草人获奖？因为他在他的领域表现出色！', response_metadata={'token_usage': {'completion_tokens': 17, 'prompt_tokens': 11, 'total_tokens': 28}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_c2295e73ad', 'finish_reason': 'stop', 'logprobs': None}, id='run-39d9e1e8-7766-4970-b1d8-f50213fd94c5-0')
```

```python
%%time
# 第二次调用时，由于已经缓存，所以速度更快
llm.invoke("Tell me a joke")
```

```output
CPU times: user 52.2 ms, sys: 60.5 ms, total: 113 ms
Wall time: 127 ms
```

```output
AIMessage(content='为什么稻草人获奖？因为他在他的领域表现出色！', id='run-39d9e1e8-7766-4970-b1d8-f50213fd94c5-0')
```

## 下一步

您现在已经学会了如何缓存模型的响应以节省时间和金钱。

接下来，请查看本部分中有关聊天模型的其他操作指南，比如[如何让模型返回结构化输出](/docs/how_to/structured_output)或[如何创建自己的自定义聊天模型](/docs/how_to/custom_chat_model)。