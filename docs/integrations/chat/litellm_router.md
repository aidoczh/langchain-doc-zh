---

sidebar_label: LiteLLM 路由器

---

# ChatLiteLLMRouter

[LiteLLM](https://github.com/BerriAI/litellm) 是一个简化了对 Anthropic、Azure、Huggingface、Replicate 等的调用的库。

本文档介绍了如何开始使用 Langchain + LiteLLM 路由器 I/O 库。

```python
from langchain_community.chat_models import ChatLiteLLMRouter
from langchain_core.messages import HumanMessage
from litellm import Router
```

```python
model_list = [
    {
        "model_name": "gpt-4",
        "litellm_params": {
            "model": "azure/gpt-4-1106-preview",
            "api_key": "<your-api-key>",
            "api_version": "2023-05-15",
            "api_base": "https://<your-endpoint>.openai.azure.com/",
        },
    },
    {
        "model_name": "gpt-4",
        "litellm_params": {
            "model": "azure/gpt-4-1106-preview",
            "api_key": "<your-api-key>",
            "api_version": "2023-05-15",
            "api_base": "https://<your-endpoint>.openai.azure.com/",
        },
    },
]
litellm_router = Router(model_list=model_list)
chat = ChatLiteLLMRouter(router=litellm_router)
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
AIMessage(content="J'aime programmer.")
```

## `ChatLiteLLMRouter` 还支持异步和流式功能：

```python
from langchain_core.callbacks import CallbackManager, StreamingStdOutCallbackHandler
```

```python
await chat.agenerate([messages])
```

```output
LLMResult(generations=[[ChatGeneration(text="J'adore programmer.", generation_info={'finish_reason': 'stop'}, message=AIMessage(content="J'adore programmer."))]], llm_output={'token_usage': {'completion_tokens': 6, 'prompt_tokens': 19, 'total_tokens': 25}, 'model_name': None}, run=[RunInfo(run_id=UUID('75003ec9-1e2b-43b7-a216-10dcc0f75e00'))])
```

```python
chat = ChatLiteLLMRouter(
    router=litellm_router,
    streaming=True,
    verbose=True,
    callback_manager=CallbackManager([StreamingStdOutCallbackHandler()]),
)
chat(messages)
```

```output
J'adore programmer.
```

```output
AIMessage(content="J'adore programmer.")
```