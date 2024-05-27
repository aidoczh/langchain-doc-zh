# LLMRails

让我们加载 LLMRails Embeddings 类。

要使用 LLMRails 嵌入，您需要通过参数传递 API 密钥，或者在环境中设置 `LLM_RAILS_API_KEY` 键。

要获取 API 密钥，您需要在 https://console.llmrails.com/signup 注册，然后转到 https://console.llmrails.com/api-keys，在平台上创建一个密钥后从那里复制密钥。

```python
from langchain_community.embeddings import LLMRailsEmbeddings
```

```python
embeddings = LLMRailsEmbeddings(model="embedding-english-v1")  # 或者 embedding-multi-v1
```

```python
text = "This is a test document."
```

要生成嵌入，您可以查询单个文本，也可以查询文本列表。

```python
query_result = embeddings.embed_query(text)
query_result[:5]
```

```output
[-0.09996652603149414,
 0.015568195842206478,
 0.17670190334320068,
 0.16521021723747253,
 0.21193109452724457]
```

```python
doc_result = embeddings.embed_documents([text])
doc_result[0][:5]
```

```output
[-0.04242777079343796,
 0.016536075621843338,
 0.10052520781755447,
 0.18272875249385834,
 0.2079043835401535]
```