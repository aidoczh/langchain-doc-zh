# AzureMLChatOnlineEndpoint

>[Azure Machine Learning](https://azure.microsoft.com/en-us/products/machine-learning/) 是一个用于构建、训练和部署机器学习模型的平台。用户可以在模型目录中探索要部署的模型类型，该目录提供了来自不同提供商的基础和通用模型。

通常情况下，您需要部署模型以使用其预测（推理）。在 `Azure Machine Learning` 中，[在线端点](https://learn.microsoft.com/en-us/azure/machine-learning/concept-endpoints) 用于部署这些模型以进行实时服务。它们基于 `端点` 和 `部署` 的思想，允许您将生产工作负载的接口与提供服务的实现解耦。

本笔记本介绍了如何使用托管在 `Azure Machine Learning Endpoint` 上的聊天模型。

```python
from langchain_community.chat_models.azureml_endpoint import AzureMLChatOnlineEndpoint
```

## 设置

您必须在 Azure ML 上[部署模型](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-use-foundation-models?view=azureml-api-2#deploying-foundation-models-to-endpoints-for-inferencing) 或[到 Azure AI studio](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/deploy-models-open) 并获取以下参数：

- `endpoint_url`：端点提供的 REST 端点 URL。

- `endpoint_api_type`：在部署模型到**专用端点**（托管的托管基础设施）时使用 `endpoint_type='dedicated'`。在使用**按使用量付费**提供的服务时使用 `endpoint_type='serverless'`。

- `endpoint_api_key`：端点提供的 API 密钥。

## 内容格式化程序

`content_formatter` 参数是一个处理程序类，用于转换 AzureML 端点的请求和响应，以匹配所需的模式。由于模型目录中有各种模型，每个模型可能会以不同方式处理数据，因此提供了一个 `ContentFormatterBase` 类，允许用户根据自己的喜好转换数据。提供了以下内容格式化程序：

- `CustomOpenAIChatContentFormatter`：为像 LLaMa2-chat 这样遵循 OpenAI API 规范的模型格式化请求和响应数据。

*注意：`langchain.chat_models.azureml_endpoint.LlamaChatContentFormatter` 正在被弃用，并将被 `langchain.chat_models.azureml_endpoint.CustomOpenAIChatContentFormatter` 替换。*

您可以根据自己的模型实现特定的自定义内容格式化程序，从 `langchain_community.llms.azureml_endpoint.ContentFormatterBase` 类派生。

## 示例

以下部分包含有关如何使用此类的示例：

### 示例：使用实时端点进行聊天完成

```python
from langchain_community.chat_models.azureml_endpoint import (
    AzureMLEndpointApiType,
    CustomOpenAIChatContentFormatter,
)
from langchain_core.messages import HumanMessage
chat = AzureMLChatOnlineEndpoint(
    endpoint_url="https://<your-endpoint>.<your_region>.inference.ml.azure.com/score",
    endpoint_api_type=AzureMLEndpointApiType.dedicated,
    endpoint_api_key="my-api-key",
    content_formatter=CustomOpenAIChatContentFormatter(),
)
response = chat.invoke(
    [HumanMessage(content="Will the Collatz conjecture ever be solved?")]
)
response
```

```output
AIMessage(content='  The Collatz Conjecture is one of the most famous unsolved problems in mathematics, and it has been the subject of much study and research for many years. While it is impossible to predict with certainty whether the conjecture will ever be solved, there are several reasons why it is considered a challenging and important problem:\n\n1. Simple yet elusive: The Collatz Conjecture is a deceptively simple statement that has proven to be extraordinarily difficult to prove or disprove. Despite its simplicity, the conjecture has eluded some of the brightest minds in mathematics, and it remains one of the most famous open problems in the field.\n2. Wide-ranging implications: The Collatz Conjecture has far-reaching implications for many areas of mathematics, including number theory, algebra, and analysis. A solution to the conjecture could have significant impacts on these fields and potentially lead to new insights and discoveries.\n3. Computational evidence: While the conjecture remains unproven, extensive computational evidence supports its validity. In fact, no counterexample to the conjecture has been found for any starting value up to 2^64 (a number', additional_kwargs={}, example=False)
```

### 示例：使用按使用量付费部署（模型即服务）进行聊天完成

```python
chat = AzureMLChatOnlineEndpoint(
    endpoint_url="https://<your-endpoint>.<your_region>.inference.ml.azure.com/v1/chat/completions",
    endpoint_api_type=AzureMLEndpointApiType.serverless,
    endpoint_api_key="my-api-key",
    content_formatter=CustomOpenAIChatContentFormatter,
)
response = chat.invoke(
    [HumanMessage(content="Will the Collatz conjecture ever be solved?")]
)
response
```

如果您需要向模型传递额外的参数，请使用 `model_kwargs` 参数：

```python
chat = AzureMLChatOnlineEndpoint(
    endpoint_url="https://<your-endpoint>.<your_region>.inference.ml.azure.com/v1/chat/completions",
    endpoint_api_type=AzureMLEndpointApiType.serverless,
    endpoint_api_key="my-api-key",
    content_formatter=CustomOpenAIChatContentFormatter,
    model_kwargs={"temperature": 0.8},
)
```

参数也可以在调用过程中传递：

```python
response = chat.invoke(
    [HumanMessage(content="Will the Collatz conjecture ever be solved?")],
    max_tokens=512,
)
response
```

```