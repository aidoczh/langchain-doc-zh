# PromptLayer OpenAI

`PromptLayer` 是第一个允许您跟踪、管理和共享您的 GPT 提示工程的平台。`PromptLayer` 充当您的代码与 `OpenAI` 的 Python 库之间的中间件。

`PromptLayer` 记录所有您的 `OpenAI API` 请求，使您能够在 `PromptLayer` 仪表板中搜索和探索请求历史记录。

这个示例展示了如何连接到 [PromptLayer](https://www.promptlayer.com) 来开始记录您的 OpenAI 请求。

另一个示例在[这里](/docs/integrations/providers/promptlayer)。

## 安装 PromptLayer

使用 PromptLayer 需要安装 `promptlayer` 包。使用 pip 安装 `promptlayer`。

```python
%pip install --upgrade --quiet  promptlayer
```

## 导入

```python
import os
import promptlayer
from langchain_community.llms import PromptLayerOpenAI
```

## 设置环境 API 密钥

您可以在 [www.promptlayer.com](https://www.promptlayer.com) 上通过单击导航栏中的设置齿轮创建一个 PromptLayer API 密钥。

将其设置为名为 `PROMPTLAYER_API_KEY` 的环境变量。

您还需要一个名为 `OPENAI_API_KEY` 的 OpenAI 密钥。

```python
from getpass import getpass
PROMPTLAYER_API_KEY = getpass()
```

```output
 ········
```

```python
os.environ["PROMPTLAYER_API_KEY"] = PROMPTLAYER_API_KEY
```

```python
from getpass import getpass
OPENAI_API_KEY = getpass()
```

```output
 ········
```

```python
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

## 像平常一样使用 PromptLayerOpenAI LLM

*您可以选择传入 `pl_tags` 来使用 PromptLayer 的标记功能跟踪您的请求。*

```python
llm = PromptLayerOpenAI(pl_tags=["langchain"])
llm("I am a cat and I want")
```

**上述请求现在应该出现在您的 [PromptLayer 仪表板](https://www.promptlayer.com) 上。**

## 使用 PromptLayer Track

如果您想使用任何 [PromptLayer 跟踪功能](https://magniv.notion.site/Track-4deee1b1f7a34c1680d085f82567dab9)，您需要在实例化 PromptLayer LLM 时传入参数 `return_pl_id` 以获取请求 ID。

```python
llm = PromptLayerOpenAI(return_pl_id=True)
llm_results = llm.generate(["Tell me a joke"])
for res in llm_results.generations:
    pl_request_id = res[0].generation_info["pl_request_id"]
    promptlayer.track.score(request_id=pl_request_id, score=100)
```

使用这个功能可以让您在 PromptLayer 仪表板中跟踪您模型的性能。如果您使用提示模板，也可以将模板附加到请求中。总的来说，这让您有机会在 PromptLayer 仪表板中跟踪不同模板和模型的性能。