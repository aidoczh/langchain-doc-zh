# KoboldAI API

[KoboldAI](https://github.com/KoboldAI/KoboldAI-Client) 是一个“基于浏览器的前端，用于多个本地和远程 AI 模型辅助写作的工具...”。它具有公共和本地 API，可以在 langchain 中使用。

以下示例介绍了如何在 LangChain 中使用该 API。

文档可以在浏览器中添加 /api 到您的端点末尾找到（例如 http://127.0.0.1/:5000/api）。

```python
from langchain_community.llms import KoboldApiLLM
```

将下面看到的端点替换为在启动 webui 时显示的端点，使用 --api 或 --public-api。

可选地，您可以传入参数，如温度或最大长度。

```python
llm = KoboldApiLLM(endpoint="http://192.168.1.144:5000", max_length=80)
```

```python
response = llm.invoke(
    "### 指令：\n圣经的第一本书是什么？\n### 回复："
)
```