# 如何创建自定义回调处理程序

:::info 先决条件

本指南假定您熟悉以下概念：

- [回调](/docs/concepts/#callbacks)

:::

LangChain具有一些内置的回调处理程序，但通常您会希望创建具有自定义逻辑的自定义处理程序。

要创建自定义回调处理程序，我们需要确定我们希望处理的[event(s)](https://api.python.langchain.com/en/latest/callbacks/langchain_core.callbacks.base.BaseCallbackHandler.html#langchain-core-callbacks-base-basecallbackhandler)，以及在触发事件时我们希望回调处理程序执行的操作。然后，我们只需将回调处理程序附加到对象上，例如通过[构造函数](/docs/how_to/callbacks_constructor)或[运行时](/docs/how_to/callbacks_runtime)。

在下面的示例中，我们将使用自定义处理程序实现流式处理。

在我们的自定义回调处理程序`MyCustomHandler`中，我们实现了`on_llm_new_token`处理程序，以打印我们刚收到的令牌。然后，我们将自定义处理程序作为构造函数回调附加到模型对象上。

```python
from langchain_anthropic import ChatAnthropic
from langchain_core.callbacks import BaseCallbackHandler
from langchain_core.prompts import ChatPromptTemplate
class MyCustomHandler(BaseCallbackHandler):
    def on_llm_new_token(self, token: str, **kwargs) -> None:
        print(f"My custom handler, token: {token}")
prompt = ChatPromptTemplate.from_messages(["Tell me a joke about {animal}"])
# 为启用流式处理，我们在ChatModel构造函数中传入`streaming=True`
# 另外，我们将自定义处理程序作为回调参数的列表传入
model = ChatAnthropic(
    model="claude-3-sonnet-20240229", streaming=True, callbacks=[MyCustomHandler()]
)
chain = prompt | model
response = chain.invoke({"animal": "bears"})
```

```output
My custom handler, token: Here
My custom handler, token: 's
My custom handler, token:  a
My custom handler, token:  bear
My custom handler, token:  joke
My custom handler, token:  for
My custom handler, token:  you
My custom handler, token: :
My custom handler, token: 
Why
My custom handler, token:  di
My custom handler, token: d the
My custom handler, token:  bear
My custom handler, token:  dissol
My custom handler, token: ve
My custom handler, token:  in
My custom handler, token:  water
My custom handler, token: ?
My custom handler, token: 
Because
My custom handler, token:  it
My custom handler, token:  was
My custom handler, token:  a
My custom handler, token:  polar
My custom handler, token:  bear
My custom handler, token: !
```

您可以查看[此参考页面](https://api.python.langchain.com/en/latest/callbacks/langchain_core.callbacks.base.BaseCallbackHandler.html#langchain-core-callbacks-base-basecallbackhandler)以获取您可以处理的事件列表。请注意，`handle_chain_*`事件适用于大多数LCEL可运行对象。

## 下一步

您现在已经学会了如何创建自定义回调处理程序。

接下来，请查看本部分中的其他操作指南，例如[如何将回调附加到可运行对象](/docs/how_to/callbacks_attach)。