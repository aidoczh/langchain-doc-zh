# Javelin AI Gateway 教程

这个 Jupyter Notebook 将探讨如何使用 Python SDK 与 Javelin AI Gateway 进行交互。

Javelin AI Gateway 通过提供安全且统一的端点，促进了大型语言模型（LLMs）如 OpenAI、Cohere、Anthropic 等的利用。该网关本身提供了一个集中的机制，可以系统地推出模型，为企业提供访问安全性、策略和成本控制等。

要查看 Javelin 的所有功能和优势的完整列表，请访问 [www.getjavelin.io](https://www.getjavelin.io)。

## 步骤 1：介绍

[Javelin AI Gateway](https://www.getjavelin.io) 是面向 AI 应用的企业级 API 网关。它集成了强大的访问安全性，确保与大型语言模型的安全交互。在 [官方文档](https://docs.getjavelin.io) 中了解更多。

## 步骤 2：安装

在开始之前，我们必须安装 `javelin_sdk` 并将 Javelin API 密钥设置为环境变量。

```python
pip install 'javelin_sdk'
```

```output
Requirement already satisfied: javelin_sdk in /usr/local/Caskroom/miniconda/base/lib/python3.11/site-packages (0.1.8)
Requirement already satisfied: httpx<0.25.0,>=0.24.0 in /usr/local/Caskroom/miniconda/base/lib/python3.11/site-packages (from javelin_sdk) (0.24.1)
...
Note: you may need to restart the kernel to use updated packages.
```

## 步骤 3：完成示例

本节将演示如何与 Javelin AI Gateway 交互，从大型语言模型获取完成。以下是演示此操作的 Python 脚本：

（注意）假设您在网关中设置了名为 'eng_dept03' 的路由。

```python
from langchain.chains import LLMChain
from langchain_community.llms import JavelinAIGateway
from langchain_core.prompts import PromptTemplate
route_completions = "eng_dept03"
gateway = JavelinAIGateway(
    gateway_uri="http://localhost:8000",  # 替换为 Javelin 的服务 URL 或主机/端口
    route=route_completions,
    model_name="gpt-3.5-turbo-instruct",
)
prompt = PromptTemplate("Translate the following English text to French: {text}")
llmchain = LLMChain(llm=gateway, prompt=prompt)
result = llmchain.run("podcast player")
print(result)
```

```output
---------------------------------------------------------------------------
```

## 步骤 4：嵌入示例

本节演示如何使用 Javelin AI Gateway 获取文本查询和文档的嵌入。以下是演示此操作的 Python 脚本：

（注意）假设您在网关中设置了名为 'embeddings' 的路由。

```python
from langchain_community.embeddings import JavelinAIGatewayEmbeddings
embeddings = JavelinAIGatewayEmbeddings(
    gateway_uri="http://localhost:8000",  # 替换为 Javelin 的服务 URL 或主机/端口
    route="embeddings",
)
print(embeddings.embed_query("hello"))
print(embeddings.embed_documents(["hello"]))
```

```output
---------------------------------------------------------------------------
```

# 第五步：聊天示例

本节演示了如何与 Javelin AI Gateway 交互，以便与大型语言模型进行聊天。以下是一个演示这一过程的 Python 脚本：

（注意）假设您已在网关中设置了一个名为 'mychatbot_route' 的路由

```python
from langchain_community.chat_models import ChatJavelinAIGateway
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(
        content="You are a helpful assistant that translates English to French."
    ),
    HumanMessage(
        content="Artificial Intelligence has the power to transform humanity and make the world a better place"
    ),
]
chat = ChatJavelinAIGateway(
    gateway_uri="http://localhost:8000",  # 替换为 Javelin 的服务 URL 或主机/端口
    route="mychatbot_route",
    model_name="gpt-3.5-turbo",
    params={"temperature": 0.1},
)
print(chat(messages))
```

```output
---------------------------------------------------------------------------
``````output
ImportError                               Traceback (most recent call last)
``````output
Cell In[8], line 1
----> 1 from langchain.chat_models import ChatJavelinAIGateway
      2 from langchain.schema import HumanMessage, SystemMessage
      4 messages = [
      5     SystemMessage(
      6         content="You are a helpful assistant that translates English to French."
   (...)
     10     ),
     11 ]
``````output
ImportError: cannot import name 'ChatJavelinAIGateway' from 'langchain.chat_models' (/usr/local/Caskroom/miniconda/base/lib/python3.11/site-packages/langchain/chat_models/__init__.py)
```

# 第六步：结论

本教程介绍了 Javelin AI Gateway，并演示了如何使用 Python SDK 与其交互。

记得查看 Javelin [Python SDK](https://www.github.com/getjavelin.io/javelin-python) 获取更多示例，并探索官方文档以获取更多详细信息。

祝编码愉快！

```