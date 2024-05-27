# Predibase

学习如何在 Predibase 上使用 LangChain 模型。

## 设置

- 创建一个 [Predibase](https://predibase.com/) 账户和 [API 密钥](https://docs.predibase.com/sdk-guide/intro)。

- 使用 `pip install predibase` 安装 Predibase Python 客户端。

- 使用 API 密钥进行身份验证。

### LLM

Predibase 通过实现 LLM 模块与 LangChain 集成。您可以在下面看到一个简短的示例，或者在 LLM > 集成 > Predibase 下找到完整的笔记本。

```python
import os
os.environ["PREDIBASE_API_TOKEN"] = "{PREDIBASE_API_TOKEN}"
from langchain_community.llms import Predibase
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
)
response = model.invoke("Can you recommend me a nice dry wine?")
print(response)
```

Predibase 还支持在给定 `model` 参数的基础模型上进行微调的 Predibase 托管和 HuggingFace 托管适配器：

```python
import os
os.environ["PREDIBASE_API_TOKEN"] = "{PREDIBASE_API_TOKEN}"
from langchain_community.llms import Predibase
# 经过微调的适配器托管在 Predibase 上（必须指定 adapter_version）。
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
    adapter_id="e2e_nlg",
    adapter_version=1,
)
response = model.invoke("Can you recommend me a nice dry wine?")
print(response)
```

Predibase 还支持在给定 `model` 参数的基础模型上进行微调的适配器：

```python
import os
os.environ["PREDIBASE_API_TOKEN"] = "{PREDIBASE_API_TOKEN}"
from langchain_community.llms import Predibase
# 经过微调的适配器托管在 HuggingFace 上（adapter_version 不适用，将被忽略）。
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
    adapter_id="predibase/e2e_nlg",
)
response = model.invoke("Can you recommend me a nice dry wine?")
print(response)
```