# 如何将回调函数传递给模块构造函数

:::info 前提条件

本指南假定您熟悉以下概念：

- [回调函数](/docs/concepts/#callbacks)

- [自定义回调处理程序](/docs/how_to/custom_callbacks)

:::

大多数 LangChain 模块允许您直接将 `callbacks` 传递给构造函数。在这种情况下，回调函数将仅对该实例（以及任何嵌套运行）进行调用。

以下是一个示例：

```python
from typing import Any, Dict, List
from langchain_anthropic import ChatAnthropic
from langchain_core.callbacks import BaseCallbackHandler
from langchain_core.messages import BaseMessage
from langchain_core.outputs import LLMResult
from langchain_core.prompts import ChatPromptTemplate
class LoggingHandler(BaseCallbackHandler):
    def on_chat_model_start(
        self, serialized: Dict[str, Any], messages: List[List[BaseMessage]], **kwargs
    ) -> None:
        print("Chat model started")
    def on_llm_end(self, response: LLMResult, **kwargs) -> None:
        print(f"Chat model ended, response: {response}")
    def on_chain_start(
        self, serialized: Dict[str, Any], inputs: Dict[str, Any], **kwargs
    ) -> None:
        print(f"Chain {serialized.get('name')} started")
    def on_chain_end(self, outputs: Dict[str, Any], **kwargs) -> None:
        print(f"Chain ended, outputs: {outputs}")
callbacks = [LoggingHandler()]
llm = ChatAnthropic(model="claude-3-sonnet-20240229", callbacks=callbacks)
prompt = ChatPromptTemplate.from_template("What is 1 + {number}?")
chain = prompt | llm
chain.invoke({"number": "2"})
```

```output
Chat model started
Chat model ended, response: generations=[[ChatGeneration(text='1 + 2 = 3', message=AIMessage(content='1 + 2 = 3', response_metadata={'id': 'msg_01CdKsRmeS9WRb8BWnHDEHm7', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}}, id='run-2d7fdf2a-7405-4e17-97c0-67e6b2a65305-0'))]] llm_output={'id': 'msg_01CdKsRmeS9WRb8BWnHDEHm7', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}} run=None
```

```output
AIMessage(content='1 + 2 = 3', response_metadata={'id': 'msg_01CdKsRmeS9WRb8BWnHDEHm7', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}}, id='run-2d7fdf2a-7405-4e17-97c0-67e6b2a65305-0')
```

您可以看到我们只看到了来自聊天模型运行的事件 - 没有来自提示或更广泛链的链事件。

## 下一步

您现在已经学会了如何将回调函数传递给构造函数。

接下来，请查看本节中的其他操作指南，例如如何[在运行时传递回调函数](/docs/how_to/callbacks_runtime)。