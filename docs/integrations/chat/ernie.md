---

sidebar_label: Ernie Bot 聊天

---

# ErnieBot 聊天

[ERNIE-Bot](https://cloud.baidu.com/doc/WENXINWORKSHOP/s/jlil56u11) 是百度开发的一种大型语言模型，涵盖了大量的中文数据。

本文档介绍了如何开始使用 ErnieBot 聊天模型。

**已弃用警告**

我们建议使用 `langchain_community.chat_models.ErnieBotChat` 的用户改用 `langchain_community.chat_models.QianfanChatEndpoint`。

`QianfanChatEndpoint` 的文档在[这里](/docs/integrations/chat/baidu_qianfan_endpoint/)。

我们推荐用户使用 `QianfanChatEndpoint` 的原因有以下四点：

1. `QianfanChatEndpoint` 支持更多的 Qianfan 平台上的语言模型。

2. `QianfanChatEndpoint` 支持流式传输模式。

3. `QianfanChatEndpoint` 支持函数调用用法。

4. `ErnieBotChat` 缺乏维护并已弃用。

迁移的一些建议：

- 将 `ernie_client_id` 改为 `qianfan_ak`，将 `ernie_client_secret` 改为 `qianfan_sk`。

- 安装 `qianfan` 包，例如 `pip install qianfan`。

- 将 `ErnieBotChat` 改为 `QianfanChatEndpoint`。

```python
from langchain_community.chat_models.baidu_qianfan_endpoint import QianfanChatEndpoint
chat = QianfanChatEndpoint(
    qianfan_ak="你的 qianfan ak",
    qianfan_sk="你的 qianfan sk",
)
```

## 用法

```python
from langchain_community.chat_models import ErnieBotChat
from langchain_core.messages import HumanMessage
chat = ErnieBotChat(
    ernie_client_id="YOUR_CLIENT_ID", ernie_client_secret="YOUR_CLIENT_SECRET"
)
```

或者你可以在环境变量中设置 `client_id` 和 `client_secret`

```bash
export ERNIE_CLIENT_ID=YOUR_CLIENT_ID
export ERNIE_CLIENT_SECRET=YOUR_CLIENT_SECRET
```

```python
chat([HumanMessage(content="你好，你是谁？")])
```

```output
AIMessage(content='你好，我是一个人工智能语言模型。我的目的是帮助用户回答问题或提供信息。我能为您做些什么？', additional_kwargs={}, example=False)
```