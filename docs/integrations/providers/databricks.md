

Databricks

==========

[Databricks](https://www.databricks.com/) Lakehouse 平台将数据、分析和人工智能统一到一个平台上。

Databricks 在各种方式上支持 LangChain 生态系统：

1. SQLDatabase Chain 的 Databricks 连接器：SQLDatabase.from_databricks() 提供了通过 LangChain 查询 Databricks 数据的简便方法。

2. Databricks MLflow 与 LangChain 集成：跟踪和提供 LangChain 应用程序的 LangChain 集成更简单。

3. Databricks 作为 LLM 提供商：通过服务端点或集群驱动代理应用在 Databricks 上部署经过优化的 LLMs，并通过 langchain.llms.Databricks 进行查询。

4. Databricks Dolly：Databricks 开源了 Dolly，可用于商业用途，并可通过 Hugging Face Hub 访问。

Databricks connector for the SQLDatabase Chain

----------------------------------------------

您可以使用 LangChain 的 SQLDatabase 封装器连接到 [Databricks runtimes](https://docs.databricks.com/runtime/index.html) 和 [Databricks SQL](https://www.databricks.com/product/databricks-sql)。

Databricks MLflow 与 LangChain 集成

-------------------------------------------

MLflow 是一个开源平台，用于管理机器学习生命周期，包括实验、可重现性、部署和中央模型注册。有关 MLflow 与 LangChain 集成的详细信息，请参阅笔记本 [MLflow Callback Handler](/docs/integrations/providers/mlflow_tracking)。

Databricks 提供了完全托管和托管版本的 MLflow，集成了企业安全功能、高可用性以及其他 Databricks 工作区功能，如实验和运行管理以及笔记本修订捕获。Databricks 上的 MLflow 提供了一个集成体验，用于跟踪和保护机器学习模型训练运行和运行机器学习项目。有关更多详细信息，请参阅 [MLflow 指南](https://docs.databricks.com/mlflow/index.html)。

Databricks MLflow 使在 Databricks 上开发 LangChain 应用程序更加便捷。对于 MLflow 跟踪，您无需设置跟踪 uri。对于 MLflow 模型服务，您可以将 LangChain Chains 保存在 MLflow langchain flavor 中，然后在 Databricks 上注册和提供 Chain，几次点击即可完成，凭据由 MLflow 模型服务安全管理。

Databricks 外部模型

--------------------------

[Databricks 外部模型](https://docs.databricks.com/generative-ai/external-models/index.html) 是一个旨在简化组织内各种大型语言模型（LLM）提供商（如 OpenAI 和 Anthropic）使用和管理的服务。它提供了一个高级接口，通过提供一个统一的端点来处理特定的 LLM 相关请求，从而简化与这些服务的交互。以下示例创建一个端点，用于提供 OpenAI 的 GPT-4 模型并生成聊天响应：

```python
from langchain_community.chat_models import ChatDatabricks
from langchain_core.messages import HumanMessage
from mlflow.deployments import get_deploy_client
client = get_deploy_client("databricks")
name = f"chat"
client.create_endpoint(
    name=name,
    config={
        "served_entities": [
            {
                "name": "test",
                "external_model": {
                    "name": "gpt-4",
                    "provider": "openai",
                    "task": "llm/v1/chat",
                    "openai_config": {
                        "openai_api_key": "{{secrets/<scope>/<key>}}",
                    },
                },
            }
        ],
    },
)
chat = ChatDatabricks(endpoint=name, temperature=0.1)
print(chat([HumanMessage(content="hello")]))
# -> content='Hello! How can I assist you today?'
```

Databricks 基础模型 API

--------------------------------

[Databricks 基础模型 API](https://docs.databricks.com/machine-learning/foundation-models/index.html) 允许您从专用服务端点访问和查询最先进的开源模型。借助基础模型 API，开发人员可以快速轻松地构建利用高质量生成式 AI 模型的应用程序，而无需维护自己的模型部署。以下示例使用 `databricks-bge-large-en` 端点从文本生成嵌入：

```python
from langchain_community.embeddings import DatabricksEmbeddings
embeddings = DatabricksEmbeddings(endpoint="databricks-bge-large-en")
print(embeddings.embed_query("hello")[:3])
# -> [0.051055908203125, 0.007221221923828125, 0.003879547119140625, ...]
```

Databricks 作为 LLM 提供商

-----------------------------

笔记本 [Wrap Databricks endpoints as LLMs](/docs/integrations/llms/databricks#wrapping-a-serving-endpoint-custom-model) 演示了如何将由 MLflow 注册为 Databricks 端点的自定义模型提供服务。

它支持两种类型的端点：建议在生产和开发中都使用的服务端点，以及建议在交互式开发中使用的集群驱动代理应用程序端点。

Databricks 矢量搜索

------------------------

Databricks 矢量搜索是一种无服务器相似性搜索引擎，允许您将数据的矢量表示（包括元数据）存储在矢量数据库中。使用矢量搜索，您可以从由 Unity Catalog 管理的 Delta 表中创建自动更新的矢量搜索索引，并使用简单的 API 进行查询，以返回最相似的矢量。请参阅笔记本 [Databricks 矢量搜索](/docs/integrations/vectorstores/databricks_vector_search) 了解如何在 LangChain 中使用它的说明。