# IFTTT WebHooks

这篇笔记展示了如何使用 IFTTT Webhooks。

来源：[Connecting IFTTT Services](https://github.com/SidU/teams-langchain-js/wiki/Connecting-IFTTT-Services)。

## 创建 Webhook

- 访问 https://ifttt.com/create

## 配置 "If This"

- 在 IFTTT 界面中点击 "If This" 按钮。

- 在搜索栏中搜索 "Webhooks"。

- 选择第一个选项 "Receive a web request with a JSON payload."。

- 选择一个与你计划连接的服务相关的事件名称。

这将使你更容易管理 webhook URL。

例如，如果你要连接 Spotify，你可以使用 "Spotify" 作为你的事件名称。

- 点击 "Create Trigger" 按钮保存设置并创建 webhook。

## 配置 "Then That"

- 在 IFTTT 界面中点击 "Then That" 按钮。

- 搜索你想要连接的服务，比如 Spotify。

- 从该服务中选择一个动作，比如 "Add track to a playlist"。

- 通过指定必要的细节来配置动作，比如播放列表名称，例如 "Songs from AI"。

- 在你的动作中引用 Webhook 收到的 JSON Payload。对于 Spotify 场景，选择 "{{JsonPayload}}" 作为你的搜索查询。

- 点击 "Create Action" 按钮保存动作设置。

- 配置完动作后，点击 "Finish" 按钮完成设置。

- 恭喜！你已成功将 Webhook 连接到所需的服务，现在可以开始接收数据和触发动作 🎉

## 完成设置

- 要获取你的 webhook URL，请访问 https://ifttt.com/maker_webhooks/settings

- 从那里复制 IFTTT key 值。URL 的格式为 https://maker.ifttt.com/use/YOUR_IFTTT_KEY。获取 YOUR_IFTTT_KEY 值。

```python
from langchain_community.tools.ifttt import IFTTTWebhook
```

```python
import os
key = os.environ["IFTTTKey"]
url = f"https://maker.ifttt.com/trigger/spotify/json/with/key/{key}"
tool = IFTTTWebhook(
    name="Spotify", description="Add a song to spotify playlist", url=url
)
```

```python
tool.run("taylor swift")
```

```output
"恭喜！你已触发了 Spotify 的 JSON 事件"
```