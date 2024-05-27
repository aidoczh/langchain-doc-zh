

# 如何处理工具错误

使用模型调用工具存在一些明显的潜在故障模式。首先，模型需要返回一个可以被解析的输出。其次，模型需要返回有效的工具参数。

我们可以在我们的链中构建错误处理来减轻这些故障模式。

## 设置

我们需要安装以下软件包：

```python
%pip install --upgrade --quiet langchain-core langchain-openai
```

如果您想要在 [LangSmith](https://docs.smith.langchain.com/) 中跟踪您的运行，请取消注释并设置以下环境变量：

```python
import getpass
import os
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 链

假设我们有以下（虚拟）工具和工具调用链。我们将故意使我们的工具复杂以尝试使模型出错。

```python
# 定义工具
from langchain_core.tools import tool
@tool
def complex_tool(int_arg: int, float_arg: float, dict_arg: dict) -> int:
    """使用复杂工具进行复杂操作。"""
    return int_arg * float_arg
```

```python
llm_with_tools = llm.bind_tools(
    [complex_tool],
)
```

```python
# 定义链
chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool
```

我们可以看到，即使我们尝试使用一个相当明确的输入来调用此链，模型仍无法正确调用工具（它忘记了 `dict_arg` 参数）。

```python
chain.invoke(
    "使用复杂工具。参数为 5，2.1，空字典。不要忘记 dict_arg"
)
```

```output
---------------------------------------------------------------------------
``````output
ValidationError                           Traceback (most recent call last)
``````output
Cell In[12], line 1
----> 1 chain.invoke(
      2     "使用复杂工具。参数为 5，2.1，空字典。不要忘记 dict_arg"
      3 )
``````output
File ~/langchain/libs/core/langchain_core/runnables/base.py:2499, in RunnableSequence.invoke(self, input, config)
   2497 try:
   2498     for i, step in enumerate(self.steps):
-> 2499         input = step.invoke(
   2500             input,
   2501             # mark each step as a child run
   2502             patch_config(
   2503                 config, callbacks=run_manager.get_child(f"seq:step:{i+1}")
   2504             ),
   2505         )
   2506 # finish the root run
   2507 except BaseException as e:
``````output
File ~/langchain/libs/core/langchain_core/tools.py:241, in BaseTool.invoke(self, input, config, **kwargs)
    234 def invoke(
    235     self,
    236     input: Union[str, Dict],
    237     config: Optional[RunnableConfig] = None,
    238     **kwargs: Any,
    239 ) -> Any:
    240     config = ensure_config(config)
--> 241     return self.run(
    242         input,
    243         callbacks=config.get("callbacks"),
    244         tags=config.get("tags"),
    245         metadata=config.get("metadata"),
    246         run_name=config.get("run_name"),
    247         run_id=config.pop("run_id", None),
    248         **kwargs,
    249     )
``````output
File ~/langchain/libs/core/langchain_core/tools.py:387, in BaseTool.run(self, tool_input, verbose, start_color, color, callbacks, tags, metadata, run_name, run_id, **kwargs)
    385 except ValidationError as e:
    386     if not self.handle_validation_error:
--> 387         raise e
    388     elif isinstance(self.handle_validation_error, bool):
    389         observation = "Tool input validation error"
``````output
File ~/langchain/libs/core/langchain_core/tools.py:378, in BaseTool.run(self, tool_input, verbose, start_color, color, callbacks, tags, metadata, run_name, run_id, **kwargs)
    364 run_manager = callback_manager.on_tool_start(
    365     {"name": self.name, "description": self.description},
    366     tool_input if isinstance(tool_input, str) else str(tool_input),
   (...)
    375     **kwargs,
    376 )
    377 try:
--> 378     parsed_input = self._parse_input(tool_input)
    379     tool_args, tool_kwargs = self._to_args_and_kwargs(parsed_input)
    380     observation = (
    381         self._run(*tool_args, run_manager=run_manager, **tool_kwargs)
    382         if new_arg_supported
    383         else self._run(*tool_args, **tool_kwargs)
    384     )
``````output
File ~/langchain/libs/core/langchain_core/tools.py:283, in BaseTool._parse_input(self, tool_input)
    281 else:
    282     if input_args is not None:
--> 283         result = input_args.parse_obj(tool_input)
    284         return {
    285             k: getattr(result, k)
    286             for k, v in result.dict().items()
    287             if k in tool_input
    288         }
    289 return tool_input
``````output
File ~/langchain/.venv/lib/python3.9/site-packages/pydantic/v1/main.py:526, in BaseModel.parse_obj(cls, obj)
    524         exc = TypeError(f'{cls.__name__} expected dict not {obj.__class__.__name__}')
    525         raise ValidationError([ErrorWrapper(exc, loc=ROOT_KEY)], cls) from e
--> 526 return cls(**obj)
``````output
File ~/langchain/.venv/lib/python3.9/site-packages/pydantic/v1/main.py:341, in BaseModel.__init__(__pydantic_self__, **data)
    339 values, fields_set, validation_error = validate_model(__pydantic_self__.__class__, data)
    340 if validation_error:
--> 341     raise validation_error
    342 try:
    343     object_setattr(__pydantic_self__, '__dict__', values)
``````output
ValidationError: 1 validation error for complex_toolSchema
dict_arg
  field required (type=value_error.missing)```

## 使用 try/except 处理工具调用

更优雅地处理错误的最简单方法是在工具调用步骤中使用 try/except，并在出现错误时返回有用的信息：

```python
from typing import Any
from langchain_core.runnables import Runnable, RunnableConfig
def try_except_tool(tool_args: dict, config: RunnableConfig) -> Runnable:
    try:
        complex_tool.invoke(tool_args, config=config)
    except Exception as e:
        return f"调用工具时使用以下参数：\n\n{tool_args}\n\n引发了以下错误：\n\n{type(e)}: {e}"
chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | try_except_tool
```

```python
print(
    chain.invoke(
        "使用复杂工具。参数为 5、2.1、空字典。不要忘记 dict_arg"
    )
)
```

```output
调用工具时使用以下参数：
{'int_arg': 5, 'float_arg': 2.1}
引发了以下错误：
<class 'pydantic.v1.error_wrappers.ValidationError'>: 1 validation error for complex_toolSchema
dict_arg
  field required (type=value_error.missing)
```

## 回退

在工具调用出错时，我们还可以尝试回退到一个更好的模型。在这种情况下，我们将回退到一个完全相同的链，只是使用 `gpt-4-1106-preview` 替代了 `gpt-3.5-turbo`。

```python
chain = llm_with_tools | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool
better_model = ChatOpenAI(model="gpt-4-1106-preview", temperature=0).bind_tools(
    [complex_tool], tool_choice="complex_tool"
)
better_chain = better_model | (lambda msg: msg.tool_calls[0]["args"]) | complex_tool
chain_with_fallback = chain.with_fallbacks([better_chain])
chain_with_fallback.invoke(
    "使用复杂工具。参数为 5、2.1、空字典。不要忘记 dict_arg"
)
```

```output
10.5
```

通过查看此链运行的 [Langsmith 追踪](https://smith.langchain.com/public/00e91fc2-e1a4-4b0f-a82e-e6b3119d196c/r)，我们可以看到第一个链调用按预期失败，而回退成功。

## 带异常重试

更进一步，我们可以尝试自动重新运行带有传入的异常的链，以便模型可以纠正其行为：

```python
import json
from typing import Any
from langchain_core.messages import AIMessage, HumanMessage, ToolCall, ToolMessage
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables import RunnablePassthrough
class CustomToolException(Exception):
    """自定义的 LangChain 工具异常。"""
    def __init__(self, tool_call: ToolCall, exception: Exception) -> None:
        super().__init__()
        self.tool_call = tool_call
        self.exception = exception
def tool_custom_exception(msg: AIMessage, config: RunnableConfig) -> Runnable:
    try:
        return complex_tool.invoke(msg.tool_calls[0]["args"], config=config)
    except Exception as e:
        raise CustomToolException(msg.tool_calls[0], e)
def exception_to_messages(inputs: dict) -> dict:
    exception = inputs.pop("exception")
    # 将历史消息添加到原始输入中，以便模型知道它在上一次工具调用中犯了一个错误。
    messages = [
        AIMessage(content="", tool_calls=[exception.tool_call]),
        ToolMessage(
            tool_call_id=exception.tool_call["id"], content=str(exception.exception)
        ),
        HumanMessage(
            content="上一次工具调用引发了异常。请尝试使用更正的参数再次调用工具。不要重复错误。"
        ),
    ]
    inputs["last_output"] = messages
    return inputs
# 我们在提示中添加了一个 last_output MessagesPlaceholder，如果不传入它，不会影响提示，但我们可以选择在重试时插入错误消息。
prompt = ChatPromptTemplate.from_messages(
    [("human", "{input}"), MessagesPlaceholder("last_output", optional=True)]
)
chain = prompt | llm_with_tools | tool_custom_exception
# 如果初始链调用失败，我们使用传入的异常作为消息重新运行它。
self_correcting_chain = chain.with_fallbacks(
    [exception_to_messages | chain], exception_key="exception"
)
```

```python
self_correcting_chain.invoke(
    {
        "input": "使用复杂工具。参数为 5、2.1、空字典。不要忘记 dict_arg"
    }
)
```

```output
10.5
```

我们的链成功了！通过查看此链运行的 [LangSmith 追踪](https://smith.langchain.com/public/c11e804c-e14f-4059-bd09-64766f999c14/r)，我们可以看到我们的初始链仍然失败，只有在重试时链才成功。