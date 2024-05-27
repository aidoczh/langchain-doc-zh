

# Azure OpenAI

本文介绍如何在 [Azure OpenAI](https://aka.ms/azure-openai) 中使用 Langchain。

Azure OpenAI API 兼容 OpenAI 的 API。`openai` Python 软件包使得同时使用 OpenAI 和 Azure OpenAI 变得简单。您可以按照以下异常注意事项调用 Azure OpenAI，方式与调用 OpenAI 相同。

## API 配置

您可以通过环境变量配置 `openai` 软件包以使用 Azure OpenAI。以下是针对 `bash` 的配置：

```bash
# 您想要使用的 API 版本：将其设置为 `2023-12-01-preview` 以使用发布版本。
export OPENAI_API_VERSION=2023-12-01-preview
# Azure OpenAI 资源的基本 URL。您可以在 Azure 门户中找到此信息，位于您的 Azure OpenAI 资源下。
export AZURE_OPENAI_ENDPOINT=https://your-resource-name.openai.azure.com
# Azure OpenAI 资源的 API 密钥。您可以在 Azure 门户中找到此信息，位于您的 Azure OpenAI 资源下。
export AZURE_OPENAI_API_KEY=<your Azure OpenAI API key>
```

另外，您也可以在运行中的 Python 环境中配置 API：

```python
import os
os.environ["OPENAI_API_VERSION"] = "2023-12-01-preview"
```

## Azure Active Directory 认证

有两种方式可以对 Azure OpenAI 进行认证：

- API 密钥

- Azure Active Directory (AAD)

使用 API 密钥是最简单的入门方式。您可以在 Azure 门户中找到您的 API 密钥，位于您的 Azure OpenAI 资源下。

然而，如果您有复杂的安全需求，您可能希望使用 Azure Active Directory。您可以在 [此处](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/managed-identity) 找到有关如何在 Azure OpenAI 中使用 AAD 的更多信息。

如果您在本地开发，您需要安装 Azure CLI 并登录。您可以在 [此处](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) 安装 Azure CLI。然后，运行 `az login` 进行登录。

添加一个角色分配到您的 Azure OpenAI 资源，角色为 `Cognitive Services OpenAI User`。这将允许您从 AAD 获取一个令牌以用于 Azure OpenAI。您可以将此角色分配给用户、组、服务主体或托管标识。有关 Azure OpenAI RBAC 角色的更多信息，请参阅 [此处](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/role-based-access-control)。

要在 Python 中使用 AAD 和 LangChain，安装 `azure-identity` 软件包。然后，将 `OPENAI_API_TYPE` 设置为 `azure_ad`。接下来，使用 `DefaultAzureCredential` 类通过调用 `get_token` 来从 AAD 获取一个令牌，如下所示。最后，将 `OPENAI_API_KEY` 环境变量设置为令牌值。

```python
import os
from azure.identity import DefaultAzureCredential
# 获取 Azure 凭据
credential = DefaultAzureCredential()
# 将 API 类型设置为 `azure_ad`
os.environ["OPENAI_API_TYPE"] = "azure_ad"
# 将 API_KEY 设置为来自 Azure 凭据的令牌
os.environ["OPENAI_API_KEY"] = credential.get_token("https://cognitiveservices.azure.com/.default").token
```

`DefaultAzureCredential` 类是使用 AAD 进行认证的简单方式。如果需要，您还可以自定义凭据链。在下面的示例中，我们首先尝试托管标识，然后回退到 Azure CLI。如果您在 Azure 中运行代码但希望在本地开发，这将非常有用。

```python
from azure.identity import ChainedTokenCredential, ManagedIdentityCredential, AzureCliCredential
credential = ChainedTokenCredential(
    ManagedIdentityCredential(),
    AzureCliCredential()
)
```

## 部署

使用 Azure OpenAI，您可以设置自己的 GPT-3 和 Codex 模型的部署。在调用 API 时，您需要指定要使用的部署。

_**注意**：这些文档适用于 Azure 文本补全模型。像 GPT-4 这样的模型是聊天模型。它们具有略有不同的接口，并且可以通过 `AzureChatOpenAI` 类访问。有关 Azure 聊天的文档，请参阅 [Azure Chat OpenAI 文档](/docs/integrations/chat/azure_chat_openai)_。

假设您的部署名称是 `gpt-35-turbo-instruct-prod`。在 `openai` Python API 中，您可以使用 `engine` 参数指定此部署。例如：

```python
import openai
client = AzureOpenAI(
    api_version="2023-12-01-preview",
)
response = client.completions.create(
    model="gpt-35-turbo-instruct-prod",
    prompt="Test prompt"
)
```

```python
%pip install --upgrade --quiet  langchain-openai
```

```python
import os
os.environ["OPENAI_API_VERSION"] = "2023-12-01-preview"
os.environ["AZURE_OPENAI_ENDPOINT"] = "..."
os.environ["AZURE_OPENAI_API_KEY"] = "..."
```

```python
# 导入 Azure OpenAI
from langchain_openai import AzureOpenAI
```

```python
# 创建 Azure OpenAI 实例
# 将部署名称替换为您自己的
llm = AzureOpenAI(
    deployment_name="gpt-35-turbo-instruct-0914",
)
```

```python
# 运行LLM
llm.invoke("告诉我一个笑话")
```

```output
"为什么自行车不能自己站起来呢？\n\n因为它太累了！"
```

我们还可以打印LLM并查看其自定义打印输出。

```python
print(llm)
```

```output
AzureOpenAI
Params: {'deployment_name': 'gpt-35-turbo-instruct-0914', 'model_name': 'gpt-3.5-turbo-instruct', 'temperature': 0.7, 'top_p': 1, 'frequency_penalty': 0, 'presence_penalty': 0, 'n': 1, 'logit_bias': {}, 'max_tokens': 256}
```