# 花瓣

`Petals` 在家中以 BitTorrent 风格运行超过 100 亿个语言模型。

本笔记将介绍如何在 [Petals](https://github.com/bigscience-workshop/petals) 上使用 Langchain。

## 安装 petals

使用 Petals API 需要安装 `petals` 软件包。使用 `pip3 install petals` 命令来安装 `petals`。

对于使用 Apple Silicon（M1/M2）芯片的用户，请按照这个指南 [https://github.com/bigscience-workshop/petals/issues/147#issuecomment-1365379642](https://github.com/bigscience-workshop/petals/issues/147#issuecomment-1365379642) 来安装 petals。

```python
!pip3 install petals
```

## 导入库

```python
import os
from langchain.chains import LLMChain
from langchain_community.llms import Petals
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥

确保从 Huggingface 获取[您的 API 密钥](https://huggingface.co/docs/api-inference/quicktour#get-your-api-token)。

```python
from getpass import getpass
HUGGINGFACE_API_KEY = getpass()
```

```output
 ········
```

```python
os.environ["HUGGINGFACE_API_KEY"] = HUGGINGFACE_API_KEY
```

## 创建 Petals 实例

您可以指定不同的参数，如模型名称、最大新标记、温度等。

```python
# 这可能需要几分钟来下载大文件！
llm = Petals(model_name="bigscience/bloom-petals")
```

```output
Downloading:   1%|▏                        | 40.8M/7.19G [00:24<15:44, 7.57MB/s]
```

## 创建一个提示模板

我们将为问题和答案创建一个提示模板。

```python
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
```

## 初始化 LLMChain

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

## 运行 LLMChain

提供一个问题并运行 LLMChain。

```python
question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"
llm_chain.run(question)
```