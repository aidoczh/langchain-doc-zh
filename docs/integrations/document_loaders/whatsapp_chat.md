# WhatsApp 聊天

[WhatsApp](https://www.whatsapp.com/)（也称为 `WhatsApp Messenger`）是一款免费的跨平台集中式即时通讯（IM）和网络电话（VoIP）服务。它允许用户发送文本和语音消息，进行语音和视频通话，以及分享图片、文档、用户位置和其他内容。

这篇笔记介绍了如何将 `WhatsApp 聊天` 中的数据加载到可以被 LangChain 吸收的格式中。

```python
from langchain_community.document_loaders import WhatsAppChatLoader
```

```python
loader = WhatsAppChatLoader("example_data/whatsapp_chat.txt")
```

```python
loader.load()
```