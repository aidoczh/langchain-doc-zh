# 如何创建自定义的LLM类
本文档介绍如何创建自定义的LLM包装器，以便您可以使用自己的LLM或与LangChain支持的包装器不同的包装器。
将您的LLM与标准的`LLM`接口包装在一起，可以让您在现有的LangChain程序中使用您的LLM，并且只需进行最少的代码修改！
作为额外的好处，您的LLM将自动成为LangChain的`Runnable`，并且将受益于一些开箱即用的优化、异步支持、`astream_events` API等。
## 实现
自定义LLM只需要实现两个必需的内容：
| 方法           | 描述                                                         |
|----------------|--------------------------------------------------------------|
| `_call`        | 接受一个字符串和一些可选的停用词，并返回一个字符串。被`invoke`使用。 |
| `_llm_type`    | 返回一个字符串的属性，仅用于记录目的。                       
可选的实现：
| 方法                | 描述                                                                                   |
|---------------------|----------------------------------------------------------------------------------------|
| `_identifying_params` | 用于帮助识别模型并打印LLM；应返回一个字典。这是一个 **@property**。                 |
| `_acall`             | 提供`_call`的异步本机实现，被`ainvoke`使用。                                           |
| `_stream`            | 逐个令牌流式输出的方法。                                                               |
| `_astream`           | 提供`_stream`的异步本机实现；在较新的LangChain版本中，默认为`_stream`。             |
让我们实现一个简单的自定义LLM，它只返回输入的前n个字符。
```python
from typing import Any, Dict, Iterator, List, Mapping, Optional
from langchain_core.callbacks.manager import CallbackManagerForLLMRun
from langchain_core.language_models.llms import LLM
from langchain_core.outputs import GenerationChunk
class CustomLLM(LLM):
    """一个自定义聊天模型，回显输入的最后`n`个字符。
    在为LangChain贡献实现时，仔细记录模型，包括初始化参数，包括如何初始化模型的示例，并包括任何相关的链接到底层模型的文档或API。
    示例：
        .. code-block:: python
            model = CustomChatModel(n=2)
            result = model.invoke([HumanMessage(content="hello")])
            result = model.batch([[HumanMessage(content="hello")],
                                 [HumanMessage(content="world")]])
    """
    n: int
    """要回显的输入的字符数。"""
    def _call(
        self,
        prompt: str,
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> str:
        """在给定输入上运行LLM。
        重写此方法以实现LLM逻辑。
        参数：
            prompt：要生成的提示。
            stop：在生成时要使用的停用词。模型输出在任何停用子字符串的第一次出现时被截断。
                如果不支持停用词，请考虑引发NotImplementedError。
            run_manager：运行的回调管理器。
            **kwargs：任意的额外关键字参数。这些通常传递给模型提供者API调用。
        返回：
            作为字符串的模型输出。实际完成不应包括提示。
        """
        if stop is not None:
            raise ValueError("不允许使用停用词参数。")
        return prompt[: self.n]
    def _stream(
        self,
        prompt: str,
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> Iterator[GenerationChunk]:
        """在给定提示上流式传输LLM。
        子类应该重写此方法以支持流式传输。
        如果未实现，调用stream的默认行为将退回到模型的非流式版本，并将输出作为单个块返回。
        参数：
            prompt：要生成的提示。
            stop：在生成时要使用的停用词。模型输出在任何这些子字符串的第一次出现时被截断。
            run_manager：运行的回调管理器。
            **kwargs：任意的额外关键字参数。这些通常传递给模型提供者API调用。
        返回：
            一个GenerationChunks的迭代器。
        """
        for char in prompt[: self.n]:
            chunk = GenerationChunk(text=char)
            if run_manager:
                run_manager.on_llm_new_token(chunk.text, chunk=chunk)
            yield chunk
    @property
    def _identifying_params(self) -> Dict[str, Any]:
        """返回一个标识参数的字典。"""
        return {
            # 模型名称允许用户在LLM监控应用程序中指定自定义令牌计数规则（例如，在LangSmith中，用户可以为其模型提供每个令牌的定价，并监视给定LLM的成本。）
            "model_name": "CustomChatModel",
        }
    @property
    def _llm_type(self) -> str:
        """获取此聊天模型使用的语言模型的类型。仅用于记录目的。"""
        return "custom"
```
### 让我们来测试一下 🧪
这个 LLM 将实现 LangChain 的标准 `Runnable` 接口，许多 LangChain 抽象都支持！
```python
llm = CustomLLM(n=5)
print(llm)
```
```output
CustomLLM
参数: {'model_name': 'CustomChatModel'}
```
```python
llm.invoke("这是一个 foobar 东西")
```
```output
'This '
```
```python
await llm.ainvoke("world")
```
```output
'world'
```
```python
llm.batch(["woof woof woof", "meow meow meow"])
```
```output
['woof ', 'meow ']
```
```python
await llm.abatch(["woof woof woof", "meow meow meow"])
```
```output
['woof ', 'meow ']
```
```python
async for token in llm.astream("hello"):
    print(token, end="|", flush=True)
```
```output
h|e|l|l|o|
```
让我们确认它与其他 `LangChain` API 很好地集成。
```python
from langchain_core.prompts import ChatPromptTemplate
```
```python
prompt = ChatPromptTemplate.from_messages(
    [("system", "you are a bot"), ("human", "{input}")]
)
```
```python
llm = CustomLLM(n=7)
chain = prompt | llm
```
```python
idx = 0
async for event in chain.astream_events({"input": "hello there!"}, version="v1"):
    print(event)
    idx += 1
    if idx > 7:
        # 截断
        break
```
```output
{'event': 'on_chain_start', 'run_id': '05f24b4f-7ea3-4fb6-8417-3aa21633462f', 'name': 'RunnableSequence', 'tags': [], 'metadata': {}, 'data': {'input': {'input': 'hello there!'}}}
{'event': 'on_prompt_start', 'name': 'ChatPromptTemplate', 'run_id': '7e996251-a926-4344-809e-c425a9846d21', 'tags': ['seq:step:1'], 'metadata': {}, 'data': {'input': {'input': 'hello there!'}}}
{'event': 'on_prompt_end', 'name': 'ChatPromptTemplate', 'run_id': '7e996251-a926-4344-809e-c425a9846d21', 'tags': ['seq:step:1'], 'metadata': {}, 'data': {'input': {'input': 'hello there!'}, 'output': ChatPromptValue(messages=[SystemMessage(content='you are a bot'), HumanMessage(content='hello there!')])}}
{'event': 'on_llm_start', 'name': 'CustomLLM', 'run_id': 'a8766beb-10f4-41de-8750-3ea7cf0ca7e2', 'tags': ['seq:step:2'], 'metadata': {}, 'data': {'input': {'prompts': ['System: you are a bot\nHuman: hello there!']}}}
{'event': 'on_llm_stream', 'name': 'CustomLLM', 'run_id': 'a8766beb-10f4-41de-8750-3ea7cf0ca7e2', 'tags': ['seq:step:2'], 'metadata': {}, 'data': {'chunk': 'S'}}
{'event': 'on_chain_stream', 'run_id': '05f24b4f-7ea3-4fb6-8417-3aa21633462f', 'tags': [], 'metadata': {}, 'name': 'RunnableSequence', 'data': {'chunk': 'S'}}
{'event': 'on_llm_stream', 'name': 'CustomLLM', 'run_id': 'a8766beb-10f4-41de-8750-3ea7cf0ca7e2', 'tags': ['seq:step:2'], 'metadata': {}, 'data': {'chunk': 'y'}}
{'event': 'on_chain_stream', 'run_id': '05f24b4f-7ea3-4fb6-8417-3aa21633462f', 'tags': [], 'metadata': {}, 'name': 'RunnableSequence', 'data': {'chunk': 'y'}}
```
## 贡献
我们感谢所有聊天模型集成的贡献。
以下是一个检查列表，以确保您的贡献被添加到 LangChain 中：
文档：
* 模型包含所有初始化参数的文档字符串，因为这些将显示在 [APIReference](https://api.python.langchain.com/en/stable/langchain_api_reference.html) 中。
* 如果模型由服务提供支持，则模型的类文档字符串包含指向模型 API 的链接。
测试：
* [ ] 为重写的方法添加单元测试或集成测试。如果您已经重写了相应的代码，请验证 `invoke`、`ainvoke`、`batch`、`stream` 是否正常工作。
流式处理（如果您正在实现它）：
* [ ] 确保调用 `on_llm_new_token` 回调
* [ ] 在产生块之前调用 `on_llm_new_token`
停止令牌行为：
* [ ] 应尊重停止令牌
* [ ] 停止令牌应包含在响应中
秘密 API 密钥：
* [ ] 如果您的模型连接到 API，则可能会接受 API 密钥作为其初始化的一部分。使用 Pydantic 的 `SecretStr` 类型来处理秘密，这样当用户打印模型时不会意外打印出秘密。
