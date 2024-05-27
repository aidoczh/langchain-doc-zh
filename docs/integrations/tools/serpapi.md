# SerpAPI

本文介绍如何使用 SerpAPI 组件来搜索网络。

```python
from langchain_community.utilities import SerpAPIWrapper
```

```python
search = SerpAPIWrapper()
```

```python
search.run("奥巴马的名字是什么？")
```

```output
'巴拉克·侯赛因·奥巴马二世'
```

## 自定义参数

您还可以使用任意参数自定义 SerpAPI 包装器。例如，在下面的示例中，我们将使用 `bing` 而不是 `google`。

```python
params = {
    "engine": "bing",
    "gl": "us",
    "hl": "en",
}
search = SerpAPIWrapper(params=params)
```

```python
search.run("奥巴马的名字是什么？")
```

```output
'巴拉克·侯赛因·奥巴马二世是一位美国政治家，曾于2009年至2017年担任美国第44任总统。奥巴马是民主党成员，也是美国历史上第一位非裔美国总统。在此之前，他曾于2005年至2008年担任伊利诺伊州的美国参议员，以及于1997年至2004年担任伊利诺伊州参议员，并在从政之前是一名民权律师。Wikipedia barackobama.com'
```

```python
from langchain_core.tools import Tool
# 您可以创建工具并传递给代理
repl_tool = Tool(
    name="python_repl",
    description="一个 Python shell。使用它来执行 Python 命令。输入应该是有效的 Python 命令。如果您想要查看一个值的输出，您应该使用 `print(...)` 打印出来。",
    func=search.run,
)
```