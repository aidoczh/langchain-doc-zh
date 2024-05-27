# Airbyte 问答

这篇笔记展示了如何在结构化数据上进行问答，本例中使用了 `AirbyteStripeLoader`。

向量存储通常很难回答需要计算、分组和过滤结构化数据的问题，因此高层次的想法是使用 `pandas` dataframe 来帮助解决这些类型的问题。

1. 使用 Airbyte 从 Stripe 加载数据。使用 `record_handler` 参数从数据加载器返回一个 JSON。

```python
import os
import pandas as pd
from langchain.agents import AgentType
from langchain_community.document_loaders.airbyte import AirbyteStripeLoader
from langchain_experimental.agents import create_pandas_dataframe_agent
from langchain_openai import ChatOpenAI
stream_name = "customers"
config = {
    "client_secret": os.getenv("STRIPE_CLIENT_SECRET"),
    "account_id": os.getenv("STRIPE_ACCOUNT_D"),
    "start_date": "2023-01-20T00:00:00Z",
}
def handle_record(record: dict, _id: str):
    return record.data
loader = AirbyteStripeLoader(
    config=config,
    record_handler=handle_record,
    stream_name=stream_name,
)
data = loader.load()
```

2. 将数据传递给 `pandas` dataframe。

```python
df = pd.DataFrame(data)
```

3. 将 dataframe `df` 传递给 `create_pandas_dataframe_agent` 并调用。

```python
agent = create_pandas_dataframe_agent(
    ChatOpenAI(temperature=0, model="gpt-4"),
    df,
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTIONS,
)
```

4. 运行 agent。

```python
output = agent.run("有多少行？")
```