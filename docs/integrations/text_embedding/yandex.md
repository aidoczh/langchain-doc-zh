# YandexGPT

本文介绍如何使用 Langchain 与 [YandexGPT](https://cloud.yandex.com/en/services/yandexgpt) 嵌入模型。

要使用，您应该已安装 `yandexcloud` Python 包。

```python
%pip install --upgrade --quiet  yandexcloud
```

首先，您应该使用 `ai.languageModels.user` 角色[创建服务帐户](https://cloud.yandex.com/en/docs/iam/operations/sa/create)。

接下来，您有两种身份验证选项：

- [IAM 令牌](https://cloud.yandex.com/en/docs/iam/operations/iam-token/create-for-sa)。

    您可以在构造函数参数 `iam_token` 或环境变量 `YC_IAM_TOKEN` 中指定令牌。

- [API 密钥](https://cloud.yandex.com/en/docs/iam/operations/api-key/create)。

    您可以在构造函数参数 `api_key` 或环境变量 `YC_API_KEY` 中指定密钥。

要指定模型，您可以使用 `model_uri` 参数，详细信息请参阅[文档](https://cloud.yandex.com/en/docs/yandexgpt/concepts/models#yandexgpt-embeddings)。

默认情况下，从参数 `folder_id` 或环境变量 `YC_FOLDER_ID` 指定的文件夹中使用 `text-search-query` 的最新版本。

```python
from langchain_community.embeddings.yandex import YandexGPTEmbeddings
```

```python
embeddings = YandexGPTEmbeddings()
```

```python
text = "This is a test document."
```

```python
query_result = embeddings.embed_query(text)
```

```python
doc_result = embeddings.embed_documents([text])
```

```python
query_result[:5]
```

```output
[-0.021392822265625,
 0.096435546875,
 -0.046966552734375,
 -0.0183258056640625,
 -0.00555419921875]
```

```python
doc_result[0][:5]
```

```output
[-0.021392822265625,
 0.096435546875,
 -0.046966552734375,
 -0.0183258056640625,
 -0.00555419921875]
```