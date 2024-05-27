# 谷歌生成式 AI 嵌入

使用 `GoogleGenerativeAIEmbeddings` 类连接到谷歌的生成式 AI 嵌入服务，该类位于 [langchain-google-genai](https://pypi.org/project/langchain-google-genai/) 包中。

## 安装

```python
%pip install --upgrade --quiet  langchain-google-genai
```

## 凭据

```python
import getpass
import os
if "GOOGLE_API_KEY" not in os.environ:
    os.environ["GOOGLE_API_KEY"] = getpass("在此处提供您的谷歌 API 密钥")
```

## 用法

```python
from langchain_google_genai import GoogleGenerativeAIEmbeddings
embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001")
vector = embeddings.embed_query("hello, world!")
vector[:5]
```

```output
[0.05636945, 0.0048285457, -0.0762591, -0.023642512, 0.05329321]
```

## 批处理

您还可以一次嵌入多个字符串以加快处理速度：

```python
vectors = embeddings.embed_documents(
    [
        "今天是星期一",
        "今天是星期二",
        "今天是愚人节",
    ]
)
len(vectors), len(vectors[0])
```

```output
(3, 768)
```

## 任务类型

`GoogleGenerativeAIEmbeddings` 可选择支持 `task_type`，目前必须是以下之一：

- task_type_unspecified

- retrieval_query

- retrieval_document

- semantic_similarity

- classification

- clustering

默认情况下，我们在 `embed_documents` 方法中使用 `retrieval_document`，在 `embed_query` 方法中使用 `retrieval_query`。如果您提供了任务类型，我们将对所有方法使用该任务类型。

```python
%pip install --upgrade --quiet  matplotlib scikit-learn
```

```output
注意：您可能需要重新启动内核以使用更新的软件包。
```

```python
query_embeddings = GoogleGenerativeAIEmbeddings(
    model="models/embedding-001", task_type="retrieval_query"
)
doc_embeddings = GoogleGenerativeAIEmbeddings(
    model="models/embedding-001", task_type="retrieval_document"
)
```

所有这些都将使用 'retrieval_query' 任务设置进行嵌入

```python
query_vecs = [query_embeddings.embed_query(q) for q in [query, query_2, answer_1]]
```

所有这些都将使用 'retrieval_document' 任务设置进行嵌入

```python
doc_vecs = [doc_embeddings.embed_query(q) for q in [query, query_2, answer_1]]
```

在检索中，相对距离很重要。在上面的图像中，您可以看到在“相关文档”和“相似查询”之间的相似性分数之间的差异，在后一种情况下，相似查询和相关文档之间的相似性得分之间的差异更大。

## 附加配置

您可以传递以下参数给 ChatGoogleGenerativeAI 以自定义 SDK 的行为：

- `client_options`：[客户端选项](https://googleapis.dev/python/google-api-core/latest/client_options.html#module-google.api_core.client_options) 传递给谷歌 API 客户端，例如自定义 `client_options["api_endpoint"]`

- `transport`：要使用的传输方法，例如 `rest`、`grpc` 或 `grpc_asyncio`。