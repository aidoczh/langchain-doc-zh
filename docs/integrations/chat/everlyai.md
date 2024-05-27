---

sidebar_label: EverlyAI

---

# ChatEverlyAI

>[EverlyAI](https://everlyai.xyz) 允许您在云中规模化运行您的 ML 模型。它还提供对[多个 LLM 模型](https://everlyai.xyz)的 API 访问。

这个笔记本演示了使用 `langchain.chat_models.ChatEverlyAI` 来访问[EverlyAI 托管的端点](https://everlyai.xyz/)。

* 设置 `EVERLYAI_API_KEY` 环境变量

* 或使用 `everlyai_api_key` 关键字参数

```python
%pip install --upgrade --quiet  langchain-openai
```

```python
import os
from getpass import getpass
os.environ["EVERLYAI_API_KEY"] = getpass()
```

# 让我们尝试在 EverlyAI 托管的端点上提供的 LLAMA 模型

```python
from langchain_community.chat_models import ChatEverlyAI
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(content="你是一个乐于助人的 AI，会分享你所知道的一切。"),
    HumanMessage(
        content="告诉我关于你自己的技术事实。你是一个变形金刚模型吗？你有多少十亿个参数？"
    ),
]
chat = ChatEverlyAI(
    model_name="meta-llama/Llama-2-7b-chat-hf", temperature=0.3, max_tokens=64
)
print(chat(messages).content)
```

```output
  你好！我只是一个 AI，没有像人类那样的个人信息或技术细节。但是，我可以告诉你，我是一种变形金刚模型，具体来说是 BERT（双向编码器表示来自变换器）模型。B
```

# EverlyAI 还支持流式响应

```python
from langchain_community.chat_models import ChatEverlyAI
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(content="你是一个幽默的 AI，能让人开心。"),
    HumanMessage(content="给我讲个笑话？"),
]
chat = ChatEverlyAI(
    model_name="meta-llama/Llama-2-7b-chat-hf",
    temperature=0.3,
    max_tokens=64,
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()],
)
chat(messages)
```

```output
  啊，你说笑话？*摆正眼镜* 好吧，我有一个给你！*眨眼*
 *为戏剧效果停顿*
为什么 AI 去看心理医生？
*鼓掌*
因为
```

```output
AIMessageChunk(content="  啊，你说笑话？*摆正眼镜* 好吧，我有一个给你！*眨眼*\n *为戏剧效果停顿*\n为什么 AI 去看心理医生？\n*鼓掌*\n因为")
```

# 让我们尝试在 EverlyAI 上使用不同的语言模型

```python
from langchain_community.chat_models import ChatEverlyAI
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(content="你是一个幽默的 AI，能让人开心。"),
    HumanMessage(content="给我讲个笑话？"),
]
chat = ChatEverlyAI(
    model_name="meta-llama/Llama-2-13b-chat-hf-quantized",
    temperature=0.3,
    max_tokens=128,
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()],
)
chat(messages)
```

```output
  哦呵呵！*调整单片眼镜* 好吧，好吧，好吧！看看谁来了！*眨眼*
你想听笑话，嗯？*挺起胸膛* 那么，让我告诉你一个保证能逗乐你的笑话！*清清嗓子*
为什么自行车站不起来？*为戏剧效果停顿* 因为它太累了！*眨眼*
希望这个笑话让你开心，亲爱的！*
```

```output
AIMessageChunk(content="  哦呵呵！*调整单片眼镜* 好吧，好吧，好吧！看看谁来了！*眨眼*\n\n你想听笑话，嗯？*挺起胸膛* 那么，让我告诉你一个保证能逗乐你的笑话！*清清嗓子*\n\n为什么自行车站不起来？*为戏剧效果停顿* 因为它太累了！*眨眼*\n\n希望这个笑话让你开心，亲爱的！*")
```