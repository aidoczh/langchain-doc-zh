# Dappier AI

**Dappier：利用动态实时数据模型赋能人工智能**

Dappier提供了一流的平台，让开发者立即获得涵盖新闻、娱乐、金融、市场数据、天气等各种实时数据模型。借助我们预训练的数据模型，您可以为自己的人工智能应用注入动力，确保其提供精准、及时的回应，最大程度地减少不准确性。

Dappier数据模型帮助您构建下一代LLM应用，提供来自世界领先品牌的可信、最新内容。通过简单的API，为任何GPT应用或人工智能工作流程增添可操作的专有数据，释放您的创造力。利用来自可信来源的专有数据增强您的人工智能，是确保无论提问内容如何，都能提供事实准确、最新回应，并减少幻觉的最佳途径。

为开发者而设计

Dappier专为开发者打造，简化了从数据集成到变现的过程，提供清晰、直接的路径，让您的人工智能模型得以部署并获利。体验新互联网的变现基础设施的未来，尽在**https://dappier.com/**。

以下示例介绍了如何使用LangChain与Dappier AI模型进行交互

-----------------------------------------------------------------------------------

要使用我们的Dappier AI数据模型之一，您需要一个API密钥。请访问Dappier平台（https://platform.dappier.com/）登录并在您的个人资料中创建API密钥。

您可以在API参考中找到更多详细信息：https://docs.dappier.com/introduction

要使用我们的Dappier Chat模型，您可以在初始化类时直接通过名为dappier_api_key的参数传递密钥，或者设置为环境变量。

```bash
export DAPPIER_API_KEY="..."
```

```python
from langchain_community.chat_models.dappier import ChatDappierAI
from langchain_core.messages import HumanMessage
```

```python
chat = ChatDappierAI(
    dappier_endpoint="https://api.dappier.com/app/datamodelconversation",
    dappier_model="dm_01hpsxyfm2fwdt2zet9cg6fdxt",
    dappier_api_key="...",
)
```

```python
messages = [HumanMessage(content="2024年超级碗的冠军是谁？")]
chat.invoke(messages)
```

```output
AIMessage(content='嘿！2024年的超级碗冠军是堪萨斯城酋长队。他们在延长赛中以25-22的最终比分击败旧金山49人队。那是一场相当精彩的比赛！🏈')
```

```python
await chat.ainvoke(messages)
```

```output
AIMessage(content='2024年的超级碗冠军是堪萨斯城酋长队！🏈')
```