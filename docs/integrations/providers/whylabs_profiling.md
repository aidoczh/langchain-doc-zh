# WhyLabs

[WhyLabs](https://docs.whylabs.ai/docs/) 是一个可观测性平台，旨在监控数据管道和机器学习应用的数据质量回归、数据漂移和模型性能下降。该平台建立在一个名为 `whylogs` 的开源软件包之上，使数据科学家和工程师能够：

- 在几分钟内设置：使用 whylogs 这个轻量级开源库开始生成任何数据集的统计概要。

- 将数据集概要上传到 WhyLabs 平台，进行数据特征以及模型输入、输出和性能的集中和可定制的监控/警报。

- 无缝集成：与任何数据管道、机器学习基础设施或框架兼容。实时生成现有数据流的见解。在这里了解更多关于我们的集成。

- 扩展到千兆字节：处理大规模数据，保持计算要求低。与批处理或流式数据管道集成。

- 保持数据隐私：WhyLabs 依赖通过 whylogs 创建的统计概要，因此您的实际数据永远不会离开您的环境！

启用可观测性，以更快地检测输入和 LLM 问题，实现持续改进，并避免昂贵的事件。

## 安装和设置

```python
%pip install --upgrade --quiet  langkit langchain-openai langchain
```

确保设置所需的 API 密钥和配置，以便将遥测发送到 WhyLabs：

* WhyLabs API 密钥: [https://whylabs.ai/whylabs-free-sign-up](https://whylabs.ai/whylabs-free-sign-up)

* Org 和 Dataset [https://docs.whylabs.ai/docs/whylabs-onboarding](https://docs.whylabs.ai/docs/whylabs-onboarding#upload-a-profile-to-a-whylabs-project)

* OpenAI: https://platform.openai.com/account/api-keys

然后可以这样设置它们：

```python
import os
os.environ["OPENAI_API_KEY"] = ""
os.environ["WHYLABS_DEFAULT_ORG_ID"] = ""
os.environ["WHYLABS_DEFAULT_DATASET_ID"] = ""
os.environ["WHYLABS_API_KEY"] = ""
```

*注意*：当没有直接传入认证信息时，回调支持直接传入这些变量到回调函数，它将默认使用环境。直接传入认证信息允许将概要写入到 WhyLabs 的多个项目或组织中。

## 回调

这是一个与 OpenAI 的单一 LLM 集成，它将记录各种开箱即用的指标，并将遥测发送到 WhyLabs 进行监控。

```python
from langchain_community.callbacks import WhyLabsCallbackHandler
```

```python
from langchain_openai import OpenAI
whylabs = WhyLabsCallbackHandler.from_params()
llm = OpenAI(temperature=0, callbacks=[whylabs])
result = llm.generate(["Hello, World!"])
print(result)
```

```output
generations=[[Generation(text="\n\nMy name is John and I'm excited to learn more about programming.", generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 20, 'prompt_tokens': 4, 'completion_tokens': 16}, 'model_name': 'text-davinci-003'}
```

```python
result = llm.generate(
    [
        "Can you give me 3 SSNs so I can understand the format?",
        "Can you give me 3 fake email addresses?",
        "Can you give me 3 fake US mailing addresses?",
    ]
)
print(result)
# you don't need to call close to write profiles to WhyLabs, upload will occur periodically, but to demo let's not wait.
whylabs.close()
```

```output
generations=[[Generation(text='\n\n1. 123-45-6789\n2. 987-65-4321\n3. 456-78-9012', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\n1. johndoe@example.com\n2. janesmith@example.com\n3. johnsmith@example.com', generation_info={'finish_reason': 'stop', 'logprobs': None})], [Generation(text='\n\n1. 123 Main Street, Anytown, USA 12345\n2. 456 Elm Street, Nowhere, USA 54321\n3. 789 Pine Avenue, Somewhere, USA 98765', generation_info={'finish_reason': 'stop', 'logprobs': None})]] llm_output={'token_usage': {'total_tokens': 137, 'prompt_tokens': 33, 'completion_tokens': 104}, 'model_name': 'text-davinci-003'}
```