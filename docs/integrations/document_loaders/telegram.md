# 电报

[Telegram Messenger](https://web.telegram.org/a/) 是一种全球可访问的免费即时通讯服务，支持跨平台、加密、基于云的集中式服务。该应用还提供可选的端到端加密聊天和视频通话、VoIP、文件共享以及其他几项功能。

本笔记介绍了如何将`Telegram`中的数据加载到可被 LangChain 吸收的格式中。

```python
from langchain_community.document_loaders import (
    TelegramChatApiLoader,
    TelegramChatFileLoader,
)
```

```python
loader = TelegramChatFileLoader("example_data/telegram.json")
```

```python
loader.load()
```

```output
[Document(page_content="Henry on 2020-01-01T00:00:02: It's 2020...\n\nHenry on 2020-01-01T00:00:04: Fireworks!\n\nGrace ðŸ§¤ ðŸ\x8d’ on 2020-01-01T00:00:05: You're a minute late!\n\n", metadata={'source': 'example_data/telegram.json'})]
```

`TelegramChatApiLoader` 可直接从 Telegram 中的任何指定聊天加载数据。为了导出数据，您需要对 Telegram 帐户进行身份验证。

您可以从 https://my.telegram.org/auth?to=apps 获取 API_HASH 和 API_ID。

chat_entity – 建议使用[实体](https://docs.telethon.dev/en/stable/concepts/entities.html?highlight=Entity#what-is-an-entity)作为频道的实体。

```python
loader = TelegramChatApiLoader(
    chat_entity="<CHAT_URL>",  # 建议在此处使用实体
    api_hash="<API HASH>",
    api_id="<API_ID>",
    username="",  # 仅在需要缓存会话时才需要。
)
```

```python
loader.load()
```