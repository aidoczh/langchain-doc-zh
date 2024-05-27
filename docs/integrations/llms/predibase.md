# Predibase

[Predibase](https://predibase.com/) 允许您训练、微调和部署任何机器学习模型，从线性回归到大型语言模型。

这个示例演示了如何在 Predibase 上使用 Langchain 部署模型。

# 设置

要运行这个笔记本，您需要一个[Predibase 账户](https://predibase.com/free-trial/?utm_source=langchain)和一个[API 密钥](https://docs.predibase.com/sdk-guide/intro)。

您还需要安装 Predibase Python 包：

```python
%pip install --upgrade --quiet  predibase
import os
os.environ["PREDIBASE_API_TOKEN"] = "{PREDIBASE_API_TOKEN}"
```

## 初始调用

```python
from langchain_community.llms import Predibase
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
)
```

```python
from langchain_community.llms import Predibase
# 使用在 Predibase 上部署的微调适配器（必须指定 adapter_version）。
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
    adapter_id="e2e_nlg",
    adapter_version=1,
)
```

```python
from langchain_community.llms import Predibase
# 使用在 HuggingFace 上部署的微调适配器（adapter_version 不适用，将被忽略）。
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
    adapter_id="predibase/e2e_nlg",
)
```

```python
response = model.invoke("Can you recommend me a nice dry wine?")
print(response)
```

## 链式调用设置

```python
from langchain_community.llms import Predibase
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
)
```

```python
# 使用在 Predibase 上部署的微调适配器（必须指定 adapter_version）。
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
    adapter_id="e2e_nlg",
    adapter_version=1,
)
```

```python
# 使用在 HuggingFace 上部署的微调适配器（adapter_version 不适用，将被忽略）。
llm = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
    adapter_id="predibase/e2e_nlg",
)
```

## SequentialChain

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
```

```python
# 这是一个 LLMChain，根据剧名撰写剧情简介。
template = """You are a playwright. Given the title of play, it is your job to write a synopsis for that title.
Title: {title}
Playwright: This is a synopsis for the above play:"""
prompt_template = PromptTemplate(input_variables=["title"], template=template)
synopsis_chain = LLMChain(llm=llm, prompt=prompt_template)
```

```python
# 这是一个 LLMChain，根据剧情简介撰写一篇评论。
template = """You are a play critic from the New York Times. Given the synopsis of play, it is your job to write a review for that play.
Play Synopsis:
{synopsis}
Review from a New York Times play critic of the above play:"""
prompt_template = PromptTemplate(input_variables=["synopsis"], template=template)
review_chain = LLMChain(llm=llm, prompt=prompt_template)
```

```python
# 这是一个整体链条，我们按顺序运行这两个链条。
from langchain.chains import SimpleSequentialChain
overall_chain = SimpleSequentialChain(
    chains=[synopsis_chain, review_chain], verbose=True
)
```

```python
review = overall_chain.run("Tragedy at sunset on the beach")
```

## 微调的LLM（使用您自己在 Predibase 上微调的LLM）

```python
from langchain_community.llms import Predibase
model = Predibase(
    model="my-base-LLM",
    predibase_api_key=os.environ.get(
        "PREDIBASE_API_TOKEN"
    ),  # Adapter argument is optional.
    predibase_sdk_version=None,  # 可选参数（如果省略，默认为最新的 Predibase SDK 版本）
    adapter_id="my-finetuned-adapter-id",  # 支持 Predibase 托管和 HuggingFace 托管的适配器存储库。
    adapter_version=1,  # 对于 Predibase 托管的适配器是必需的（对于 HuggingFace 托管的适配器将被忽略）
)
# 将 my-base-LLM 替换为您在 Predibase 中选择的无服务器基础模型的名称
```

```python
# response = model.invoke("Can you help categorize the following emails into positive, negative, and neutral?")
```