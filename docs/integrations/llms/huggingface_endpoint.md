# Huggingface 终端

[Hugging Face Hub](https://huggingface.co/docs/hub/index) 是一个拥有超过 120,000 个模型、20,000 个数据集和 50,000 个演示应用程序（Spaces）的平台，所有内容均为开源且公开可用，人们可以在这个在线平台上轻松合作并共同构建机器学习。

`Hugging Face Hub` 还提供了各种终端来构建机器学习应用程序。以下示例展示了如何连接到不同类型的终端。

具体来说，文本生成推理由 [Text Generation Inference](https://github.com/huggingface/text-generation-inference) 提供支持：这是一个专为快速文本生成推理而构建的 Rust、Python 和 gRPC 服务器。

```python
from langchain_huggingface import HuggingFaceEndpoint
```

## 安装和设置

要使用，您应该已安装 ``huggingface_hub`` python [包](https://huggingface.co/docs/huggingface_hub/installation)。

```python
%pip install --upgrade --quiet huggingface_hub
```

```python
# 获取令牌：https://huggingface.co/docs/api-inference/quicktour#get-your-api-token
from getpass import getpass
HUGGINGFACEHUB_API_TOKEN = getpass()
```

```python
import os
os.environ["HUGGINGFACEHUB_API_TOKEN"] = HUGGINGFACEHUB_API_TOKEN
```

## 准备示例

```python
from langchain_huggingface import HuggingFaceEndpoint
```

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
```

```python
question = "1994 年谁赢得了 FIFA 世界杯？"
template = """问题：{question}
答案：让我们一步一步来思考。"""
prompt = PromptTemplate.from_template(template)
```

## 示例

以下是一个示例，展示了如何访问 `HuggingFaceEndpoint` 集成的免费 [无服务器终端](https://huggingface.co/inference-endpoints/serverless) API。

```python
repo_id = "mistralai/Mistral-7B-Instruct-v0.2"
llm = HuggingFaceEndpoint(
    repo_id=repo_id,
    max_length=128,
    temperature=0.5,
    huggingfacehub_api_token=HUGGINGFACEHUB_API_TOKEN,
)
llm_chain = prompt | llm
print(llm_chain.invoke({"question": question}))
```

## 专用终端

免费的无服务器 API 让您可以快速实现解决方案并进行迭代，但对于大型使用情况可能会有速率限制，因为负载是与其他请求共享的。

对于企业工作负载，最好使用 [推理终端 - 专用](https://huggingface.co/inference-endpoints/dedicated)。这将提供完全托管的基础设施，提供更多灵活性和速度。这些资源配备持续支持和正常运行时间保证，以及像自动缩放等选项。

```python
# 将您的推理终端 URL 设置为下方的值
your_endpoint_url = "https://fayjubiy2xqn36z0.us-east-1.aws.endpoints.huggingface.cloud"
```

```python
llm = HuggingFaceEndpoint(
    endpoint_url=f"{your_endpoint_url}",
    max_new_tokens=512,
    top_k=10,
    top_p=0.95,
    typical_p=0.95,
    temperature=0.01,
    repetition_penalty=1.03,
)
llm("foo 对 bar 说了什么？")
```

### 流式处理

```python
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_huggingface import HuggingFaceEndpoint
llm = HuggingFaceEndpoint(
    endpoint_url=f"{your_endpoint_url}",
    max_new_tokens=512,
    top_k=10,
    top_p=0.95,
    typical_p=0.95,
    temperature=0.01,
    repetition_penalty=1.03,
    streaming=True,
)
llm("foo 对 bar 说了什么？", callbacks=[StreamingStdOutCallbackHandler()])
```