# Fiddler

[Fiddler](https://www.fiddler.ai/) 提供了一个统一的平台，可以在企业规模下监控、解释、分析和改进机器学习部署。

## 安装和设置

使用 Fiddler 设置您的模型[链接](https://demo.fiddler.ai)：

* 用于连接到 Fiddler 的 URL

* 您的组织 ID

* 您的授权令牌

安装 Python 包：

```bash
pip install fiddler-client
```

## 回调

```python
from langchain_community.callbacks.fiddler_callback import FiddlerCallbackHandler
```

查看一个[示例](/docs/integrations/callbacks/fiddler)。