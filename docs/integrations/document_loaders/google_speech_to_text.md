# 谷歌语音转文字音频转录

`GoogleSpeechToTextLoader` 允许使用 [Google Cloud 语音转文字 API](https://cloud.google.com/speech-to-text) 对音频文件进行转录，并将转录的文本加载到文档中。

要使用它，您应该已安装了 `google-cloud-speech` Python 包，并且拥有启用了 [语音转文字 API](https://cloud.google.com/speech-to-text/v2/docs/transcribe-client-libraries#before_you_begin) 的 Google Cloud 项目。

- [将大型模型的能力带入谷歌云语音 API](https://cloud.google.com/blog/products/ai-machine-learning/bringing-power-large-models-google-clouds-speech-api)

## 安装与设置

首先，您需要安装 `google-cloud-speech` Python 包。

您可以在 [语音转文字客户端库](https://cloud.google.com/speech-to-text/v2/docs/libraries) 页面找到更多信息。

按照谷歌云文档中的 [快速入门指南](https://cloud.google.com/speech-to-text/v2/docs/sync-recognize) 创建项目并启用 API。

```python
%pip install --upgrade --quiet langchain-google-community[speech]
```

## 示例

`GoogleSpeechToTextLoader` 必须包括 `project_id` 和 `file_path` 参数。音频文件可以指定为谷歌云存储 URI (`gs://...`) 或本地文件路径。

加载器仅支持同步请求，每个音频文件的 [限制为 60 秒或 10MB](https://cloud.google.com/speech-to-text/v2/docs/sync-recognize#:~:text=60%20seconds%20and/or%2010%20MB)。

```python
from langchain_google_community import GoogleSpeechToTextLoader
project_id = "<PROJECT_ID>"
file_path = "gs://cloud-samples-data/speech/audio.flac"
# 或本地文件路径: file_path = "./audio.wav"
loader = GoogleSpeechToTextLoader(project_id=project_id, file_path=file_path)
docs = loader.load()
```

注意：调用 `loader.load()` 会阻塞，直到转录完成。

转录的文本可在 `page_content` 中找到：

```python
docs[0].page_content
```

```
"布鲁克林大桥多大年纪了？"
```

`metadata` 包含了完整的 JSON 响应和更多的元信息：

```python
docs[0].metadata
```

```json
{
  'language_code': 'en-US',
  'result_end_offset': datetime.timedelta(seconds=1)
}
```

## 识别配置

您可以指定 `config` 参数来使用不同的语音识别模型和启用特定功能。

参考 [语音转文字识别器文档](https://cloud.google.com/speech-to-text/v2/docs/recognizers) 和 [`RecognizeRequest`](https://cloud.google.com/python/docs/reference/speech/latest/google.cloud.speech_v2.types.RecognizeRequest) API 参考，了解如何设置自定义配置。

如果不指定 `config`，将自动选择以下选项：

- 模型：[Chirp 通用语音模型](https://cloud.google.com/speech-to-text/v2/docs/chirp-model)

- 语言：`en-US`

- 音频编码：自动检测

- 自动标点：已启用

```python
from google.cloud.speech_v2 import (
    AutoDetectDecodingConfig,
    RecognitionConfig,
    RecognitionFeatures,
)
from langchain_google_community import GoogleSpeechToTextLoader
project_id = "<PROJECT_ID>"
location = "global"
recognizer_id = "<RECOGNIZER_ID>"
file_path = "./audio.wav"
config = RecognitionConfig(
    auto_decoding_config=AutoDetectDecodingConfig(),
    language_codes=["en-US"],
    model="long",
    features=RecognitionFeatures(
        enable_automatic_punctuation=False,
        profanity_filter=True,
        enable_spoken_punctuation=True,
        enable_spoken_emojis=True,
    ),
)
loader = GoogleSpeechToTextLoader(
    project_id=project_id,
    location=location,
    recognizer_id=recognizer_id,
    file_path=file_path,
    config=config,
)
```