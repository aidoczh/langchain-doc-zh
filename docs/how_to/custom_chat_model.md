# 如何创建自定义聊天模型类
:::info 先决条件
本指南假定您熟悉以下概念：
- [聊天模型](/docs/concepts/#chat-models)
:::
在本指南中，我们将学习如何使用 LangChain 抽象创建自定义聊天模型。
通过将您的 LLM 与标准的 [`BaseChatModel`](https://api.python.langchain.com/en/latest/language_models/langchain_core.language_models.chat_models.BaseChatModel.html) 接口包装，您可以在现有的 LangChain 程序中使用您的 LLM，并且只需进行最少的代码修改！
作为额外福利，您的 LLM 将自动成为 LangChain 的 `Runnable`，并将从一些优化中受益（例如，通过线程池进行批处理），异步支持，`astream_events` API 等。
## 输入和输出
首先，我们需要讨论**消息**，这些是聊天模型的输入和输出。
### 消息
聊天模型将消息作为输入，并返回消息作为输出。
LangChain 有一些[内置消息类型](/docs/concepts/#message-types)：
| 消息类型              | 描述                                                                                           |
|-----------------------|-----------------------------------------------------------------------------------------------|
| `SystemMessage`       | 用于启动 AI 行为，通常作为输入消息序列的第一个消息传递。                                       |
| `HumanMessage`        | 表示与聊天模型交互的人的消息。                                                                 |
| `AIMessage`           | 表示来自聊天模型的消息。这可以是文本，也可以是请求调用工具。                                  |
| `FunctionMessage` / `ToolMessage` | 用于将工具调用的结果传递回模型的消息。                                       |
| `AIMessageChunk` / `HumanMessageChunk` / ... | 每种消息类型的分块变体。 |
::: {.callout-note}
`ToolMessage` 和 `FunctionMessage` 紧随 OpenAI 的 `function` 和 `tool` 角色。
这是一个快速发展的领域，随着更多模型添加函数调用功能，预计将会对此模式进行补充。
:::
```python
from langchain_core.messages import (
    AIMessage,
    BaseMessage,
    FunctionMessage,
    HumanMessage,
    SystemMessage,
    ToolMessage,
)
```
### 流式变体
所有聊天消息都有一个包含 `Chunk` 的流式变体。
```python
from langchain_core.messages import (
    AIMessageChunk,
    FunctionMessageChunk,
    HumanMessageChunk,
    SystemMessageChunk,
    ToolMessageChunk,
)
```
这些分块在从聊天模型中输出流时使用，并且它们都定义了一个可添加的属性！
```python
AIMessageChunk(content="Hello") + AIMessageChunk(content=" World!")
```
```output
AIMessageChunk(content='Hello World!')
```
## 基础聊天模型
让我们实现一个聊天模型，它会回显提示中最后一条消息的前 `n` 个字符！
为此，我们将继承自 `BaseChatModel`，并且我们需要实现以下内容：
| 方法/属性                    | 描述                                                       | 必需/可选  |
|------------------------------------|-------------------------------------------------------------------|--------------------|
| `_generate`                        | 用于从提示生成聊天结果                       | 必需           |
| `_llm_type` (属性)             | 用于唯一标识模型的类型。用于日志记录。| 必需           |
| `_identifying_params` (属性)   | 用于跟踪目的表示模型参数化。            | 可选           |
| `_stream`                          | 用于实现流式处理。                                       | 可选           |
| `_agenerate`                       | 用于实现本机异步方法。                           | 可选           |
| `_astream`                         | 用于实现 `_stream` 的异步版本。                      | 可选           |
:::tip
`_astream` 实现使用 `run_in_executor` 在单独的线程中启动同步 `_stream`，如果实现了 `_stream`，否则将回退到使用 `_agenerate`。
如果您想要重用 `_stream` 实现，可以使用这个技巧，但如果能够实现原生异步代码，那将是更好的解决方案，因为该代码将以更少的开销运行。
:::
### 实现
```python
from typing import Any, AsyncIterator, Dict, Iterator, List, Optional
from langchain_core.callbacks import (
    AsyncCallbackManagerForLLMRun,
    CallbackManagerForLLMRun,
)
from langchain_core.language_models import BaseChatModel, SimpleChatModel
from langchain_core.messages import AIMessageChunk, BaseMessage, HumanMessage
from langchain_core.outputs import ChatGeneration, ChatGenerationChunk, ChatResult
from langchain_core.runnables import run_in_executor
class CustomChatModelAdvanced(BaseChatModel):
    """一个自定义聊天模型，回显提示中最后一条消息的前 `n` 个字符。
    在为 LangChain 贡献实现时，仔细记录
    包括模型的初始化参数，包括
    如何初始化模型的示例，并包括任何相关的
    链接到底层模型文档或 API。
    示例：
        .. code-block:: python
            model = CustomChatModel(n=2)
            result = model.invoke([HumanMessage(content="hello")])
            result = model.batch([[HumanMessage(content="hello")],
                                 [HumanMessage(content="world")]])
    """
    model_name: str
    """模型的名称"""
    n: int
    """要回显的提示中最后一条消息的字符数。"""
    def _generate(
        self,
        messages: List[BaseMessage],
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> ChatResult:
        """覆盖 _generate 方法以实现聊天模型逻辑。
        这可以是对 API 的调用，对本地模型的调用，或者生成对输入提示的响应的任何其他
        实现。
        Args:
            messages: 由消息列表组成的提示。
            stop: 模型应停止生成的字符串列表。
                  如果生成由于停止令牌而停止，则停止令牌本身
                  应包含在输出中。目前尚未强制执行
                  跨模型，但遵循这一实践是很好的，因为
                  这样可以更轻松地解析模型的输出
                  下游，并了解生成停止的原因。
            run_manager: 具有 LLM 回调的运行管理器。
        """
        # 用实际逻辑替换此处以从消息列表生成响应。
        last_message = messages[-1]
        tokens = last_message.content[: self.n]
        message = AIMessage(
            content=tokens,
            additional_kwargs={},  # 用于添加附加负载（例如，函数调用请求）
            response_metadata={  # 用于响应元数据
                "time_in_seconds": 3,
            },
        )
        ##
        generation = ChatGeneration(message=message)
        return ChatResult(generations=[generation])
    def _stream(
        self,
        messages: List[BaseMessage],
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> Iterator[ChatGenerationChunk]:
        """流式处理模型的输出。
        如果模型支持流式处理，则应实现此方法。
        如果模型不支持流式处理，
        则不要实现它。在这种情况下，流式处理请求将自动处理
        由 _generate 方法。
        Args:
            messages: 由消息列表组成的提示。
            stop: 模型应停止生成的字符串列表。
                  如果生成由于停止令牌而停止，则停止令牌本身
                  应包含在输出中。目前尚未强制执行
                  跨模型，但遵循这一实践是很好的，因为
                  这样可以更轻松地解析模型的输出
                  下游，并了解生成停止的原因。
            run_manager: 具有 LLM 回调的运行管理器。
        """
        last_message = messages[-1]
        tokens = last_message.content[: self.n]
        for token in tokens:
            chunk = ChatGenerationChunk(message=AIMessageChunk(content=token))
            if run_manager:
                # 在较新版本的 LangChain 中，这是可选的
                # on_llm_new_token 将自动调用
                run_manager.on_llm_new_token(token, chunk=chunk)
            yield chunk
        # 让我们添加一些其他信息（例如，响应元数据）
        chunk = ChatGenerationChunk(
            message=AIMessageChunk(content="", response_metadata={"time_in_sec": 3})
        )
        if run_manager:
            # 在较新版本的 LangChain 中，这是可选的
            # on_llm_new_token 将自动调用
            run_manager.on_llm_new_token(token, chunk=chunk)
        yield chunk
    @property
    def _llm_type(self) -> str:
        """获取此聊天模型使用的语言模型类型。"""
        return "echoing-chat-model-advanced"
    @property
    def _identifying_params(self) -> Dict[str, Any]:
        """返回一个标识参数的字典。
        此信息由 LangChain 回调系统使用，该系统
        用于跟踪目的，使其能够监视 LLM。
        """
        return {
            # 模型名称允许用户在 LLM 监控应用程序中指定自定义令牌计数规则
            # （例如，在 LangSmith 中，用户可以为其模型提供每个令牌的定价，并监视
            # 给定 LLM 的成本。）
            "model_name": self.model_name,
        }
```
### 让我们来测试一下 🧪
聊天模型将实现 LangChain 的标准 `Runnable` 接口，许多 LangChain 抽象支持这一接口！
```python
model = CustomChatModelAdvanced(n=3, model_name="my_custom_model")
model.invoke(
    [
        HumanMessage(content="hello!"),
        AIMessage(content="Hi there human!"),
        HumanMessage(content="Meow!"),
    ]
)
```
```output
AIMessage(content='Meo', response_metadata={'time_in_seconds': 3}, id='run-ddb42bd6-4fdd-4bd2-8be5-e11b67d3ac29-0')
```
```python
model.invoke("hello")
```
```output
AIMessage(content='hel', response_metadata={'time_in_seconds': 3}, id='run-4d3cc912-44aa-454b-977b-ca02be06c12e-0')
```
```python
model.batch(["hello", "goodbye"])
```
```output
[AIMessage(content='hel', response_metadata={'time_in_seconds': 3}, id='run-9620e228-1912-4582-8aa1-176813afec49-0'),
 AIMessage(content='goo', response_metadata={'time_in_seconds': 3}, id='run-1ce8cdf8-6f75-448e-82f7-1bb4a121df93-0')]
```
```python
for chunk in model.stream("cat"):
    print(chunk.content, end="|")
```
```output
c|a|t||
```
请查看模型中 `_astream` 的实现！如果不实现它，将不会有输出流。
```python
async for chunk in model.astream("cat"):
    print(chunk.content, end="|")
```
```output
c|a|t||
```
让我们尝试使用 astream 事件 API，这也将有助于双重检查所有回调是否已实现！
```python
async for event in model.astream_events("cat", version="v1"):
    print(event)
```
```output
{'event': 'on_chat_model_start', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'name': 'CustomChatModelAdvanced', 'tags': [], 'metadata': {}, 'data': {'input': 'cat'}}
{'event': 'on_chat_model_stream', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'name': 'CustomChatModelAdvanced', 'data': {'chunk': AIMessageChunk(content='c', id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
{'event': 'on_chat_model_stream', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'name': 'CustomChatModelAdvanced', 'data': {'chunk': AIMessageChunk(content='a', id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
{'event': 'on_chat_model_stream', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'name': 'CustomChatModelAdvanced', 'data': {'chunk': AIMessageChunk(content='t', id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
{'event': 'on_chat_model_stream', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'name': 'CustomChatModelAdvanced', 'data': {'chunk': AIMessageChunk(content='', response_metadata={'time_in_sec': 3}, id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
{'event': 'on_chat_model_end', 'name': 'CustomChatModelAdvanced', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'data': {'output': AIMessageChunk(content='cat', response_metadata={'time_in_sec': 3}, id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
``````output
/home/eugene/src/langchain/libs/core/langchain_core/_api/beta_decorator.py:87: LangChainBetaWarning: This API is in beta and may change in the future.
  warn_beta(
```
## 贡献
我们感谢所有聊天模型集成的贡献。
以下是一个检查列表，可帮助确保您的贡献被添加到 LangChain 中：
文档：
* 模型包含所有初始化参数的文档字符串，因为这些将显示在 [APIReference](https://api.python.langchain.com/en/stable/langchain_api_reference.html) 中。
* 如果模型由服务提供支持，则模型的类文档字符串包含指向模型 API 的链接。
测试：
* [ ] 为重写的方法添加单元测试或集成测试。如果您已重写相应的代码，请验证 `invoke`、`ainvoke`、`batch`、`stream` 是否正常工作。
流式处理（如果您正在实现）：
* [ ] 实现 _stream 方法以使流式处理正常工作
停止标记行为：
* [ ] 应遵守停止标记
* [ ] 停止标记应包含在响应中
秘密 API 密钥：
* [ ] 如果您的模型连接到 API，则可能会将 API 密钥作为初始化的一部分。使用 Pydantic 的 `SecretStr` 类型来处理秘密，以防止在打印模型时意外泄露。
识别参数：
* [ ] 在识别参数中包含 `model_name`
优化：
考虑提供本机异步支持以减少模型的开销！
* [ ] 提供 `_agenerate` 的本机异步支持（由 `ainvoke` 使用）
* [ ] 提供 `_astream` 的本机异步支持（由 `astream` 使用）
## 下一步
您现在已经学会了如何创建自己的定制聊天模型。
接下来，请查看本部分中有关聊天模型的其他操作指南，比如[如何让模型返回结构化输出](/docs/how_to/structured_output)或[如何跟踪聊天模型令牌使用情况](/docs/how_to/chat_token_usage_tracking)。