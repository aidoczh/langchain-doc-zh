# IBM watsonx.ai

[WatsonxLLM](https://ibm.github.io/watsonx-ai-python-sdk/fm_extensions.html#langchain) 是 IBM [watsonx.ai](https://www.ibm.com/products/watsonx-ai) 基础模型的封装器。

这个示例展示了如何使用 `LangChain` 与 `watsonx.ai` 模型进行通信。

## 设置

安装 `langchain-ibm` 包。

```python
!pip install -qU langchain-ibm
```

这段代码定义了与 watsonx Foundation Model 推理工作所需的 WML 凭据。

**操作：** 提供 IBM Cloud 用户 API 密钥。详情请参阅[文档](https://cloud.ibm.com/docs/account?topic=account-userapikey&interface=ui)。

```python
import os
from getpass import getpass
watsonx_api_key = getpass()
os.environ["WATSONX_APIKEY"] = watsonx_api_key
```

此外，您可以将其他机密作为环境变量传递。

```python
import os
os.environ["WATSONX_URL"] = "your service instance url"
os.environ["WATSONX_TOKEN"] = "your token for accessing the CPD cluster"
os.environ["WATSONX_PASSWORD"] = "your password for accessing the CPD cluster"
os.environ["WATSONX_USERNAME"] = "your username for accessing the CPD cluster"
os.environ["WATSONX_INSTANCE_ID"] = "your instance_id for accessing the CPD cluster"
```

## 加载模型

您可能需要调整不同模型或任务的模型 `parameters`。详情请参阅[文档](https://ibm.github.io/watsonx-ai-python-sdk/fm_model.html#metanames.GenTextParamsMetaNames)。

使用先前设置的参数初始化 `WatsonxLLM` 类。

**注意：**

- 要为 API 调用提供上下文，必须添加 `project_id` 或 `space_id`。更多信息请参阅[文档](https://www.ibm.com/docs/en/watsonx-as-a-service?topic=projects)。

- 根据您配置的服务实例的区域，使用[这里](https://ibm.github.io/watsonx-ai-python-sdk/setup_cloud.html#authentication)描述的其中一个 URL。

在此示例中，我们将使用 `project_id` 和 Dallas URL。

您需要指定用于推理的 `model_id`。您可以在[文档](https://ibm.github.io/watsonx-ai-python-sdk/fm_model.html#ibm_watsonx_ai.foundation_models.utils.enums.ModelTypes)中找到所有可用的模型。

```python
from langchain_ibm import WatsonxLLM
watsonx_llm = WatsonxLLM(
    model_id="ibm/granite-13b-instruct-v2",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

或者，您可以使用 Cloud Pak for Data 凭据。详情请参阅[文档](https://ibm.github.io/watsonx-ai-python-sdk/setup_cpd.html)。

```python
watsonx_llm = WatsonxLLM(
    model_id="ibm/granite-13b-instruct-v2",
    url="PASTE YOUR URL HERE",
    username="PASTE YOUR USERNAME HERE",
    password="PASTE YOUR PASSWORD HERE",
    instance_id="openshift",
    version="4.8",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

您还可以传递先前调整模型的 `deployment_id` 而不是 `model_id`。整个模型调整工作流程在[这里](https://ibm.github.io/watsonx-ai-python-sdk/pt_working_with_class_and_prompt_tuner.html)有描述。

```python
watsonx_llm = WatsonxLLM(
    deployment_id="PASTE YOUR DEPLOYMENT_ID HERE",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

您还可以将 IBM 的 [`ModelInference`](https://ibm.github.io/watsonx-ai-python-sdk/fm_model_inference.html) 对象传递到 `WatsonxLLM` 类。

```python
from ibm_watsonx_ai.foundation_models import ModelInference
model = ModelInference(...)
watsonx_llm = WatsonxLLM(watsonx_model=model)
```

## 创建链

创建 `PromptTemplate` 对象，负责生成一个随机问题。

```python
from langchain_core.prompts import PromptTemplate
template = "Generate a random question about {topic}: Question: "
prompt = PromptTemplate.from_template(template)
```

提供一个主题并运行链。

```python
llm_chain = prompt | watsonx_llm
topic = "dog"
llm_chain.invoke(topic)
```

```output
'What is the difference between a dog and a wolf?'
```

## 直接调用模型

要获取完成的结果，您可以直接使用字符串提示调用模型。

```python
# 调用单个提示
watsonx_llm.invoke("Who is man's best friend?")
```

```output
"Man's best friend is his dog. "
```

```python
# 调用多个提示
watsonx_llm.generate(
    [
        "The fastest dog in the world?",
        "Describe your chosen dog breed",
    ]
)
```

```output
LLMResult(generations=[[Generation(text='The fastest dog in the world is the greyhound, which can run up to 45 miles per hour. This is about the same speed as a human running down a track. Greyhounds are very fast because they have long legs, a streamlined body, and a strong tail. They can run this fast for short distances, but they can also run for long distances, like a marathon. ', generation_info={'finish_reason': 'eos_token'})], [Generation(text='The Beagle is a scent hound, meaning it is bred to hunt by following a trail of scents.', generation_info={'finish_reason': 'eos_token'})]], llm_output={'token_usage': {'generated_token_count': 106, 'input_token_count': 13}, 'model_id': 'ibm/granite-13b-instruct-v2', 'deployment_id': ''}, run=[RunInfo(run_id=UUID('52cb421d-b63f-4c5f-9b04-d4770c664725')), RunInfo(run_id=UUID('df2ea606-1622-4ed7-8d5d-8f6e068b71c4'))])
```

## 流式传输模型输出

您可以流式传输模型输出。

```python
for chunk in watsonx_llm.stream(
    "描述一下你最喜欢的狗品种，以及为什么它是你最喜欢的。"
):
    print(chunk, end="")
```

```output
我最喜欢的狗品种是拉布拉多寻回犬。拉布拉多是我最喜欢的，因为它们非常聪明、友好，并且喜欢和人在一起。它们也非常好动，喜欢四处奔跑，充满活力。
```