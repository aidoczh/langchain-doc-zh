# 在异步环境中如何使用回调函数

:::info 前提条件

本指南假设您熟悉以下概念：

- [回调函数](/docs/concepts/#callbacks)

- [自定义回调处理程序](/docs/how_to/custom_callbacks)

:::

如果您计划使用异步 API，建议使用并扩展 [`AsyncCallbackHandler`](https://api.python.langchain.com/en/latest/callbacks/langchain_core.callbacks.base.AsyncCallbackHandler.html) 来避免阻塞运行循环。

**注意**：如果您在运行 LLM / Chain / Tool / Agent 时同时使用同步的 `CallbackHandler` 和异步的方法，它仍然可以工作。然而，在底层，它将使用 [`run_in_executor`](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.run_in_executor) 调用，如果您的 `CallbackHandler` 不是线程安全的，可能会导致问题。

```python
import asyncio
from typing import Any, Dict, List
from langchain_anthropic import ChatAnthropic
from langchain_core.callbacks import AsyncCallbackHandler, BaseCallbackHandler
from langchain_core.messages import HumanMessage
from langchain_core.outputs import LLMResult
class MyCustomSyncHandler(BaseCallbackHandler):
    def on_llm_new_token(self, token: str, **kwargs) -> None:
        print(f"在 `thread_pool_executor` 中调用同步处理程序：token: {token}")
class MyCustomAsyncHandler(AsyncCallbackHandler):
    """用于处理来自 langchain 的回调的异步回调处理程序。"""
    async def on_llm_start(
        self, serialized: Dict[str, Any], prompts: List[str], **kwargs: Any
    ) -> None:
        """在链开始运行时运行。"""
        print("zzzz....")
        await asyncio.sleep(0.3)
        class_name = serialized["name"]
        print("嗨！我刚刚醒来。您的 llm 正在启动")
    async def on_llm_end(self, response: LLMResult, **kwargs: Any) -> None:
        """在链结束运行时运行。"""
        print("zzzz....")
        await asyncio.sleep(0.3)
        print("嗨！我刚刚醒来。您的 llm 正在结束")
# 要启用流式传输，我们将 `streaming=True` 传递给 ChatModel 构造函数
# 此外，我们传递一个包含我们自定义处理程序的列表
chat = ChatAnthropic(
    model="claude-3-sonnet-20240229",
    max_tokens=25,
    streaming=True,
    callbacks=[MyCustomSyncHandler(), MyCustomAsyncHandler()],
)
await chat.agenerate([[HumanMessage(content="给我讲个笑话")]])
```

```output
zzzz....
嗨！我刚刚醒来。您的 llm 正在启动
在 `thread_pool_executor` 中调用同步处理程序：token: Here
在 `thread_pool_executor` 中调用同步处理程序：token: 's
在 `thread_pool_executor` 中调用同步处理程序：token:  a
在 `thread_pool_executor` 中调用同步处理程序：token:  little
在 `thread_pool_executor` 中调用同步处理程序：token:  joke
在 `thread_pool_executor` 中调用同步处理程序：token:  for
在 `thread_pool_executor` 中调用同步处理程序：token:  you
在 `thread_pool_executor` 中调用同步处理程序：token: :
在 `thread_pool_executor` 中调用同步处理程序：token: 
为什么
在 `thread_pool_executor` 中调用同步处理程序：token:  不能
在 `thread_pool_executor` 中调用同步处理程序：token:  一个
在 `thread_pool_executor` 中调用同步处理程序：token:  自行车
在 `thread_pool_executor` 中调用同步处理程序：token:  自己
在 `thread_pool_executor` 中调用同步处理程序：token:  站起来
在 `thread_pool_executor` 中调用同步处理程序：token:  ？
在 `thread_pool_executor` 中调用同步处理程序：token:  因为
在 `thread_pool_executor` 中调用同步处理程序：token:  它
在 `thread_pool_executor` 中调用同步处理程序：token:  是
在 `thread_pool_executor` 中调用同步处理程序：token:  两
在 `thread_pool_executor` 中调用同步处理程序：token:  -
在 `thread_pool_executor` 中调用同步处理程序：token:  轮
zzzz....
嗨！我刚刚醒来。您的 llm 正在结束
```

```output
LLMResult(generations=[[ChatGeneration(text="这是一个小笑话：\n\n为什么一个自行车不能自己站起来？因为它是两轮", message=AIMessage(content="这是一个小笑话：\n\n为什么一个自行车不能自己站起来？因为它是两轮", id='run-8afc89e8-02c0-4522-8480-d96977240bd4-0'))]], llm_output={}, run=[RunInfo(run_id=UUID('8afc89e8-02c0-4522-8480-d96977240bd4'))])
```

## 下一步

您已经学会了如何创建自己的自定义回调处理程序。

接下来，请查看本节中的其他指南，例如[如何将回调附加到可运行对象](/docs/how_to/callbacks_attach)。