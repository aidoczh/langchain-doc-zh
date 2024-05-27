# Wolfram Alpha

本笔记介绍如何使用 Wolfram Alpha 组件。

首先，您需要设置您的 Wolfram Alpha 开发者账户并获取您的 APP ID：

1. 前往 [这里](https://developer.wolframalpha.com/) 的 Wolfram Alpha 网站注册开发者账户

2. 创建一个应用并获取您的 APP ID

3. 运行 `pip install wolframalpha` 安装 Wolfram Alpha Python 包

然后，我们需要设置一些环境变量：

1. 将您的 APP ID 保存到 WOLFRAM_ALPHA_APPID 环境变量中

```python
pip install wolframalpha
```

```python
import os
os.environ["WOLFRAM_ALPHA_APPID"] = "您的APP ID"
```

```python
from langchain_community.utilities.wolfram_alpha import WolframAlphaAPIWrapper
```

```python
wolfram = WolframAlphaAPIWrapper()
```

```python
wolfram.run("What is 2x+5 = -3x + 7?")
```

```output
'x = 2/5'
```