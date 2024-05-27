# Azure OpenAI

让我们使用设置了指示使用 Azure 端点的环境变量来加载 Azure OpenAI Embedding 类。

```python
import os
os.environ["AZURE_OPENAI_API_KEY"] = "..."
os.environ["AZURE_OPENAI_ENDPOINT"] = "https://<your-endpoint>.openai.azure.com/"
```

```python
from langchain_openai import AzureOpenAIEmbeddings
embeddings = AzureOpenAIEmbeddings(
    azure_deployment="<your-embeddings-deployment-name>",
    openai_api_version="2023-05-15",
)
```

```python
text = "this is a test document"
```

```python
query_result = embeddings.embed_query(text)
```

```python
doc_result = embeddings.embed_documents([text])
```

```python
doc_result[0][:5]
```

```output
[-0.012222584727053133,
 0.0072103982392216145,
 -0.014818063280923775,
 -0.026444746872933557,
 -0.0034330499700826883]
```

## [Legacy] 当使用 `openai<1`

```python
# 设置 openai 包所需的环境变量，以便知道如何访问 Azure
import os
os.environ["OPENAI_API_TYPE"] = "azure"
os.environ["OPENAI_API_BASE"] = "https://<your-endpoint.openai.azure.com/"
os.environ["OPENAI_API_KEY"] = "your AzureOpenAI key"
os.environ["OPENAI_API_VERSION"] = "2023-05-15"
```

```python
from langchain_openai import OpenAIEmbeddings
embeddings = OpenAIEmbeddings(deployment="your-embeddings-deployment-name")
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