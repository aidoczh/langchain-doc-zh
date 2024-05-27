# OpenLM

[OpenLM](https://github.com/r2d4/openlm) 是一个零依赖的 OpenAI 兼容的 LLM 提供者，可以通过 HTTP 直接调用不同的推理端点。

它实现了 OpenAI Completion 类，因此可以作为 OpenAI API 的即插即用替代品。这个变更集利用了 BaseOpenAI 来添加最少的代码。

这个示例演示了如何使用 LangChain 与 OpenAI 和 HuggingFace 进行交互。您需要从两者那里获取 API 密钥。

### 设置

安装依赖项并设置 API 密钥。

```python
# 如果尚未安装 openlm 和 openai，请取消注释以下代码
%pip install --upgrade --quiet  openlm
%pip install --upgrade --quiet  langchain-openai
```

```python
import os
from getpass import getpass
# 检查是否设置了 OPENAI_API_KEY 环境变量
if "OPENAI_API_KEY" not in os.environ:
    print("输入您的 OpenAI API 密钥：")
    os.environ["OPENAI_API_KEY"] = getpass()
# 检查是否设置了 HF_API_TOKEN 环境变量
if "HF_API_TOKEN" not in os.environ:
    print("输入您的 HuggingFace Hub API 密钥：")
    os.environ["HF_API_TOKEN"] = getpass()
```

### 使用 LangChain 与 OpenLM

这里我们将在一个 LLMChain 中调用两个模型，`text-davinci-003` 来自 OpenAI，`gpt2` 来自 HuggingFace。

```python
from langchain.chains import LLMChain
from langchain_community.llms import OpenLM
from langchain_core.prompts import PromptTemplate
```

```python
question = "法国的首都是哪里？"
template = """问题: {question}
回答: 让我们一步一步来思考。"""
prompt = PromptTemplate.from_template(template)
for model in ["text-davinci-003", "huggingface.co/gpt2"]:
    llm = OpenLM(model=model)
    llm_chain = LLMChain(prompt=prompt, llm=llm)
    result = llm_chain.run(question)
    print(
        """模型: {}
结果: {}""".format(model, result)
    )
```

```output
模型: text-davinci-003
结果: 法国是欧洲的一个国家。法国的首都是巴黎。
模型: huggingface.co/gpt2
结果: 问题: 法国的首都是哪里？
回答: 让我们一步一步来思考。我不会撒谎，这是一个复杂的问题，我看不到任何解决方案，但这还远远不够
```