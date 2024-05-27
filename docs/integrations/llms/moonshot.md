# MoonshotChat

[Moonshot](https://platform.moonshot.cn/) 是一家中国初创公司，为企业和个人提供 LLM 服务。

以下示例介绍如何使用 LangChain 与 Moonshot 进行交互。

```python
from langchain_community.llms.moonshot import Moonshot
```

```python
import os
# 从以下链接生成您的 API 密钥：https://platform.moonshot.cn/console/api-keys
os.environ["MOONSHOT_API_KEY"] = "MOONSHOT_API_KEY"
```

```python
llm = Moonshot()
# 或者使用特定模型
# 可用模型：https://platform.moonshot.cn/docs
# llm = Moonshot(model="moonshot-v1-128k")
```

```python
# 提问模型
llm.invoke("熊猫和熊之间有什么区别？")
```