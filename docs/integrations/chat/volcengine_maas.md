---

sidebar_label: Volc Engine Maas

---

# VolcEngineMaasChat

这篇笔记本为您提供了如何开始使用 Volc Engine Maas 聊天模型的指南。

```python
# 安装包
%pip install --upgrade --quiet  volcengine
```

```python
from langchain_community.chat_models import VolcEngineMaasChat
from langchain_core.messages import HumanMessage
```

```python
chat = VolcEngineMaasChat(volc_engine_maas_ak="your ak", volc_engine_maas_sk="your sk")
```

或者您可以在环境变量中设置 access_key 和 secret_key

```bash
export VOLC_ACCESSKEY=YOUR_AK
export VOLC_SECRETKEY=YOUR_SK
```

```python
chat([HumanMessage(content="给我讲个笑话")])
```

```output
AIMessage(content='好的，这是一个笑话：\n\n为什么鸟儿不会玩电脑游戏？\n\n因为它们没有翅膀！')
```

# volc engine maas chat with stream

```python
chat = VolcEngineMaasChat(
    volc_engine_maas_ak="your ak",
    volc_engine_maas_sk="your sk",
    streaming=True,
)
```

```python
chat([HumanMessage(content="给我讲个笑话")])
```

```output
AIMessage(content='好的，这是一个笑话：\n\n三岁的女儿说她会造句了，妈妈让她用“年轻”造句，女儿说：“妈妈减肥，一年轻了好几斤”。')
```