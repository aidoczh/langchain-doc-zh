# Discord

>[Discord](https://discord.com/) 是一款语音通话和即时消息社交平台。用户可以通过语音通话、视频通话、文字消息、媒体和文件在私聊或作为名为“服务器”的社区的一部分进行沟通。服务器是一组持久的聊天室和语音频道，可以通过邀请链接访问。

按照以下步骤下载你的 `Discord` 数据：

1. 进入**用户设置**

2. 然后进入**隐私与安全**

3. 转到**请求我的所有数据**，点击**请求数据**按钮

你可能需要等待30天才能收到你的数据。你会收到一封发送到你在 Discord 注册的邮箱地址的电子邮件。邮件中会有一个下载按钮，你可以使用它下载你的个人 Discord 数据。

```python
import os
import pandas as pd
```

```python
path = input('请输入 Discord "messages" 文件夹内容的路径：')
li = []
for f in os.listdir(path):
    expected_csv_path = os.path.join(path, f, "messages.csv")
    csv_exists = os.path.isfile(expected_csv_path)
    if csv_exists:
        df = pd.read_csv(expected_csv_path, index_col=None, header=0)
        li.append(df)
df = pd.concat(li, axis=0, ignore_index=True, sort=False)
```

```python
from langchain_community.document_loaders.discord import DiscordChatLoader
```

```python
loader = DiscordChatLoader(df, user_id_col="ID")
print(loader.load())
```

[20]