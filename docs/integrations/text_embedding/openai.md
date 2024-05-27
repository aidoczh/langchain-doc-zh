# OpenAI

让我们加载 OpenAI Embedding 类。

## 设置

首先我们安装 langchain-openai 并设置所需的环境变量

```python
%pip install -qU langchain-openai
```

```python
import getpass
import os
os.environ["OPENAI_API_KEY"] = getpass.getpass()
```

```python
from langchain_openai import OpenAIEmbeddings
```

```python
embeddings = OpenAIEmbeddings(model="text-embedding-3-large")
```

```python
text = "This is a test document."
```

## 用法

### 嵌入查询

```python
query_result = embeddings.embed_query(text)
```

```output
警告：未找到模型。使用 cl100k_base 编码。
```

```python
query_result[:5]
```

```output
[-0.014380056377383358,
 -0.027191711627651764,
 -0.020042716111860304,
 0.057301379620345545,
 -0.022267658631828974]
```

## 嵌入文档

```python
doc_result = embeddings.embed_documents([text])
```

```output
警告：未找到模型。使用 cl100k_base 编码。
```

```python
doc_result[0][:5]
```

```output
[-0.014380056377383358,
 -0.027191711627651764,
 -0.020042716111860304,
 0.057301379620345545,
 -0.022267658631828974]
```

## 指定维度

使用 `text-embedding-3` 类模型，您可以指定要返回的嵌入的大小。例如，默认情况下 `text-embedding-3-large` 返回维度为 3072 的嵌入：

```python
len(doc_result[0])
```

```output
3072
```

但是通过传入 `dimensions=1024`，我们可以将嵌入的大小减小到 1024：

```python
embeddings_1024 = OpenAIEmbeddings(model="text-embedding-3-large", dimensions=1024)
```

```python
len(embeddings_1024.embed_documents([text])[0])
```

```output
警告：未找到模型。使用 cl100k_base 编码。
```

```output
1024
```