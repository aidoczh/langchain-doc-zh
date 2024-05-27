# GigaChat

本笔记本展示了如何在 [GigaChat](https://developers.sber.ru/portal/products/gigachat) 中使用 LangChain。要使用它，您需要安装 ```gigachat``` Python 包。

```python
%pip install --upgrade --quiet  gigachat
```

要获取 GigaChat 凭据，您需要[创建账户](https://developers.sber.ru/studio/login)并[获取 API 访问权限](https://developers.sber.ru/docs/ru/gigachat/individuals-quickstart)

## 示例

```python
import os
from getpass import getpass
os.environ["GIGACHAT_CREDENTIALS"] = getpass()
```

```python
from langchain_community.chat_models import GigaChat
chat = GigaChat(verify_ssl_certs=False, scope="GIGACHAT_API_PERS")
```

```python
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(
        content="You are a helpful AI that shares everything you know. Talk in English."
    ),
    HumanMessage(content="What is capital of Russia?"),
]
print(chat.invoke(messages).content)
```

```output
俄罗斯的首都是莫斯科。
```