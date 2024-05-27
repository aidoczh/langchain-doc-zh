# Eleven Labs 文本转语音

这篇笔记展示了如何与 `ElevenLabs API` 交互，实现文本转语音的功能。

首先，你需要设置一个 ElevenLabs 账户。你可以按照[这里](https://docs.elevenlabs.io/welcome/introduction)的说明进行操作。

```python
%pip install --upgrade --quiet  elevenlabs
```

```python
import os
os.environ["ELEVEN_API_KEY"] = ""
```

## 使用方法

```python
from langchain_community.tools import ElevenLabsText2SpeechTool
text_to_speak = "Hello world! I am the real slim shady"
tts = ElevenLabsText2SpeechTool()
tts.name
```

```output
'eleven_labs_text2speech'
```

我们可以生成音频，将其保存到临时文件，然后播放它。

```python
speech_file = tts.run(text_to_speak)
tts.play(speech_file)
```

或者直接流式传输音频。

```python
tts.stream_speech(text_to_speak)
```

## 在代理程序中使用

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
```

```python
llm = OpenAI(temperature=0)
tools = load_tools(["eleven_labs_text2speech"])
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```

```python
audio_file = agent.run("Tell me a joke and read it out for me.")
```

```output
> 进入新的 AgentExecutor 链...
动作:
```

{

  "action": "eleven_labs_text2speech",

  "action_input": {

    "query": "Why did the chicken cross the playground? To get to the other slide!"

  }

}

```
观察结果: /tmp/tmpsfg783f1.wav
思考: 我已经准备好音频文件，可以发送给人类了
动作:
```

{

  "action": "Final Answer",

  "action_input": "/tmp/tmpsfg783f1.wav"

}

```
> 链结束。
```

```python
tts.play(audio_file)
```