# Python REPL

有时，对于复杂的计算，与其让 LLM 直接生成答案，让 LLM 生成计算答案的代码，然后运行该代码以获得答案可能更好。为了方便这样做，我们提供了一个简单的 Python REPL 来执行命令。

此接口只会返回被打印出来的内容 - 因此，如果你想用它来计算一个答案，请确保让它打印出答案。

```python
from langchain_core.tools import Tool
from langchain_experimental.utilities import PythonREPL
```

```python
python_repl = PythonREPL()
```

```python
python_repl.run("print(1+1)")
```

```output
Python REPL 可以执行任意代码。请谨慎使用。
```

```output
'2\n'
```

```python
# 你可以创建一个工具来传递给一个代理
repl_tool = Tool(
    name="python_repl",
    description="一个 Python shell。使用它来执行 Python 命令。输入应该是一个有效的 Python 命令。如果你想看到一个值的输出，你应该用 `print(...)` 来打印出来。",
    func=python_repl.run,
)
```