# YandexGPT

本文介绍如何在[YandexGPT](https://cloud.yandex.com/en/services/yandexgpt)中使用Langchain。

要使用，您应该已安装`yandexcloud` Python包。

```python
%pip install --upgrade --quiet  yandexcloud
```

首先，您应该[创建服务帐户](https://cloud.yandex.com/en/docs/iam/operations/sa/create)，并赋予`ai.languageModels.user`角色。

接下来，您有两种身份验证选项：

- [IAM 令牌](https://cloud.yandex.com/en/docs/iam/operations/iam-token/create-for-sa)。

    您可以在构造函数参数`iam_token`中指定令牌，或者在环境变量`YC_IAM_TOKEN`中指定。

- [API 密钥](https://cloud.yandex.com/en/docs/iam/operations/api-key/create)。

    您可以在构造函数参数`api_key`中指定密钥，或者在环境变量`YC_API_KEY`中指定。

要指定模型，您可以使用`model_uri`参数，详细信息请参阅[文档](https://cloud.yandex.com/en/docs/yandexgpt/concepts/models#yandexgpt-generation)。

默认情况下，使用指定在参数`folder_id`或环境变量`YC_FOLDER_ID`中的文件夹中的最新版本的`yandexgpt-lite`。

```python
from langchain.chains import LLMChain
from langchain_community.llms import YandexGPT
from langchain_core.prompts import PromptTemplate
```

```python
template = "What is the capital of {country}?"
prompt = PromptTemplate.from_template(template)
```

```python
llm = YandexGPT()
```

```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

```python
country = "Russia"
llm_chain.invoke(country)
```

```output
'The capital of Russia is Moscow.'
```