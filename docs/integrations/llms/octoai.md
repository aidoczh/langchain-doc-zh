# OctoAI

[OctoAI](https://docs.octoai.cloud/docs) 提供了高效计算的便捷方式，使用户能够将自己选择的 AI 模型集成到应用程序中。`OctoAI` 计算服务可以帮助您轻松运行、调优和扩展 AI 应用程序。

本示例介绍如何使用 LangChain 与 `OctoAI` 的 [LLM 端点](https://octoai.cloud/templates) 进行交互。

## 设置

要运行我们的示例应用程序，只需执行两个简单的步骤：

1. 从 [您的 OctoAI 帐户页面](https://octoai.cloud/settings) 获取 API 令牌。

   

2. 将您的 API 密钥粘贴到下面的代码单元格中。

注意：如果您想使用不同的 LLM 模型，您可以将模型容器化，并自己创建一个自定义的 OctoAI 端点，方法是按照 [使用 Python 构建容器](https://octo.ai/docs/bring-your-own-model/advanced-build-a-container-from-scratch-in-python) 和 [从容器创建自定义端点](https://octo.ai/docs/bring-your-own-model/create-custom-endpoints-from-a-container/create-custom-endpoints-from-a-container) 进行操作，然后更新您的 `OCTOAI_API_BASE` 环境变量。

```python
import os
os.environ["OCTOAI_API_TOKEN"] = "OCTOAI_API_TOKEN"
```

```python
from langchain.chains import LLMChain
from langchain_community.llms.octoai_endpoint import OctoAIEndpoint
from langchain_core.prompts import PromptTemplate
```

## 示例

```python
template = """以下是描述一个任务的指令。请编写一个适当完成请求的回答。\n 指令:\n{question}\n 回答: """
prompt = PromptTemplate.from_template(template)
```

```python
llm = OctoAIEndpoint(
    model_name="llama-2-13b-chat-fp16",
    max_tokens=200,
    presence_penalty=0,
    temperature=0.1,
    top_p=0.9,
)
```

```python
question = "谁是莱昂纳多·达·芬奇？"
chain = prompt | llm
print(chain.invoke(question))
```

莱昂纳多·达·芬奇是一个真正的文艺复兴人物。他于1452年出生在意大利的文奇，并以在艺术、科学、工程和数学等领域的工作而闻名。他被认为是有史以来最伟大的画家之一，他最著名的作品包括《蒙娜丽莎》和《最后的晚餐》。除了他的艺术作品，达·芬奇还对工程学和解剖学做出了重要贡献，他的机器和发明设计超前于他所处的时代几个世纪。他还以他广泛的日记和绘画而闻名，这些作品提供了对他的思想和想法的宝贵见解。达·芬奇的遗产继续激励和影响着今天世界各地的艺术家、科学家和思想家。