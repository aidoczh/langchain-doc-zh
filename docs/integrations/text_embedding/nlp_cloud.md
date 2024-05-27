# NLP Cloud

[NLP Cloud](https://docs.nlpcloud.com/#introduction) 是一个人工智能平台，允许您使用最先进的AI引擎，甚至可以使用自己的数据训练自己的引擎。

[embeddings](https://docs.nlpcloud.com/#embeddings) 端点提供以下模型：

* `paraphrase-multilingual-mpnet-base-v2`：Paraphrase Multilingual MPNet Base V2 是基于Sentence Transformers的非常快速的模型，非常适合在50多种语言中提取嵌入（在此处查看完整列表）。

```python
%pip install --upgrade --quiet  nlpcloud
```

```python
from langchain_community.embeddings import NLPCloudEmbeddings
```

```python
import os
os.environ["NLPCLOUD_API_KEY"] = "xxx"
nlpcloud_embd = NLPCloudEmbeddings()
```

```python
text = "This is a test document."
```

```python
query_result = nlpcloud_embd.embed_query(text)
```

```python
doc_result = nlpcloud_embd.embed_documents([text])
```