# AssemblyAI 音频转录

`AssemblyAIAudioTranscriptLoader` 允许使用 [AssemblyAI API](https://www.assemblyai.com) 对音频文件进行转录，并将转录的文本加载到文档中。

要使用它，您应该已安装了 `assemblyai` python 包，并且设置了环境变量 `ASSEMBLYAI_API_KEY` 为您的 API 密钥。另外，API 密钥也可以作为参数传递。

有关 AssemblyAI 的更多信息：

- [网站](https://www.assemblyai.com/)

- [获取免费 API 密钥](https://www.assemblyai.com/dashboard/signup)

- [AssemblyAI API 文档](https://www.assemblyai.com/docs)

## 安装

首先，您需要安装 `assemblyai` python 包。

您可以在 [assemblyai-python-sdk GitHub 仓库](https://github.com/AssemblyAI/assemblyai-python-sdk) 中找到更多信息。

```python
%pip install --upgrade --quiet  assemblyai
```

## 示例

`AssemblyAIAudioTranscriptLoader` 至少需要 `file_path` 参数。音频文件可以指定为 URL 或本地文件路径。

```python
from langchain_community.document_loaders import AssemblyAIAudioTranscriptLoader
audio_file = "https://storage.googleapis.com/aai-docs-samples/nbc.mp3"
# 或本地文件路径：audio_file = "./nbc.mp3"
loader = AssemblyAIAudioTranscriptLoader(file_path=audio_file)
docs = loader.load()
```

注意：调用 `loader.load()` 会阻塞，直到转录完成。

转录的文本可在 `page_content` 中找到：

```python
docs[0].page_content
```

```
"Load time, a new president and new congressional makeup. Same old ..."
```

`metadata` 包含了更多的元信息，以完整的 JSON 响应形式返回：

```python
docs[0].metadata
```

```
{'language_code': <LanguageCode.en_us: 'en_us'>,
 'audio_url': 'https://storage.googleapis.com/aai-docs-samples/nbc.mp3',
 'punctuate': True,
 'format_text': True,
  ...
}
```

## 转录格式

您可以指定 `transcript_format` 参数来选择不同的格式。

根据格式的不同，会返回一个或多个文档。以下是不同的 `TranscriptFormat` 选项：

- `TEXT`：包含转录文本的一个文档

- `SENTENCES`：多个文档，将转录按句子拆分

- `PARAGRAPHS`：多个文档，将转录按段落拆分

- `SUBTITLES_SRT`：以 SRT 字幕格式导出的转录文本的一个文档

- `SUBTITLES_VTT`：以 VTT 字幕格式导出的转录文本的一个文档

```python
from langchain_community.document_loaders.assemblyai import TranscriptFormat
loader = AssemblyAIAudioTranscriptLoader(
    file_path="./your_file.mp3",
    transcript_format=TranscriptFormat.SENTENCES,
)
docs = loader.load()
```

## 转录配置

您还可以指定 `config` 参数来使用不同的音频智能模型。

访问 [AssemblyAI API 文档](https://www.assemblyai.com/docs) 以获取所有可用模型的概述！

```python
import assemblyai as aai
config = aai.TranscriptionConfig(
    speaker_labels=True, auto_chapters=True, entity_detection=True
)
loader = AssemblyAIAudioTranscriptLoader(file_path="./your_file.mp3", config=config)
```

## 作为参数传递 API 密钥

除了将 API 密钥设置为环境变量 `ASSEMBLYAI_API_KEY` 外，还可以将其作为参数传递。

```python
loader = AssemblyAIAudioTranscriptLoader(
    file_path="./your_file.mp3", api_key="YOUR_KEY"
)
```