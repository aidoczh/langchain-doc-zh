# Hugging Face

让我们加载 Hugging Face 嵌入类。

```python
%pip install --upgrade --quiet  langchain sentence_transformers
```

```python
from langchain_huggingface.embeddings import HuggingFaceEmbeddings
```

```python
embeddings = HuggingFaceEmbeddings()
```

```python
text = "This is a test document."
```

```python
query_result = embeddings.embed_query(text)
```

```python
query_result[:3]
```

[-0.04895168915390968, -0.03986193612217903, -0.021562768146395683]

```python
doc_result = embeddings.embed_documents([text])
```

## Hugging Face 推理 API

我们还可以通过 Hugging Face 推理 API 访问嵌入模型，这不需要我们安装 ``sentence_transformers`` 并在本地下载模型。

```python
import getpass
inference_api_key = getpass.getpass("输入你的 HF 推理 API 密钥:\n\n")
```

```output
输入你的 HF 推理 API 密钥:
 ········
```

```python
from langchain_community.embeddings import HuggingFaceInferenceAPIEmbeddings
embeddings = HuggingFaceInferenceAPIEmbeddings(
    api_key=inference_api_key, model_name="sentence-transformers/all-MiniLM-l6-v2"
)
query_result = embeddings.embed_query(text)
query_result[:3]
```

[-0.038338541984558105, 0.1234646737575531, -0.028642963618040085]

## Hugging Face Hub

我们还可以通过 Hugging Face Hub 包在本地生成嵌入，这需要我们安装 ``huggingface_hub``

```python
!pip install huggingface_hub
```

```python
from langchain_huggingface.embeddings import HuggingFaceEndpointEmbeddings
```

```python
embeddings = HuggingFaceEndpointEmbeddings()
```

```python
text = "This is a test document."
```

```python
query_result = embeddings.embed_query(text)
```

```python
query_result[:3]
```