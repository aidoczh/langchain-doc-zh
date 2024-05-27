# GMail

这个加载器介绍了如何从 GMail 中加载数据。有许多种方式可以从 GMail 中加载数据。这个加载器目前在如何加载数据方面有一定的偏见。它首先查找你发送过的所有消息。然后查找你回复之前邮件的消息。然后获取之前的邮件，并创建该邮件的一个训练样本，然后是你的邮件。

需要注意的是这里存在明显的限制。例如，所有创建的示例只是查看上一个邮件的上下文。

使用方法：

- 设置 Google 开发者账号：转到 Google 开发者控制台，创建一个项目，并为该项目启用 Gmail API。这将为你提供一个后面会用到的 credentials.json 文件。

- 安装 Google 客户端库：运行以下命令来安装 Google 客户端库：

```python
%pip install --upgrade --quiet  google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client
```

```python
import os.path
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
SCOPES = ["https://www.googleapis.com/auth/gmail.readonly"]
creds = None
# token.json 文件存储用户的访问和刷新令牌，在第一次授权流程完成时会自动创建。
if os.path.exists("email_token.json"):
    creds = Credentials.from_authorized_user_file("email_token.json", SCOPES)
# 如果没有（有效的）凭据可用，让用户登录。
if not creds or not creds.valid:
    if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
    else:
        flow = InstalledAppFlow.from_client_secrets_file(
            # 在这里放入你的凭据文件。请按照这里的步骤创建 json 文件 https://cloud.google.com/docs/authentication/getting-started
            "creds.json",
            SCOPES,
        )
        creds = flow.run_local_server(port=0)
    # 保存凭据以便下次运行时使用
    with open("email_token.json", "w") as token:
        token.write(creds.to_json())
```

```python
from langchain_community.chat_loaders.gmail import GMailLoader
```

```python
loader = GMailLoader(creds=creds, n=3)
```

```python
data = loader.load()
```

```python
# 有时可能会出现错误，我们会默默地忽略它们
len(data)
```

```output
2
```

```python
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
)
```

```python
# 这将使 hchase@langchain.com 发送的消息成为 AI 消息
# 这意味着你将训练一个 LLM，使其预测就像是 hchase 在回复一样
training_data = list(
    map_ai_messages(data, sender="Harrison Chase <hchase@langchain.com>")
)
```