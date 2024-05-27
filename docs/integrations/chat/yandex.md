---

sidebar_label: YandexGPT

---

# ChatYandexGPT

这篇笔记介绍了如何使用 [YandexGPT](https://cloud.yandex.com/en/services/yandexgpt) 的聊天模型。

要使用，您应该已经安装了 `yandexcloud` Python 包。

```python
%pip install --upgrade --quiet yandexcloud
```

首先，您应该使用 `ai.languageModels.user` 角色 [创建服务账号](https://cloud.yandex.com/en/docs/iam/operations/sa/create)。

接下来，您有两种身份验证选项：

- [IAM 令牌](https://cloud.yandex.com/en/docs/iam/operations/iam-token/create-for-sa)。

    您可以在构造函数参数 `iam_token` 中指定令牌，或者在环境变量 `YC_IAM_TOKEN` 中指定。

- [API 密钥](https://cloud.yandex.com/en/docs/iam/operations/api-key/create)。

    您可以在构造函数参数 `api_key` 中指定密钥，或者在环境变量 `YC_API_KEY` 中指定。

要指定模型，您可以使用 `model_uri` 参数，详细信息请参阅[文档](https://cloud.yandex.com/en/docs/yandexgpt/concepts/models#yandexgpt-generation)。

默认情况下，使用指定在参数 `folder_id` 或 `YC_FOLDER_ID` 环境变量中的文件夹中的 `yandexgpt-lite` 的最新版本。

```python
from langchain_community.chat_models import ChatYandexGPT
from langchain_core.messages import HumanMessage, SystemMessage
```

```python
chat_model = ChatYandexGPT()
```

```python
answer = chat_model.invoke(
    [
        SystemMessage(
            content="You are a helpful assistant that translates English to French."
        ),
        HumanMessage(content="I love programming."),
    ]
)
answer
```

```output
AIMessage(content='Je adore le programmement.')
```