# SambaNova

**[SambaNova](https://sambanova.ai/)** 的 [Sambaverse](https://sambaverse.sambanova.ai/) 和 [Sambastudio](https://sambanova.ai/technology/full-stack-ai-platform) 是用于运行您自己的开源模型的平台。

这个示例介绍了如何使用 LangChain 与 SambaNova 模型进行交互。

## Sambaverse

**Sambaverse** 允许您与多个开源模型进行交互。您可以查看可用模型的列表，并在 [playground](https://sambaverse.sambanova.ai/playground) 中与它们进行交互。**请注意，Sambaverse 的免费版本性能受限。** 准备评估 SambaNova 的每秒令牌性能、吞吐量以及总体拥有成本降低 10 倍的公司应该通过 [联系我们](https://sambaverse.sambanova.ai/contact-us) 获取一个无限制的评估实例。

访问 Sambaverse 模型需要 API 密钥。要获取密钥，请在 [sambaverse.sambanova.ai](https://sambaverse.sambanova.ai/) 创建一个帐户。

运行流式预测需要 [sseclient-py](https://pypi.org/project/sseclient-py/) 包。

```python
%pip install --quiet sseclient-py==1.8.0
```

将您的 API 密钥注册为环境变量：

```python
import os
sambaverse_api_key = "<您的 Sambaverse API 密钥>"
# 设置环境变量
os.environ["SAMBAVERSE_API_KEY"] = sambaverse_api_key
```

直接从 LangChain 调用 Sambaverse 模型！

```python
from langchain_community.llms.sambanova import Sambaverse
llm = Sambaverse(
    sambaverse_model_name="Meta/llama-2-7b-chat-hf",
    streaming=False,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        "process_prompt": True,
        "select_expert": "llama-2-7b-chat-hf",
        # "stop_sequences": '\"sequence1\",\"sequence2\"',
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_p": 1.0
    },
)
print(llm.invoke("为什么要使用开源模型？"))
```

```python
# 流式响应
from langchain_community.llms.sambanova import Sambaverse
llm = Sambaverse(
    sambaverse_model_name="Meta/llama-2-7b-chat-hf",
    streaming=True,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        "process_prompt": True,
        "select_expert": "llama-2-7b-chat-hf",
        # "stop_sequences": '\"sequence1\",\"sequence2\"',
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_p": 1.0
    },
)
for chunk in llm.stream("为什么要使用开源模型？"):
    print(chunk, end="", flush=True)
```

## SambaStudio

**SambaStudio** 允许您训练、运行批量推断作业，并部署在线推断端点以运行您自己微调的开源模型。

部署模型需要 SambaStudio 环境。获取更多信息，请访问 [sambanova.ai/products/enterprise-ai-platform-sambanova-suite](https://sambanova.ai/products/enterprise-ai-platform-sambanova-suite)。

运行流式预测需要 [sseclient-py](https://pypi.org/project/sseclient-py/) 包。

```python
%pip install --quiet sseclient-py==1.8.0
```

注册您的环境变量：

```python
import os
sambastudio_base_url = "<您的 SambaStudio 环境 URL>"
# sambastudio_base_uri = "<您的 SambaStudio 端点基本 URI>"  # 可选，默认设置为 "api/predict/nlp"
sambastudio_project_id = "<您的 SambaStudio 项目 ID>"
sambastudio_endpoint_id = "<您的 SambaStudio 端点 ID>"
sambastudio_api_key = "<您的 SambaStudio 端点 API 密钥>"
# 设置环境变量
os.environ["SAMBASTUDIO_BASE_URL"] = sambastudio_base_url
# os.environ["SAMBASTUDIO_BASE_URI"] = sambastudio_base_uri
os.environ["SAMBASTUDIO_PROJECT_ID"] = sambastudio_project_id
os.environ["SAMBASTUDIO_ENDPOINT_ID"] = sambastudio_endpoint_id
os.environ["SAMBASTUDIO_API_KEY"] = sambastudio_api_key
```

直接从 LangChain 调用 SambaStudio 模型！

```python
from langchain_community.llms.sambanova import SambaStudio
llm = SambaStudio(
    streaming=False,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_logprobs": 0,
        # "top_p": 1.0
    },
)
print(llm.invoke("为什么要使用开源模型？"))
```

```python
# 流式响应
from langchain_community.llms.sambanova import SambaStudio
llm = SambaStudio(
    streaming=True,
    model_kwargs={
        "do_sample": True,
        "max_tokens_to_generate": 1000,
        "temperature": 0.01,
        # "repetition_penalty":  1.0,
        # "top_k": 50,
        # "top_logprobs": 0,
        # "top_p": 1.0
    },
)
for chunk in llm.stream("为什么要使用开源模型？"):
    print(chunk, end="", flush=True)
```