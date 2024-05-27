# Trubrics

[Trubrics](https://trubrics.com) 是一个用户分析平台，可以帮助您收集、分析和管理对 AI 模型的用户提示和反馈。

请查看 [Trubrics 仓库](https://github.com/trubrics/trubrics-sdk) 以获取有关 `Trubrics` 的更多信息。

在本指南中，我们将介绍如何设置 `TrubricsCallbackHandler`。

## 安装和设置

```python
%pip install --upgrade --quiet  trubrics
```

### 获取 Trubrics 凭据

如果您没有 Trubrics 账户，请在[此处](https://trubrics.streamlit.app/)创建一个。在本教程中，我们将使用在创建账户时构建的 `default` 项目。

现在将您的凭据设置为环境变量：

```python
import os
os.environ["TRUBRICS_EMAIL"] = "***@***"
os.environ["TRUBRICS_PASSWORD"] = "***"
```

```python
from langchain_community.callbacks.trubrics_callback import TrubricsCallbackHandler
```

### 使用方法

`TrubricsCallbackHandler` 可以接收各种可选参数。请参阅[此处](https://trubrics.github.io/trubrics-sdk/platform/user_prompts/#saving-prompts-to-trubrics)以了解可以传递给 Trubrics 提示的 kwargs。

```python
class TrubricsCallbackHandler(BaseCallbackHandler):
    """
    Trubrics 的回调处理程序。
    参数:
        project: trubrics 项目，默认项目为 "default"
        email: trubrics 账户电子邮件，也可以在环境变量中设置
        password: trubrics 账户密码，也可以在环境变量中设置
        **kwargs: 所有其他 kwargs 将被解析并设置为 trubrics 提示变量，或添加到 `metadata` 字典中
    """
```

## 示例

以下是如何使用 `TrubricsCallbackHandler` 与 Langchain [LLMs](/docs/how_to#llms) 或 [Chat Models](/docs/how_to#chat-models) 的两个示例。我们将使用 OpenAI 模型，所以在这里设置您的 `OPENAI_API_KEY` 密钥：

```python
os.environ["OPENAI_API_KEY"] = "sk-***"
```

### 1. 使用 LLM

```python
from langchain_openai import OpenAI
```

```python
llm = OpenAI(callbacks=[TrubricsCallbackHandler()])
```

```output
2023-09-26 11:30:02.149 | INFO     | trubrics.platform.auth:get_trubrics_auth_token:61 - 用户 jeff.kayne@trubrics.com 已通过身份验证。
```

```python
res = llm.generate(["Tell me a joke", "Write me a poem"])
```

```output
2023-09-26 11:30:07.760 | INFO     | trubrics.platform:log_prompt:102 - 用户提示已保存到 Trubrics。
2023-09-26 11:30:08.042 | INFO     | trubrics.platform:log_prompt:102 - 用户提示已保存到 Trubrics。
```

```python
print("--> GPT 的笑话: ", res.generations[0][0].text)
print()
print("--> GPT 的诗歌: ", res.generations[1][0].text)
```

```output
--> GPT 的笑话:  
问：鱼撞到墙上时会说什么？
答：坝！
--> GPT 的诗歌:  
一首反思之诗
我站在夜晚，
星星在我上方填满我的视线。
我感到如此深刻的联系，
与世界及其完美之处。
一瞬间的清晰，
空气中的宁静如此宁静。
我的心中充满了平静，
我得到了释放。
过去和现在，
我的思绪创造了一种愉快的情绪。
我的心充满了喜悦，
我的灵魂像玩具一样飞翔。
我反思我的生活，
以及我所做的选择。
我的挣扎和我的痛苦，
我所付出的教训。
未来是一个谜，
但我已经准备好迈出这一步。
我准备好带头，
创造自己的命运。
```

### 2. 使用聊天模型

```python
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_openai import ChatOpenAI
```

```python
chat_llm = ChatOpenAI(
    callbacks=[
        TrubricsCallbackHandler(
            project="default",
            tags=["chat model"],
            user_id="user-id-1234",
            some_metadata={"hello": [1, 2]},
        )
    ]
)
```

```python
chat_res = chat_llm.invoke(
    [
        SystemMessage(content="Every answer of yours must be about OpenAI."),
        HumanMessage(content="Tell me a joke"),
    ]
)
```

```output
2023-09-26 11:30:10.550 | INFO     | trubrics.platform:log_prompt:102 - 用户提示已保存到 Trubrics。
```

```python
print(chat_res.content)
```

```output
为什么 OpenAI 的计算机去参加派对？
因为它想见到它的 AI 朋友们并玩得开心！
```