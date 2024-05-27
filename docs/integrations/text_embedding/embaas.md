# Embaas

[embaas](https://embaas.io) 是一个完全托管的自然语言处理（NLP）API服务，提供诸如嵌入生成、文档文本提取、文档到嵌入等功能。您可以选择各种[预训练模型](https://embaas.io/docs/models/embeddings)。

在本教程中，我们将向您展示如何使用 embaas 嵌入API为给定文本生成嵌入。

### 先决条件

在[https://embaas.io/register](https://embaas.io/register)创建您的免费 embaas 账户并生成一个[API密钥](https://embaas.io/dashboard/api-keys)。

```python
import os
# 设置API密钥
embaas_api_key = "YOUR_API_KEY"
# 或设置环境变量
os.environ["EMBAAS_API_KEY"] = "YOUR_API_KEY"
```

```python
from langchain_community.embeddings import EmbaasEmbeddings
```

```python
embeddings = EmbaasEmbeddings()
```

```python
# 为单个文档创建嵌入
doc_text = "This is a test document."
doc_text_embedding = embeddings.embed_query(doc_text)
```

```python
# 打印创建的嵌入
print(doc_text_embedding)
```

```python
# 为多个文档创建嵌入
doc_texts = ["This is a test document.", "This is another test document."]
doc_texts_embeddings = embeddings.embed_documents(doc_texts)
```

```python
# 打印创建的嵌入
for i, doc_text_embedding in enumerate(doc_texts_embeddings):
    print(f"文档 {i + 1} 的嵌入：{doc_text_embedding}")
```

```python
# 使用不同的模型和/或自定义指令
embeddings = EmbaasEmbeddings(
    model="instructor-large",
    instruction="Represent the Wikipedia document for retrieval",
)
```

有关 embaas 嵌入API的更详细信息，请参阅[官方 embaas API文档](https://embaas.io/api-reference)。