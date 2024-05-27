# StackExchange

>[Stack Exchange](https://stackexchange.com/) 是一个涵盖各个领域主题的问答网站网络，每个网站都涵盖特定的主题，用户可以在这些网站上提问、回答问题，并且通过声望奖励机制进行评价。这个声望系统使得网站能够自我管理。

``StackExchange`` 组件将 StackExchange API 集成到 LangChain 中，使得可以访问 Stack Exchange 网络中的 [StackOverflow](https://stackoverflow.com/) 网站。Stack Overflow 主要关注计算机编程。

本文介绍如何使用 ``StackExchange`` 组件。

首先，我们需要安装实现了 Stack Exchange API 的 Python 包 stackapi。

```python
pip install --upgrade stackapi
```

```python
from langchain_community.utilities import StackExchangeAPIWrapper
stackexchange = StackExchangeAPIWrapper()
stackexchange.run("zsh: command not found: python")
```

