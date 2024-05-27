# Clarifai

[Clarifai](https://www.clarifai.com/) 是一个 AI 平台，提供从数据探索、数据标注、模型训练、评估到推断的完整 AI 生命周期。

以下示例介绍了如何使用 LangChain 与 `Clarifai` [模型](https://clarifai.com/explore/models) 进行交互。特别是可以在[这里](https://clarifai.com/explore/models?page=1&perPage=24&filterData=%5B%7B%22field%22%3A%22model_type_id%22%2C%22value%22%3A%5B%22text-embedder%22%5D%7D%5D)找到文本嵌入模型。

要使用 Clarifai，您必须拥有一个账户和个人访问令牌（PAT）密钥。请在[这里](https://clarifai.com/settings/security)获取或创建 PAT。

# 依赖

```python
# 安装所需的依赖
%pip install --upgrade --quiet  clarifai
```

# 导入

在这里，我们将设置个人访问令牌。您可以在 Clarifai 账户的[设置/安全](https://clarifai.com/settings/security)下找到您的 PAT。

```python
# 请登录并从 https://clarifai.com/settings/security 获取您的 API 密钥
from getpass import getpass
CLARIFAI_PAT = getpass()
```

```python
# 导入所需的模块
from langchain.chains import LLMChain
from langchain_community.embeddings import ClarifaiEmbeddings
from langchain_core.prompts import PromptTemplate
```

# 输入

创建一个用于 LLM Chain 的提示模板：

```python
template = """Question: {question}
Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)
```

# 设置

将用户 ID 和应用程序 ID 设置为模型所在的应用程序。您可以在 https://clarifai.com/explore/models 上找到公共模型的列表。

您还必须初始化模型 ID，如果需要，还可以初始化模型版本 ID。一些模型有多个版本，您可以选择适合您任务的版本。

```python
USER_ID = "clarifai"
APP_ID = "main"
MODEL_ID = "BAAI-bge-base-en-v15"
MODEL_URL = "https://clarifai.com/clarifai/main/models/BAAI-bge-base-en-v15"
# 此外，您还可以提供特定的模型版本作为 model_version_id 参数。
# MODEL_VERSION_ID = "MODEL_VERSION_ID"
```

```python
# 初始化 Clarifai 嵌入模型
embeddings = ClarifaiEmbeddings(user_id=USER_ID, app_id=APP_ID, model_id=MODEL_ID)
# 使用模型 URL 初始化 Clarifai 嵌入模型
embeddings = ClarifaiEmbeddings(model_url=MODEL_URL)
# 或者您还可以使用 pat 参数初始化 clarifai 类。
```

```python
text = "roses are red violets are blue."
text2 = "Make hay while the sun shines."
```

您可以使用 embed_query 函数嵌入您的文本的单行！

```python
query_result = embeddings.embed_query(text)
```

此外，要嵌入文本/文档列表，请使用 embed_documents 函数。

```python
doc_result = embeddings.embed_documents([text, text2])
```