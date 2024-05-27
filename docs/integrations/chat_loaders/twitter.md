# Twitter (通过 Apify)

这个笔记本展示了如何从 Twitter 加载聊天消息进行微调。我们通过利用 Apify 来实现这一点。

首先，使用 Apify 导出推文。一个示例

```python
import json
from langchain_community.adapters.openai import convert_message_to_dict
from langchain_core.messages import AIMessage
```

```python
with open("example_data/dataset_twitter-scraper_2023-08-23_22-13-19-740.json") as f:
    data = json.load(f)
```

```python
# 过滤引用其他推文的推文，因为这有点奇怪
tweets = [d["full_text"] for d in data if "t.co" not in d["full_text"]]
# 将它们创建为 AI 消息
messages = [AIMessage(content=t) for t in tweets]
# 在开头添加一个系统消息
# TODO: 我们可以尝试从推文中提取主题，并将其放入系统消息中。
system_message = {"role": "system", "content": "write a tweet"}
data = [[system_message, convert_message_to_dict(m)] for m in messages]
```