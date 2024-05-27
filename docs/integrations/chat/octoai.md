# ChatOctoAI

[OctoAI](https://docs.octoai.cloud/docs) 提供了便捷的计算资源，并允许用户将自己选择的 AI 模型集成到应用程序中。`OctoAI` 计算服务可以帮助您轻松运行、调优和扩展 AI 应用程序。

本文档演示了如何使用 `langchain.chat_models.ChatOctoAI` 来访问 [OctoAI endpoints](https://octoai.cloud/text)。

## 设置

要运行我们的示例应用程序，只需执行两个简单的步骤：

1. 从 [OctoAI 账户页面](https://octoai.cloud/settings) 获取 API 令牌。

   

2. 将您的 API 令牌粘贴到下面的代码单元格中，或使用 `octoai_api_token` 关键字参数。

注意：如果您想使用不同于 [可用模型](https://octoai.cloud/text?selectedTags=Chat) 的模型，您可以将模型容器化，并自己创建一个自定义的 OctoAI 端点，方法是按照 [从 Python 构建容器](https://octo.ai/docs/bring-your-own-model/advanced-build-a-container-from-scratch-in-python) 和 [从容器创建自定义端点](https://octo.ai/docs/bring-your-own-model/create-custom-endpoints-from-a-container/create-custom-endpoints-from-a-container) 的步骤，并更新您的 `OCTOAI_API_BASE` 环境变量。

```python
import os
os.environ["OCTOAI_API_TOKEN"] = "OCTOAI_API_TOKEN"
```

```python
from langchain_community.chat_models import ChatOctoAI
from langchain_core.messages import HumanMessage, SystemMessage
```

## 示例

```python
chat = ChatOctoAI(max_tokens=300, model_name="mixtral-8x7b-instruct")
```

```python
messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="Tell me about Leonardo da Vinci briefly."),
]
print(chat(messages).content)
```

莱昂纳多·达·芬奇（1452-1519）是一位意大利多才多艺的人，他常常被认为是历史上最伟大的画家之一。然而，他的天才远不止于艺术。他还是一位科学家、发明家、数学家、工程师、解剖学家、地质学家和制图师。

达·芬奇以他的绘画作品而闻名，如《蒙娜丽莎》、《最后的晚餐》和《岩石上的圣母》。他的科学研究超前于他的时代，他的笔记本中包含了各种机器、人体解剖和自然现象的详细绘图和描述。

尽管没有接受过正规教育，但达·芬奇的求知欲和观察力使他成为许多领域的先驱。他的作品至今仍然激励和影响着艺术家、科学家和思想家。