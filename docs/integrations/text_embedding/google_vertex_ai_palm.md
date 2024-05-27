# 谷歌 Vertex AI PaLM

[Vertex AI PaLM API](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/overview) 是谷歌云上的一个服务，提供了嵌入模型。

注意：此集成与谷歌 PaLM 集成是分开的。

默认情况下，谷歌云[不使用](https://cloud.google.com/vertex-ai/docs/generative-ai/data-governance#foundation_model_development)客户数据来训练其基础模型，这是谷歌云 AI/ML 隐私承诺的一部分。关于谷歌如何处理数据的更多细节也可以在[谷歌的客户数据处理附录 (CDPA)](https://cloud.google.com/terms/data-processing-addendum)中找到。

要使用 Vertex AI PaLM，您必须安装 `langchain-google-vertexai` Python 包，并且要么：

- 配置了您环境的凭据（gcloud，工作负载身份验证等...）

- 将服务帐号 JSON 文件的路径存储为 GOOGLE_APPLICATION_CREDENTIALS 环境变量

此代码库使用 `google.auth` 库，该库首先查找上述应用凭据变量，然后查找系统级别的身份验证。

更多信息，请参阅：

- https://cloud.google.com/docs/authentication/application-default-credentials#GAC

- https://googleapis.dev/python/google-auth/latest/reference/google.auth.html#module-google.auth

```python
%pip install --upgrade --quiet langchain langchain-google-vertexai
```

```python
from langchain_google_vertexai import VertexAIEmbeddings
```

```python
embeddings = VertexAIEmbeddings()
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