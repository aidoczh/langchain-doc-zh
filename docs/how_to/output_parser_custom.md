# 如何创建自定义输出解析器
在某些情况下，您可能希望实现一个自定义解析器，将模型的输出结构化为自定义格式。
有两种实现自定义解析器的方式：
1. 使用 LCEL 中的 `RunnableLambda` 或 `RunnableGenerator` —— 我们强烈推荐大多数情况下使用这种方式
2. 通过继承输出解析的基类之一 —— 这是一种较为困难的做法
这两种方法的区别主要是表面的，主要体现在触发哪些回调（例如 `on_chain_start` 与 `on_parser_start`），以及可追踪平台（如 LangSmith）中可视化可运行的 lambda 函数与解析器的方式。
## 可运行的 Lambda 函数和生成器
推荐的解析方式是使用 **可运行的 lambda 函数** 和 **可运行的生成器**！
在这里，我们将创建一个简单的解析器，将模型的输出大小写反转。
例如，如果模型输出为："Meow"，解析器将产生 "mEOW"。
```python
from typing import Iterable
from langchain_anthropic.chat_models import ChatAnthropic
from langchain_core.messages import AIMessage, AIMessageChunk
model = ChatAnthropic(model_name="claude-2.1")
def parse(ai_message: AIMessage) -> str:
    """解析 AI 消息。"""
    return ai_message.content.swapcase()
chain = model | parse
chain.invoke("hello")
```
```output
'hELLO!'
```
:::tip
LCEL 在使用 `|` 语法组合时，会自动将函数 `parse` 升级为 `RunnableLambda(parse)`。
如果您不喜欢这种方式，可以手动导入 `RunnableLambda`，然后运行 `parse = RunnableLambda(parse)`。
:::
流式处理是否有效？
```python
for chunk in chain.stream("tell me about yourself in one sentence"):
    print(chunk, end="|", flush=True)
```
```output
i'M cLAUDE, AN ai ASSISTANT CREATED BY aNTHROPIC TO BE HELPFUL, HARMLESS, AND HONEST.|
```
不，因为解析器在解析输出之前会聚合输入。
如果我们想要实现流式解析器，可以让解析器接受输入的可迭代对象，并在结果可用时产生结果。
```python
from langchain_core.runnables import RunnableGenerator
def streaming_parse(chunks: Iterable[AIMessageChunk]) -> Iterable[str]:
    for chunk in chunks:
        yield chunk.content.swapcase()
streaming_parse = RunnableGenerator(streaming_parse)
```
:::important
请使用 `RunnableGenerator` 包装流式解析器，因为我们可能不会自动使用 `|` 语法升级它。
:::
```python
chain = model | streaming_parse
chain.invoke("hello")
```
```output
'hELLO!'
```
让我们确认一下流式处理是否有效！
```python
for chunk in chain.stream("tell me about yourself in one sentence"):
    print(chunk, end="|", flush=True)
```
```output
i|'M| cLAUDE|,| AN| ai| ASSISTANT| CREATED| BY| aN|THROP|IC| TO| BE| HELPFUL|,| HARMLESS|,| AND| HONEST|.|
```
## 继承解析基类
另一种实现解析器的方法是通过继承 `BaseOutputParser`、`BaseGenerationOutputParser` 或其他基本解析器中的一个，具体取决于您的需求。
总的来说，我们**不建议**大多数情况下使用这种方式，因为它会导致编写更多的代码而没有显著的好处。
最简单的输出解析器扩展了 `BaseOutputParser` 类，并且必须实现以下方法：
- `parse`：接受模型的字符串输出并对其进行解析
- （可选）`_type`：标识解析器的名称
当聊天模型或 LLM 的输出格式不正确时，可以抛出 `OutputParserException` 来指示由于坏输入而解析失败。使用此异常允许利用解析器的代码以一致的方式处理异常。
:::tip 解析器是可运行的！ 🏃
因为 `BaseOutputParser` 实现了 `Runnable` 接口，您通过这种方式创建的任何自定义解析器都将成为有效的 LangChain 可运行对象，并且将受益于自动的异步支持、批处理接口、日志支持等。
:::
### 简单解析器
下面是一个简单的解析器，可以解析表示布尔值的 **字符串**（例如 `YES` 或 `NO`）并将其转换为相应的 `boolean` 类型。
```python
from langchain_core.exceptions import OutputParserException
from langchain_core.output_parsers import BaseOutputParser
# [bool] 描述了一个泛型的参数化。
# 它基本上指示了 parse 的返回类型
# 在这种情况下，返回类型要么是 True，要么是 False
class BooleanOutputParser(BaseOutputParser[bool]):
    """自定义布尔解析器。"""
    true_val: str = "YES"
    false_val: str = "NO"
    def parse(self, text: str) -> bool:
        cleaned_text = text.strip().upper()
        if cleaned_text not in (self.true_val.upper(), self.false_val.upper()):
            raise OutputParserException(
                f"BooleanOutputParser 期望输出值为 {self.true_val} 或 {self.false_val}（不区分大小写）。收到 {cleaned_text}。"
            )
        return cleaned_text == self.true_val.upper()
    @property
    def _type(self) -> str:
        return "boolean_output_parser"
```
```python
parser = BooleanOutputParser()
parser.invoke("YES")
```
```output
True
```
```python
try:
    parser.invoke("MEOW")
except Exception as e:
    print(f"Triggered an exception of type: {type(e)}")
```
```output
Triggered an exception of type: <class 'langchain_core.exceptions.OutputParserException'>
```
让我们测试更改参数设置
```python
parser = BooleanOutputParser(true_val="OKAY")
parser.invoke("OKAY")
```
```output
True
```
让我们确认其他 LCEL 方法是否存在
```python
parser.batch(["OKAY", "NO"])
```
```output
[True, False]
```
```python
await parser.abatch(["OKAY", "NO"])
```
```output
[True, False]
```
```python
from langchain_anthropic.chat_models import ChatAnthropic
anthropic = ChatAnthropic(model_name="claude-2.1")
anthropic.invoke("say OKAY or NO")
```
```output
AIMessage(content='OKAY')
```
让我们测试我们的解析器是否有效！
```python
chain = anthropic | parser
chain.invoke("say OKAY or NO")
```
```output
True
```
:::note
该解析器将适用于 LLM 的输出（字符串）或聊天模型的输出（`AIMessage`）！
:::
### 解析原始模型输出
有时除了原始文本之外，模型输出中还有重要的元数据。一个例子是工具调用，其中用于传递给被调用函数的参数以单独的属性返回。如果您需要这种更精细的控制，可以子类化 `BaseGenerationOutputParser` 类。
这个类需要一个名为 `parse_result` 的方法。该方法接受原始模型输出（例如 `Generation` 或 `ChatGeneration` 的列表）并返回解析后的输出。
支持 `Generation` 和 `ChatGeneration` 使解析器可以同时处理常规 LLM 和聊天模型。
```python
from typing import List
from langchain_core.exceptions import OutputParserException
from langchain_core.messages import AIMessage
from langchain_core.output_parsers import BaseGenerationOutputParser
from langchain_core.outputs import ChatGeneration, Generation
class StrInvertCase(BaseGenerationOutputParser[str]):
    """一个示例解析器，用于反转消息中字符的大小写。
    这只是一个演示示例解析器，目的是保持示例尽可能简单。
    """
    def parse_result(self, result: List[Generation], *, partial: bool = False) -> str:
        """将模型生成的列表解析为特定格式。
        Args:
            result: 要解析的 Generation 列表。假定这些 Generation 是单个模型输入的不同候选输出。
                许多解析器假定只传递了单个生成。
                我们将对此进行断言
            partial: 是否允许部分结果。这用于支持流式传输的解析器
        """
        if len(result) != 1:
            raise NotImplementedError("此输出解析器只能用于单个生成。")
        generation = result[0]
        if not isinstance(generation, ChatGeneration):
            # 表明这个只能用于聊天生成
            raise OutputParserException("此输出解析器只能用于聊天生成。")
        return generation.message.content.swapcase()
chain = anthropic | StrInvertCase()
```
让我们测试新的解析器！它应该会反转模型的输出。
```python
chain.invoke("Tell me a short sentence about yourself")
```
```output
'hELLO! mY NAME IS cLAUDE.'
```