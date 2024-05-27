# 如何创建自定义工具
在构建代理时，您需要为其提供一个`Tool`列表，以便代理可以使用这些工具。除了实际调用的函数之外，`Tool`由几个组件组成：
| 属性           | 类型                      | 描述                                                                                                          |
|-----------------|---------------------------|--------------------------------------------------------------------------------------------------------------|
| name          | str                     | 在提供给 LLM 或代理的工具集中必须是唯一的。                                                              |
| description   | str                     | 描述工具的功能。LLM 或代理将使用此描述作为上下文。                                                  |
| args_schema   | Pydantic BaseModel      | 可选但建议，可用于提供更多信息（例如，few-shot 示例）或验证预期参数。                     |
| return_direct   | boolean      | 仅对代理相关。当为True时，在调用给定工具后，代理将停止并将结果直接返回给用户。  |
LangChain 提供了三种创建工具的方式：
1. 使用[@tool装饰器](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.tool.html#langchain_core.tools.tool) -- 定义自定义工具的最简单方式。
2. 使用[StructuredTool.from_function](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.StructuredTool.html#langchain_core.tools.StructuredTool.from_function) 类方法 -- 这类似于`@tool`装饰器，但允许更多配置和同步和异步实现的规范。
3. 通过子类化[BaseTool](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.BaseTool.html) -- 这是最灵活的方法，它提供了最大程度的控制，但需要更多的工作量和代码。
`@tool`或`StructuredTool.from_function`类方法对于大多数用例应该足够了。
:::提示
如果工具具有精心选择的名称、描述和 JSON 模式，模型的性能会更好。
:::
## @tool 装饰器
这个`@tool`装饰器是定义自定义工具的最简单方式。该装饰器默认使用函数名称作为工具名称，但可以通过传递字符串作为第一个参数来覆盖。此外，装饰器将使用函数的文档字符串作为工具的描述 - 因此必须提供文档字符串。
```python
from langchain_core.tools import tool
@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b
# 让我们检查与该工具关联的一些属性。
print(multiply.name)
print(multiply.description)
print(multiply.args)
```
```output
multiply
multiply(a: int, b: int) -> int - Multiply two numbers.
{'a': {'title': 'A', 'type': 'integer'}, 'b': {'title': 'B', 'type': 'integer'}}
```
或者创建一个**异步**实现，如下所示：
```python
from langchain_core.tools import tool
@tool
async def amultiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b
```
您还可以通过将它们传递给工具装饰器来自定义工具名称和 JSON 参数。
```python
from langchain.pydantic_v1 import BaseModel, Field
class CalculatorInput(BaseModel):
    a: int = Field(description="first number")
    b: int = Field(description="second number")
@tool("multiplication-tool", args_schema=CalculatorInput, return_direct=True)
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b
# 让我们检查与该工具关联的一些属性。
print(multiply.name)
print(multiply.description)
print(multiply.args)
print(multiply.return_direct)
```
```output
multiplication-tool
multiplication-tool(a: int, b: int) -> int - Multiply two numbers.
{'a': {'title': 'A', 'description': 'first number', 'type': 'integer'}, 'b': {'title': 'B', 'description': 'second number', 'type': 'integer'}}
True
```
## StructuredTool
`StructuredTool.from_function` 类方法提供了比`@tool`装饰器更多的可配置性，而无需太多额外的代码。
```python
from langchain_core.tools import StructuredTool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b
async def amultiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b
calculator = StructuredTool.from_function(func=multiply, coroutine=amultiply)
print(calculator.invoke({"a": 2, "b": 3}))
print(await calculator.ainvoke({"a": 2, "b": 5}))
```
```output
6
10
```
要进行配置：
```python
class CalculatorInput(BaseModel):
    a: int = Field(description="first number")
    b: int = Field(description="second number")
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b
calculator = StructuredTool.from_function(
    func=multiply,
    name="Calculator",
    description="multiply numbers",
    args_schema=CalculatorInput,
    return_direct=True,
    # coroutine= ... <- 如果需要，也可以指定异步方法
)
print(calculator.invoke({"a": 2, "b": 3}))
print(calculator.name)
print(calculator.description)
print(calculator.args)
```
## 处理工具错误
如果您正在使用带有代理的工具，您可能需要一个错误处理策略，以便代理可以从错误中恢复并继续执行。
一个简单的策略是在工具内部抛出 `ToolException`，并使用 `handle_tool_error` 指定一个错误处理程序。
当指定了错误处理程序时，异常将被捕获，错误处理程序将决定从工具返回哪个输出。
您可以将 `handle_tool_error` 设置为 `True`、字符串值或函数。如果是函数，该函数应该以 `ToolException` 作为参数，并返回一个值。
请注意，仅仅抛出 `ToolException` 是不会生效的。您需要首先设置工具的 `handle_tool_error`，因为其默认值是 `False`。
```python
from langchain_core.tools import ToolException
def get_weather(city: str) -> int:
    """获取给定城市的天气。"""
    raise ToolException(f"错误：没有名为{city}的城市。")
```
下面是一个使用默认的`handle_tool_error=True`行为的示例。
```python
get_weather_tool = StructuredTool.from_function(
    func=get_weather,
    handle_tool_error=True,
)
get_weather_tool.invoke({"city": "foobar"})
```
```output
'错误：没有名为foobar的城市。'
```
我们可以将`handle_tool_error`设置为一个始终返回的字符串。
```python
get_weather_tool = StructuredTool.from_function(
    func=get_weather,
    handle_tool_error="没有这样的城市，但可能在那里的温度超过0K！",
)
get_weather_tool.invoke({"city": "foobar"})
```
```output
"没有这样的城市，但可能在那里的温度超过0K！"
```
使用函数处理错误：
```python
def _handle_error(error: ToolException) -> str:
    return f"工具执行期间发生以下错误：`{error.args[0]}`"
get_weather_tool = StructuredTool.from_function(
    func=get_weather,
    handle_tool_error=_handle_error,
)
get_weather_tool.invoke({"city": "foobar"})
```
```output
'工具执行期间发生以下错误：`错误：没有名为foobar的城市。`'
```
