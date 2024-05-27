

# Azure 机器学习

[Azure 机器学习](https://azure.microsoft.com/en-us/products/machine-learning/) 是一个用于构建、训练和部署机器学习模型的平台。用户可以在模型目录中探索要部署的模型类型，该目录提供来自不同提供商的基础和通用模型。

本笔记介绍了如何使用托管在 `Azure 机器学习在线端点` 上的 LLM。

```python
from langchain_community.llms.azureml_endpoint import AzureMLOnlineEndpoint
```

## 设置

您必须在[Azure 机器学习上部署模型](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-use-foundation-models?view=azureml-api-2#deploying-foundation-models-to-endpoints-for-inferencing)或者在[Azure AI studio上部署模型](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/deploy-models-open)，并获取以下参数：

* `endpoint_url`：端点提供的 REST 端点 URL。

* `endpoint_api_type`：在部署模型到**专用端点**（托管托管基础设施）时使用 `endpoint_type='dedicated'`。在使用**按使用量付费**提供的服务时使用 `endpoint_type='serverless'`。

* `endpoint_api_key`：端点提供的 API 密钥。

* `deployment_name`：（可选）使用端点的模型的部署名称。

## 内容格式化器

`content_formatter` 参数是一个处理程序类，用于转换 AzureML 端点的请求和响应，以匹配所需的模式。由于模型目录中有各种模型，每个模型可能会以不同方式处理数据，因此提供了一个 `ContentFormatterBase` 类，允许用户根据自己的喜好转换数据。以下内容格式化器已提供：

* `GPT2ContentFormatter`：为 GPT2 格式化请求和响应数据

* `DollyContentFormatter`：为 Dolly-v2 格式化请求和响应数据

* `HFContentFormatter`：为文本生成 Hugging Face 模型格式化请求和响应数据

* `CustomOpenAIContentFormatter`：为遵循 OpenAI API 兼容方案的模型（如 LLaMa2）格式化请求和响应数据。

*注意：`OSSContentFormatter` 正在被弃用，并替换为 `GPT2ContentFormatter`。逻辑相同，但 `GPT2ContentFormatter` 是一个更合适的名称。您仍然可以继续使用 `OSSContentFormatter`，因为更改是向后兼容的。*

## 示例

### 示例：实时端点上的 LlaMa 2 完成

```python
from langchain_community.llms.azureml_endpoint import (
    AzureMLEndpointApiType,
    CustomOpenAIContentFormatter,
)
from langchain_core.messages import HumanMessage
llm = AzureMLOnlineEndpoint(
    endpoint_url="https://<your-endpoint>.<your_region>.inference.ml.azure.com/score",
    endpoint_api_type=AzureMLEndpointApiType.dedicated,
    endpoint_api_key="my-api-key",
    content_formatter=CustomOpenAIContentFormatter(),
    model_kwargs={"temperature": 0.8, "max_new_tokens": 400},
)
response = llm.invoke("Write me a song about sparkling water:")
response
```

在调用时也可以指定模型参数：

```python
response = llm.invoke("Write me a song about sparkling water:", temperature=0.5)
response
```

### 示例：使用按使用量付费部署的聊天完成

```python
from langchain_community.llms.azureml_endpoint import (
    AzureMLEndpointApiType,
    CustomOpenAIContentFormatter,
)
from langchain_core.messages import HumanMessage
llm = AzureMLOnlineEndpoint(
    endpoint_url="https://<your-endpoint>.<your_region>.inference.ml.azure.com/v1/completions",
    endpoint_api_type=AzureMLEndpointApiType.serverless,
    endpoint_api_key="my-api-key",
    content_formatter=CustomOpenAIContentFormatter(),
    model_kwargs={"temperature": 0.8, "max_new_tokens": 400},
)
response = llm.invoke("Write me a song about sparkling water:")
response
```

### 示例：自定义内容格式化器

以下是使用 Hugging Face 摘要模型的示例。

```python
import json
import os
from typing import Dict
from langchain_community.llms.azureml_endpoint import (
    AzureMLOnlineEndpoint,
    ContentFormatterBase,
)
class CustomFormatter(ContentFormatterBase):
    content_type = "application/json"
    accepts = "application/json"
    def format_request_payload(self, prompt: str, model_kwargs: Dict) -> bytes:
        input_str = json.dumps(
            {
                "inputs": [prompt],
                "parameters": model_kwargs,
                "options": {"use_cache": False, "wait_for_model": True},
            }
        )
        return str.encode(input_str)
    def format_response_payload(self, output: bytes) -> str:
        response_json = json.loads(output)
        return response_json[0]["summary_text"]
content_formatter = CustomFormatter()
llm = AzureMLOnlineEndpoint(
    endpoint_api_type="dedicated",
    endpoint_api_key=os.getenv("BART_ENDPOINT_API_KEY"),
    endpoint_url=os.getenv("BART_ENDPOINT_URL"),
    model_kwargs={"temperature": 0.8, "max_new_tokens": 400},
    content_formatter=content_formatter,
)
large_text = """2020年1月7日，Blockberry Creative 宣布 HaSeul 将因心理健康问题不参与 Loona 下一张专辑的宣传。据说她被诊断出现“间歇性焦虑症状”，并将花时间专注于健康。2020年2月5日，Loona 发行了第二张 EP，标题为 [#]（读作 hash），并附带主打曲《So What》。尽管 HaSeul 没有出现在主打曲中，但她的声音出现在专辑中的其他三首歌曲中，包括《365》。EP 曾在 Gaon Retail Album Chart 榜单上排名第 1，然后在 Gaon Album Chart 榜单上以第 2 名首次亮相。2020年3月12日，Loona 在 Mnet 的 M Countdown 节目中凭借《So What》赢得了他们的第一个音乐节目奖杯。
2020年10月19日，Loona 发行了第三张 EP，标题为 [12:00]（读作 midnight），并附带首支单曲《Why Not?》。HaSeul 再次没有参与专辑制作，出于自己的决定专注于康复。EP 随后成为他们首张进入 Billboard 200 榜单的专辑，首次排名第 112。11月18日，Loona 发布了《Star》的音乐视频，这是 [12:00] 中的另一首歌曲。《Star》最高排名第 40，在 Billboard Mainstream Top 40 榜单上是 Loona 的首次亮相，使他们成为第二支进入该榜单的韩国女子组合。
2021年6月1日，Loona 宣布他们将于6月28日回归，推出第四张 EP，标题为 [&]（读作 and）。次日，即6月2日，Loona 官方社交媒体账号发布了一则预告片，展示了十二双眼睛，确认了自 2020 年初休假以来一直处于休假状态的成员 HaSeul 的回归。6月12日，成员 YeoJin、Kim Lip、Choerry 和 Go Won 发布了与 Cocomong 合作的歌曲《Yum-Yum》。9月8日，他们发布了另一首合作歌曲《Yummy-Yummy》。2021年6月27日，Loona 在特别片段的结尾宣布，他们将于9月15日在环球音乐日本子标签 EMI Records 旗下进行日本出道。8月27日，宣布 Loona 将于9月15日发布双 A 面单曲《Hula Hoop / Star Seed》，并于10月20日发行实体 CD 版。12月，Chuu 提起诉讼，暂停与 Blockberry Creative 的独家合同。"""
summarized_text = llm.invoke(large_text)
print(summarized_text)
```

### 示例：使用LLMChain的多莉（Dolly）

```python
from langchain.chains import LLMChain
from langchain_community.llms.azureml_endpoint import DollyContentFormatter
from langchain_core.prompts import PromptTemplate
formatter_template = "写一篇{word_count}字的关于{topic}的文章。"
prompt = PromptTemplate(
    input_variables=["word_count", "topic"], template=formatter_template
)
content_formatter = DollyContentFormatter()
llm = AzureMLOnlineEndpoint(
    endpoint_api_key=os.getenv("DOLLY_ENDPOINT_API_KEY"),
    endpoint_url=os.getenv("DOLLY_ENDPOINT_URL"),
    model_kwargs={"temperature": 0.8, "max_tokens": 300},
    content_formatter=content_formatter,
)
chain = LLMChain(llm=llm, prompt=prompt)
print(chain.invoke({"word_count": 100, "topic": "如何交朋友"}))
```

## 序列化LLM

您也可以保存和加载LLM配置

```python
from langchain_community.llms.loading import load_llm
save_llm = AzureMLOnlineEndpoint(
    deployment_name="databricks-dolly-v2-12b-4",
    model_kwargs={
        "temperature": 0.2,
        "max_tokens": 150,
        "top_p": 0.8,
        "frequency_penalty": 0.32,
        "presence_penalty": 72e-3,
    },
)
save_llm.save("azureml.json")
loaded_llm = load_llm("azureml.json")
print(loaded_llm)
```

```