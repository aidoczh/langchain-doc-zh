# ChatLiteLLM

[LiteLLM](https://github.com/BerriAI/litellm) 是一个简化调用 Anthropic、Azure、Huggingface、Replicate 等的库。

本笔记涵盖了如何开始使用 Langchain + LiteLLM I/O 库。

```python
from langchain_community.chat_models import ChatLiteLLM
from langchain_core.messages import HumanMessage
```

```python
chat = ChatLiteLLM(model="gpt-3.5-turbo")
```

```python
messages = [
    HumanMessage(
        content="Translate this sentence from English to French. I love programming."
    )
]
chat(messages)
```

```output
AIMessage(content=" J'aime la programmation.", additional_kwargs={}, example=False)
```

## `ChatLiteLLM` 还支持异步和流式功能：

```python
from langchain_core.callbacks import CallbackManager, StreamingStdOutCallbackHandler
```

```python
await chat.agenerate([messages])
```

```output
LLMResult(generations=[[ChatGeneration(text=" J'aime programmer.", generation_info=None, message=AIMessage(content=" J'aime programmer.", additional_kwargs={}, example=False))]], llm_output={}, run=[RunInfo(run_id=UUID('8cc8fb68-1c35-439c-96a0-695036a93652'))])
```

```python
chat = ChatLiteLLM(
    streaming=True,
    verbose=True,
    callback_manager=CallbackManager([StreamingStdOutCallbackHandler()]),
)
chat(messages)
```

```output
 J'aime la programmation.
```

```output
AIMessage(content=" J'aime la programmation.", additional_kwargs={}, example=False)
```