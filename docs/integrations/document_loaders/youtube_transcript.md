# YouTube 字幕

>[YouTube](https://www.youtube.com/) 是由 Google 创建的在线视频分享和社交媒体平台。

这份笔记涵盖了如何从 `YouTube 字幕` 加载文档。

```python
from langchain_community.document_loaders import YoutubeLoader
```

```python
%pip install --upgrade --quiet  youtube-transcript-api
```

```python
loader = YoutubeLoader.from_youtube_url(
    "https://www.youtube.com/watch?v=QsYGlZkevEg", add_video_info=False
)
```

```python
loader.load()
```

### 添加视频信息

```python
%pip install --upgrade --quiet  pytube
```

```python
loader = YoutubeLoader.from_youtube_url(
    "https://www.youtube.com/watch?v=QsYGlZkevEg", add_video_info=True
)
loader.load()
```

### 添加语言偏好

语言参数：这是一个按优先级降序排列的语言代码列表，默认为 `en`。

翻译参数：这是一个翻译偏好，您可以将可用的字幕翻译成您偏好的语言。

```python
loader = YoutubeLoader.from_youtube_url(
    "https://www.youtube.com/watch?v=QsYGlZkevEg",
    add_video_info=True,
    language=["en", "id"],
    translation="en",
)
loader.load()
```

## Google Cloud 的 YouTube 加载器

### 先决条件

1. 创建一个 Google Cloud 项目或使用现有项目

2. 启用 [Youtube Api](https://console.cloud.google.com/apis/enableflow?apiid=youtube.googleapis.com&project=sixth-grammar-344520)

3. [为桌面应用程序授权凭据](https://developers.google.com/drive/api/quickstart/python#authorize_credentials_for_a_desktop_application)

4. `pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib youtube-transcript-api`

### 🧑 摄取 Google 文档数据的说明

默认情况下，`GoogleDriveLoader` 期望 `credentials.json` 文件位于 `~/.credentials/credentials.json`，但可以使用 `credentials_file` 关键字参数进行配置。`token.json` 也是同样的情况。请注意，第一次使用加载器时，`token.json` 将会自动创建。

`GoogleApiYoutubeLoader` 可以从 Google 文档文档 id 列表或文件夹 id 加载。您可以从以下 URL 获取文件夹和文档 id：

请注意，根据您的设置，`service_account_path` 需要进行设置。有关更多详细信息，请参阅[此处](https://developers.google.com/drive/api/v3/quickstart/python)。

```python
# 初始化 GoogleApiClient
from pathlib import Path
from langchain_community.document_loaders import GoogleApiClient, GoogleApiYoutubeLoader
google_api_client = GoogleApiClient(credentials_path=Path("your_path_creds.json"))
# 使用频道
youtube_loader_channel = GoogleApiYoutubeLoader(
    google_api_client=google_api_client,
    channel_name="Reducible",
    captions_language="en",
)
# 使用 Youtube Ids
youtube_loader_ids = GoogleApiYoutubeLoader(
    google_api_client=google_api_client, video_ids=["TrdevFK_am4"], add_video_info=True
)
# 返回文档列表
youtube_loader_channel.load()
```