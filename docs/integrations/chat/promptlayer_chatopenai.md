---

sidebar_label: PromptLayer ChatOpenAI

---

# PromptLayerChatOpenAI

这个示例展示了如何连接到[PromptLayer](https://www.promptlayer.com)，开始记录您的ChatOpenAI请求。

## 安装 PromptLayer

使用 PromptLayer 需要安装 `promptlayer` 包。使用 pip 安装 `promptlayer`。

```python
pip install promptlayer
```

## 导入模块

```python
import os
from langchain_community.chat_models import PromptLayerChatOpenAI
from langchain_core.messages import HumanMessage
```

## 设置环境 API 密钥

您可以在 [www.promptlayer.com](https://www.promptlayer.com) 上点击导航栏中的设置按钮创建一个 PromptLayer API 密钥。

将其设置为名为 `PROMPTLAYER_API_KEY` 的环境变量。

```python
os.environ["PROMPTLAYER_API_KEY"] = "**********"
```

## 使用 PromptLayerOpenAI LLM

*您可以选择传入 `pl_tags` 来使用 PromptLayer 的标记功能跟踪您的请求。*

```python
chat = PromptLayerChatOpenAI(pl_tags=["langchain"])
chat([HumanMessage(content="I am a cat and I want")])
```

```output
AIMessage(content='to take a nap in a cozy spot. I search around for a suitable place and finally settle on a soft cushion on the window sill. I curl up into a ball and close my eyes, relishing the warmth of the sun on my fur. As I drift off to sleep, I can hear the birds chirping outside and feel the gentle breeze blowing through the window. This is the life of a contented cat.', additional_kwargs={})
```

**上述请求现在应该出现在您的[PromptLayer仪表板](https://www.promptlayer.com)上。**

## 使用 PromptLayer Track

如果您想使用任何[PromptLayer跟踪功能](https://magniv.notion.site/Track-4deee1b1f7a34c1680d085f82567dab9)，您需要在实例化 PromptLayer LLM 时传递参数 `return_pl_id` 以获取请求 ID。

```python
import promptlayer
chat = PromptLayerChatOpenAI(return_pl_id=True)
chat_results = chat.generate([[HumanMessage(content="I am a cat and I want")]])
for res in chat_results.generations:
    pl_request_id = res[0].generation_info["pl_request_id"]
    promptlayer.track.score(request_id=pl_request_id, score=100)
```

使用这个功能可以在 PromptLayer 仪表板上跟踪模型的性能。如果您使用的是提示模板，您还可以将模板附加到请求上。总的来说，这使您有机会在 PromptLayer 仪表板上跟踪不同模板和模型的性能。