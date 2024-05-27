# GigaChat

这篇笔记展示了如何在 [GigaChat](https://developers.sber.ru/portal/products/gigachat) 中使用 LangChain。要使用它，你需要安装 ```gigachat``` Python 包。

```python
%pip install --upgrade --quiet  gigachat
```

要获取 GigaChat 凭据，你需要[创建账户](https://developers.sber.ru/studio/login)并[获取 API 访问权限](https://developers.sber.ru/docs/ru/gigachat/individuals-quickstart)。

## 示例

```python
import os
from getpass import getpass
os.environ["GIGACHAT_CREDENTIALS"] = getpass()
```

```python
from langchain_community.llms import GigaChat
llm = GigaChat(verify_ssl_certs=False, scope="GIGACHAT_API_PERS")
```

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
template = "What is capital of {country}?"
prompt = PromptTemplate.from_template(template)
llm_chain = LLMChain(prompt=prompt, llm=llm)
generated = llm_chain.invoke(input={"country": "Russia"})
print(generated["text"])
```

```output
俄罗斯的首都是莫斯科。
```