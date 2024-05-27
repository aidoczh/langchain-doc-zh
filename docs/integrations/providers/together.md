# Together AI

[Together AI](https://www.together.ai/) 提供了一个 API，可以用几行代码查询[50多个领先的开源模型](https://docs.together.ai/docs/inference-models)。

这个示例演示了如何使用 LangChain 与 Together AI 模型进行交互。

## 安装

```python
%pip install --upgrade langchain-together
```

## 环境

要使用 Together AI，您需要一个 API 密钥，您可以在这里找到：https://api.together.ai/settings/api-keys。这可以作为初始化参数``together_api_key``传递，也可以设置为环境变量``TOGETHER_API_KEY``。

## 示例

```python
# 使用 Together AI 查询聊天模型
from langchain_together import ChatTogether
# 从这里选择我们的 50 多个模型：https://docs.together.ai/docs/inference-models
chat = ChatTogether(
    # together_api_key="YOUR_API_KEY",
    model="meta-llama/Llama-3-70b-chat-hf",
)
# 从模型中流式传输响应
for m in chat.stream("Tell me fun things to do in NYC"):
    print(m.content, end="", flush=True)
# 如果您不想使用流式传输，可以使用 invoke 方法
# chat.invoke("Tell me fun things to do in NYC")
```

```python
# 使用 Together AI 查询代码和语言模型
from langchain_together import Together
llm = Together(
    model="codellama/CodeLlama-70b-Python-hf",
    # together_api_key="..."
)
print(llm.invoke("def bubble_sort(): "))
```