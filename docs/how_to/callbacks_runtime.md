# 如何在运行时传递回调函数

:::info 先决条件

本指南假定您熟悉以下概念：

- [回调函数](/docs/concepts/#callbacks)

- [自定义回调处理程序](/docs/how_to/custom_callbacks)

:::

在许多情况下，当运行对象时，传递处理程序而不是回调函数会更有优势。当我们在执行运行时使用 `callbacks` 关键字参数传递 [`CallbackHandlers`](https://api.python.langchain.com/en/latest/callbacks/langchain_core.callbacks.base.BaseCallbackHandler.html#langchain-core-callbacks-base-basecallbackhandler) 时，这些回调函数将由执行中涉及的所有嵌套对象发出。例如，当通过一个处理程序传递给一个代理时，它将用于与代理相关的所有回调以及代理执行中涉及的所有对象，即工具和LLM。

这样可以避免我们手动将处理程序附加到每个单独的嵌套对象上。以下是一个示例：

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
chain.invoke({"number": "2"}, config={"callbacks": callbacks})
```

```output
Chain RunnableSequence started
Chain ChatPromptTemplate started
Chain ended, outputs: messages=[HumanMessage(content='What is 1 + 2?')]
Chat model started
Chat model ended, response: generations=[[ChatGeneration(text='1 + 2 = 3', message=AIMessage(content='1 + 2 = 3', response_metadata={'id': 'msg_01D8Tt5FdtBk5gLTfBPm2tac', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}}, id='run-bb0dddd8-85f3-4e6b-8553-eaa79f859ef8-0'))]] llm_output={'id': 'msg_01D8Tt5FdtBk5gLTfBPm2tac', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}} run=None
Chain ended, outputs: content='1 + 2 = 3' response_metadata={'id': 'msg_01D8Tt5FdtBk5gLTfBPm2tac', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}} id='run-bb0dddd8-85f3-4e6b-8553-eaa79f859ef8-0'
```

```output
AIMessage(content='1 + 2 = 3', response_metadata={'id': 'msg_01D8Tt5FdtBk5gLTfBPm2tac', 'model': 'claude-3-sonnet-20240229', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 16, 'output_tokens': 13}}, id='run-bb0dddd8-85f3-4e6b-8553-eaa79f859ef8-0')
```

如果模块已经关联了现有的回调函数，这些函数将会在运行时传递的回调函数之外运行。

## 下一步

您已经学会了如何在运行时传递回调函数。

接下来，查看本部分中的其他指南，例如如何[将回调函数传递给模块构造函数](/docs/how_to/custom_callbacks)。	