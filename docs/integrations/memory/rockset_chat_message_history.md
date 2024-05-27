# Rockset

[Rockset](https://rockset.com/product/) 是一个实时分析数据库服务，用于提供规模化的低延迟、高并发分析查询。它在结构化和半结构化数据上构建了一个 Converged Index™，并配备了一个高效的存储向量嵌入。它支持在无模式数据上运行 SQL，使其成为运行带有元数据过滤的向量搜索的理想选择。

本文档介绍了如何使用 [Rockset](https://rockset.com/docs) 存储聊天消息历史记录。

## 设置

```python
%pip install --upgrade --quiet  rockset
```

首先，从 [Rockset控制台](https://console.rockset.com/apikeys) 获取您的 API 密钥。在 Rockset [API 参考](https://rockset.com/docs/rest-api#introduction) 中找到 Rockset 的 API 区域。

## 示例

```python
from langchain_community.chat_message_histories import (
    RocksetChatMessageHistory,
)
from rockset import Regions, RocksetClient
history = RocksetChatMessageHistory(
    session_id="MySession",
    client=RocksetClient(
        api_key="YOUR API KEY",
        host=Regions.usw2a1,  # us-west-2 Oregon
    ),
    collection="langchain_demo",
    sync=True,
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
print(history.messages)
```

输出应该类似于：

```python
[
    HumanMessage(content='hi!', additional_kwargs={'id': '2e62f1c2-e9f7-465e-b551-49bae07fe9f0'}, example=False), 
    AIMessage(content='whats up?', additional_kwargs={'id': 'b9be8eda-4c18-4cf8-81c3-e91e876927d0'}, example=False)
]
```