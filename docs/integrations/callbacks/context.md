# 背景

[Context](https://context.ai/) 为基于 LLM 技术的产品和功能提供用户分析。

使用 `Context`，您可以在不到 30 分钟内开始了解用户并改善他们的体验。

在本指南中，我们将向您展示如何与 Context 进行集成。

## 安装和设置

```python
%pip install --upgrade --quiet  langchain langchain-openai context-python
```

### 获取 API 凭据

要获取您的 Context API 令牌：

1. 转到您的 Context 帐户的设置页面 (https://with.context.ai/settings)。

2. 生成新的 API 令牌。

3. 将此令牌存储在安全的位置。

### 设置 Context

要使用 `ContextCallbackHandler`，请从 Langchain 中导入处理程序，并使用您的 Context API 令牌对其进行实例化。

在使用处理程序之前，请确保已安装 `context-python` 包。

```python
from langchain_community.callbacks.context_callback import ContextCallbackHandler
```

```python
import os
token = os.environ["CONTEXT_API_TOKEN"]
context_callback = ContextCallbackHandler(token)
```

## 使用

### 在聊天模型中使用 Context 回调

Context 回调处理程序可用于直接记录用户和 AI 助手之间的对话。

```python
import os
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_openai import ChatOpenAI
token = os.environ["CONTEXT_API_TOKEN"]
chat = ChatOpenAI(
    headers={"user_id": "123"}, temperature=0, callbacks=[ContextCallbackHandler(token)]
)
messages = [
    SystemMessage(
        content="You are a helpful assistant that translates English to French."
    ),
    HumanMessage(content="I love programming."),
]
print(chat(messages))
```

### 在 Chains 中使用 Context 回调

Context 回调处理程序还可用于记录链的输入和输出。请注意，链的中间步骤不会被记录 - 只记录起始输入和最终输出。

__注意：__ 确保将相同的上下文对象传递给聊天模型和链。

错误的做法：

> ```python
> chat = ChatOpenAI(temperature=0.9, callbacks=[ContextCallbackHandler(token)])
> chain = LLMChain(llm=chat, prompt=chat_prompt_template, callbacks=[ContextCallbackHandler(token)])
> ```

正确的做法：

>```python
>handler = ContextCallbackHandler(token)
>chat = ChatOpenAI(temperature=0.9, callbacks=[callback])
>chain = LLMChain(llm=chat, prompt=chat_prompt_template, callbacks=[callback])
>```

```python
import os
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
)
from langchain_openai import ChatOpenAI
token = os.environ["CONTEXT_API_TOKEN"]
human_message_prompt = HumanMessagePromptTemplate(
    prompt=PromptTemplate(
        template="What is a good name for a company that makes {product}?",
        input_variables=["product"],
    )
)
chat_prompt_template = ChatPromptTemplate.from_messages([human_message_prompt])
callback = ContextCallbackHandler(token)
chat = ChatOpenAI(temperature=0.9, callbacks=[callback])
chain = LLMChain(llm=chat, prompt=chat_prompt_template, callbacks=[callback])
print(chain.run("colorful socks"))
```