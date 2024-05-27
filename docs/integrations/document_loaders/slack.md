# Slack

>[Slack](https://slack.com/) 是一款即时通讯程序。

本文介绍了如何从由 `Slack` 导出生成的 Zip 文件中加载文档。

要获取这个 `Slack` 导出文件，请按照以下说明操作：

## 🧑 获取自己的数据集的说明

导出你的 Slack 数据。你可以通过转到你的 Workspace 管理页面并点击导入/导出选项 ({your_slack_domain}.slack.com/services/export) 来完成这个操作。然后，选择正确的日期范围并点击 `开始导出`。Slack 会在导出准备好时给你发送电子邮件和直接消息。

下载会在你的下载文件夹中产生一个 `.zip` 文件（或者根据你的操作系统配置，可能会在其他位置）。 

复制 `.zip` 文件的路径，并将其分配为下面的 `LOCAL_ZIPFILE`。

```python
from langchain_community.document_loaders import SlackDirectoryLoader
```

```python
# 可选地设置你的 Slack URL。这将为你提供文档来源的正确 URL。
SLACK_WORKSPACE_URL = "https://xxx.slack.com"
LOCAL_ZIPFILE = ""  # 在这里粘贴你的 Slack zip 文件的本地路径。
loader = SlackDirectoryLoader(LOCAL_ZIPFILE, SLACK_WORKSPACE_URL)
```

```python
docs = loader.load()
docs
```