# OpenAI

[OpenAI](https://platform.openai.com/docs/introduction) 提供了一系列不同功率级别的模型，适用于不同的任务。

这个示例介绍了如何使用 LangChain 与 `OpenAI` [模型](https://platform.openai.com/docs/models) 进行交互。

```python
# 获取一个令牌：https://platform.openai.com/account/api-keys
from getpass import getpass
OPENAI_API_KEY = getpass()
```

```python
import os
os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

如果您需要指定组织 ID，可以使用以下单元格。但是，如果您只是属于一个组织或打算使用默认组织，则不需要。您可以在[这里](https://platform.openai.com/account/api-keys)检查您的默认组织。

要指定您的组织，可以使用以下内容：

```python
OPENAI_ORGANIZATION = getpass()
os.environ["OPENAI_ORGANIZATION"] = OPENAI_ORGANIZATION
```

```python
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
```

```python
template = """问题：{question}
回答：让我们一步一步来思考。"""
prompt = PromptTemplate.from_template(template)
```

```python
llm = OpenAI()
```

如果您想手动指定您的 OpenAI API 密钥和/或组织 ID，可以使用以下内容：

```python
llm = OpenAI(openai_api_key="YOUR_API_KEY", openai_organization="YOUR_ORGANIZATION_ID")
```

如果不适用于您，请删除 openai_organization 参数。

```python
llm_chain = prompt | llm
```

```python
question = "贾斯汀·比伯出生年份的超级碗冠军是哪支 NFL 球队？"
llm_chain.invoke(question)
```

```output
'贾斯汀·比伯出生于1994年3月1日。超级碗通常在一月下旬或二月初举行。因此，我们需要查看1994年的超级碗。1994年的超级碗是第二十八届超级碗，于1994年1月30日举行。那届超级碗的冠军是达拉斯牛仔队。'
```

如果您在使用显式代理，请指定 http_client 以进行传递。

```python
pip install httpx
import httpx
openai = OpenAI(model_name="gpt-3.5-turbo-instruct", http_client=httpx.Client(proxies="http://proxy.yourcompany.com:8080"))
```