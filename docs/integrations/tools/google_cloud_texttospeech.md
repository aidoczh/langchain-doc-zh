# Google Cloud 文本转语音

[Google Cloud 文本转语音](https://cloud.google.com/text-to-speech) 可以让开发者合成自然音质的语音，提供100多种语音，支持多种语言和变体。它应用了 DeepMind 在 WaveNet 上的突破性研究成果，以及 Google 强大的神经网络，以实现尽可能高的保真度。

本文档展示了如何使用 `Google Cloud 文本转语音 API` 来实现语音合成功能。

首先，您需要设置一个 Google Cloud 项目。您可以按照[这里](https://cloud.google.com/text-to-speech/docs/before-you-begin)的说明进行操作。

```python
%pip install --upgrade --quiet  google-cloud-text-to-speech
```

## 使用方法

```python
from langchain_community.tools import GoogleCloudTextToSpeechTool
text_to_speak = "你好，世界！"
tts = GoogleCloudTextToSpeechTool()
tts.name
```

我们可以生成音频，将其保存到临时文件，然后播放。

```python
speech_file = tts.run(text_to_speak)
```