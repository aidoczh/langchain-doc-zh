---

sidebar_label: 百川聊天

---

# 与百川-192K聊天

百川聊天模型是由百川智能科技提供的API。更多信息，请参阅 [https://platform.baichuan-ai.com/docs/api](https://platform.baichuan-ai.com/docs/api)

```python
from langchain_community.chat_models import ChatBaichuan
from langchain_core.messages import HumanMessage
```

```python
chat = ChatBaichuan(baichuan_api_key="YOUR_API_KEY")
```

或者，您可以使用以下方式设置您的API密钥：

```python
import os
os.environ["BAICHUAN_API_KEY"] = "YOUR_API_KEY"
```

```python
chat([HumanMessage(content="我日薪8块钱，请问在闰年的二月，我月薪多少")])
```

```output
AIMessage(content='首先，我们需要确定闰年的二月有多少天。闰年的二月有29天。\n\n然后，我们可以计算你的月薪：\n\n日薪 = 月薪 / (当月天数)\n\n所以，你的月薪 = 日薪 * 当月天数\n\n将数值代入公式：\n\n月薪 = 8元/天 * 29天 = 232元\n\n因此，你在闰年的二月的月薪是232元。')
```

## 使用流式处理与百川-192K聊天

```python
chat = ChatBaichuan(
    baichuan_api_key="YOUR_API_KEY",
    streaming=True,
)
```

```python
chat([HumanMessage(content="我日薪8块钱，请问在闰年的二月，我月薪多少")])
```

```output
AIMessageChunk(content='首先，我们需要确定闰年的二月有多少天。闰年的二月有29天。\n\n然后，我们可以计算你的月薪：\n\n日薪 = 月薪 / (当月天数)\n\n所以，你的月薪 = 日薪 * 当月天数\n\n将数值代入公式：\n\n月薪 = 8元/天 * 29天 = 232元\n\n因此，你在闰年的二月的月薪是232元。')
```