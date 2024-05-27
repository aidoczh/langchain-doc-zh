# 如何将回调函数附加到模块

:::info 先决条件

本指南假定您熟悉以下概念：

- [回调函数](/docs/concepts/#callbacks)

- [自定义回调处理程序](/docs/how_to/custom_callbacks)

- [链接可运行项](/docs/how_to/sequence)

- [将运行时参数附加到可运行项](/docs/how_to/binding)

:::

如果您正在组合一系列可运行项，并希望在多次执行中重用回调函数，您可以使用 [`.with_config()`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.with_config) 方法附加回调函数。这样可以避免每次调用链时都需要传递回调函数的麻烦。

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
llm = ChatAnthropic(model="claude-3-sonnet-20240229")
prompt = ChatPromptTemplate.from_template("What is 1 + {number}?")
chain = prompt | llm
chain_with_callbacks = chain.with_config(callbacks=callbacks)
chain_with_callbacks.invoke({"number": "2"})
```

```output
Chain RunnableSequence started
Chain ChatPromptTemplate started
Chain ended, outputs: messages=[HumanMessage(content='What is 1 + 2?')]
Chat model started
Chat model ended, response: generations=[[ChatGeneration(text='1 + 2 = 3', message=AIMessage(content='1 + 2 = 3', response_metadata={'id': 'msg_01LjC57hgrmzVhEma4yXdLKF', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}}, id='run-393950f9-79b9-4fd6-ac6e-50d93d75b906-0'))]] llm_output={'id': 'msg_01LjC57hgrmzVhEma4yXdLKF', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}} run=None
Chain ended, outputs: content='1 + 2 = 3' response_metadata={'id': 'msg_01LjC57hgrmzVhEma4yXdLKF', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}} id='run-393950f9-79b9-4fd6-ac6e-50d93d75b906-0'
```

```output
AIMessage(content='1 + 2 = 3', response_metadata={'id': 'msg_01LjC57hgrmzVhEma4yXdLKF', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}}, id='run-393950f9-79b9-4fd6-ac6e-50d93d75b906-0')
```

绑定的回调函数将运行所有嵌套模块运行。

## 下一步

您现在已经学会了如何将回调函数附加到链中。

接下来，请查看本部分的其他操作指南，例如如何[在运行时传递回调函数](/docs/how_to/callbacks_runtime)。