---

sidebar_label: Yuan2.0

---

# Yuan2.0

本笔记展示了如何在 LangChain 中使用 [YUAN2 API](https://github.com/IEIT-Yuan/Yuan-2.0/blob/main/docs/inference_server.md)，并结合 langchain.chat_models.ChatYuan2 进行操作。

[*Yuan2.0*](https://github.com/IEIT-Yuan/Yuan-2.0/blob/main/README-EN.md) 是由 IEIT System 开发的新一代基础大型语言模型。我们已经发布了三个模型，分别是 Yuan 2.0-102B、Yuan 2.0-51B 和 Yuan 2.0-2B。我们还为其他开发人员提供了相关的预训练、微调和推理服务脚本。Yuan2.0 基于 Yuan1.0，利用更广泛的高质量预训练数据和指导微调数据集，增强了模型对语义、数学、推理、代码、知识等方面的理解能力。

## 入门指南

### 安装

首先，Yuan2.0 提供了一个兼容 OpenAI 的 API，我们通过使用 OpenAI 客户端将 ChatYuan2 集成到 langchain 聊天模型中。

因此，请确保在您的 Python 环境中安装了 openai 包。运行以下命令：

```python
%pip install --upgrade --quiet openai
```

### 导入所需模块

安装完成后，将必要的模块导入到您的 Python 脚本中：

```python
from langchain_community.chat_models import ChatYuan2
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
```

### 设置 API 服务器

根据 [yuan2 openai api server](https://github.com/IEIT-Yuan/Yuan-2.0/blob/main/docs/Yuan2_fastchat.md) 设置您的兼容 OpenAI 的 API 服务器。

如果您在本地部署了 API 服务器，只需设置 `yuan2_api_key="EMPTY"` 或其他任何您想要的值。

只需确保 `yuan2_api_base` 设置正确。

```python
yuan2_api_key = "your_api_key"
yuan2_api_base = "http://127.0.0.1:8001/v1"
```

### 初始化 ChatYuan2 模型

以下是初始化聊天模型的方法：

```python
chat = ChatYuan2(
    yuan2_api_base="http://127.0.0.1:8001/v1",
    temperature=1.0,
    model_name="yuan2",
    max_retries=3,
    streaming=False,
)
```

### 基本用法

像这样使用系统消息和用户消息调用模型：

```python
messages = [
    SystemMessage(content="你是一个人工智能助手。"),
    HumanMessage(content="你好，你是谁？"),
]
```

```python
print(chat.invoke(messages))
```

### 使用流式传输的基本用法

对于连续交互，使用流式传输功能：

```python
from langchain_core.callbacks import StreamingStdOutCallbackHandler
chat = ChatYuan2(
    yuan2_api_base="http://127.0.0.1:8001/v1",
    temperature=1.0,
    model_name="yuan2",
    max_retries=3,
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()],
)
messages = [
    SystemMessage(content="你是个旅游小助手。"),
    HumanMessage(content="给我介绍一下北京有哪些好玩的。"),
]
```

```python
chat.invoke(messages)
```

## 高级功能

### 使用异步调用

像这样使用非阻塞调用调用模型：

```python
async def basic_agenerate():
    chat = ChatYuan2(
        yuan2_api_base="http://127.0.0.1:8001/v1",
        temperature=1.0,
        model_name="yuan2",
        max_retries=3,
    )
    messages = [
        [
            SystemMessage(content="你是个旅游小助手。"),
            HumanMessage(content="给我介绍一下北京有哪些好玩的。"),
        ]
    ]
    result = await chat.agenerate(messages)
    print(result)
```

```python
import asyncio
asyncio.run(basic_agenerate())
```

### 使用提示模板

像这样使用非阻塞调用并使用聊天模板调用模型：

```python
async def ainvoke_with_prompt_template():
    from langchain_core.prompts.chat import (
        ChatPromptTemplate,
    )
    chat = ChatYuan2(
        yuan2_api_base="http://127.0.0.1:8001/v1",
        temperature=1.0,
        model_name="yuan2",
        max_retries=3,
    )
    prompt = ChatPromptTemplate.from_messages(
        [
            ("system", "你是一个诗人，擅长写诗。"),
            ("human", "给我写首诗，主题是{theme}。"),
        ]
    )
    chain = prompt | chat
    result = await chain.ainvoke({"theme": "明月"})
    print(f"type(result): {type(result)}; {result}")
```

```python
asyncio.run(ainvoke_with_prompt_template())
```

### 使用流式传输的异步调用

对于具有流式输出的非阻塞调用，使用 astream 方法：

```python
async def basic_astream():
    chat = ChatYuan2(
        yuan2_api_base="http://127.0.0.1:8001/v1",
        temperature=1.0,
        model_name="yuan2",
        max_retries=3,
    )
    messages = [
        SystemMessage(content="你是个旅游小助手。"),
        HumanMessage(content="给我介绍一下北京有哪些好玩的。"),
    ]
    result = chat.astream(messages)
    async for chunk in result:
        print(chunk.content, end="", flush=True)
```

```python
import asyncio
asyncio.run(basic_astream())
```