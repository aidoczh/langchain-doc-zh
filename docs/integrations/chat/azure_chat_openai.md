# AzureChatOpenAI

>[Azure OpenAI 服务](https://learn.microsoft.com/en-us/azure/ai-services/openai/overview) 提供了对 OpenAI 强大语言模型的 REST API 访问，包括 GPT-4、GPT-3.5-Turbo 和嵌入模型系列。这些模型可以轻松适应您的特定任务，包括但不限于内容生成、摘要、语义搜索和自然语言转代码。用户可以通过 REST API、Python SDK 或 Azure OpenAI Studio 中的基于 Web 的界面访问该服务。

本文介绍如何连接到托管在 Azure 上的 OpenAI 端点。首先，我们需要安装 `langchain-openai` 包。

%pip install -qU langchain-openai

接下来，让我们设置一些环境变量，以帮助我们连接到 Azure OpenAI 服务。您可以在 Azure 门户中找到这些值。

```python
import os
os.environ["AZURE_OPENAI_API_KEY"] = "..."
os.environ["AZURE_OPENAI_ENDPOINT"] = "https://<your-endpoint>.openai.azure.com/"
os.environ["AZURE_OPENAI_API_VERSION"] = "2023-06-01-preview"
os.environ["AZURE_OPENAI_CHAT_DEPLOYMENT_NAME"] = "chat"
```

接下来，让我们构建我们的模型并与之交流：

```python
from langchain_core.messages import HumanMessage
from langchain_openai import AzureChatOpenAI
model = AzureChatOpenAI(
    openai_api_version=os.environ["AZURE_OPENAI_API_VERSION"],
    azure_deployment=os.environ["AZURE_OPENAI_CHAT_DEPLOYMENT_NAME"],
)
```

```python
message = HumanMessage(
    content="Translate this sentence from English to French. I love programming."
)
model.invoke([message])
```

```output
AIMessage(content="J'adore programmer.", response_metadata={'token_usage': {'completion_tokens': 6, 'prompt_tokens': 19, 'total_tokens': 25}, 'model_name': 'gpt-35-turbo', 'system_fingerprint': None, 'prompt_filter_results': [{'prompt_index': 0, 'content_filter_results': {'hate': {'filtered': False, 'severity': 'safe'}, 'self_harm': {'filtered': False, 'severity': 'safe'}, 'sexual': {'filtered': False, 'severity': 'safe'}, 'violence': {'filtered': False, 'severity': 'safe'}}}], 'finish_reason': 'stop', 'logprobs': None, 'content_filter_results': {'hate': {'filtered': False, 'severity': 'safe'}, 'self_harm': {'filtered': False, 'severity': 'safe'}, 'sexual': {'filtered': False, 'severity': 'safe'}, 'violence': {'filtered': False, 'severity': 'safe'}}}, id='run-25ed88db-38f2-4b0c-a943-a03f217711a9-0')
```

## 模型版本

Azure OpenAI 响应包含 `model` 属性，该属性是用于生成响应的模型的名称。然而，与原生的 OpenAI 响应不同，它不包含模型的版本，该版本在 Azure 中的部署上设置。这使得很难知道用于生成响应的模型的版本，这可能导致例如使用 `OpenAICallbackHandler` 进行错误的总成本计算。

为了解决这个问题，您可以将 `model_version` 参数传递给 `AzureChatOpenAI` 类，它将被添加到 llm 输出中的模型名称中。这样，您可以轻松区分不同版本的模型。

```python
from langchain_community.callbacks import get_openai_callback
```

```python
model = AzureChatOpenAI(
    openai_api_version=os.environ["AZURE_OPENAI_API_VERSION"],
    azure_deployment=os.environ[
        "AZURE_OPENAI_CHAT_DEPLOYMENT_NAME"
    ],  # 在 Azure 中，此部署的版本为 0613 - 输入和输出令牌分别计数
)
with get_openai_callback() as cb:
    model.invoke([message])
    print(
        f"Total Cost (USD): ${format(cb.total_cost, '.6f')}"
    )  # 如果不指定模型版本，将使用每 1k 输入和输出令牌的定价 0.002 美元
```

```output
Total Cost (USD): $0.000041
```

我们可以向 `AzureChatOpenAI` 构造函数提供模型版本。它将附加到 Azure OpenAI 返回的模型名称，并且成本将被正确计算。

```python
model0301 = AzureChatOpenAI(
    openai_api_version=os.environ["AZURE_OPENAI_API_VERSION"],
    azure_deployment=os.environ["AZURE_OPENAI_CHAT_DEPLOYMENT_NAME"],
    model_version="0301",
)
with get_openai_callback() as cb:
    model0301.invoke([message])
    print(f"Total Cost (USD): ${format(cb.total_cost, '.6f')}")
```

```output
Total Cost (USD): $0.000044
```