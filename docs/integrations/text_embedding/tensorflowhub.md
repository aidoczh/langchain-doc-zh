# TensorFlow Hub

[TensorFlow Hub](https://www.tensorflow.org/hub) 是一个存储库，其中包含经过训练的机器学习模型，可随时进行微调并部署到任何地方。只需几行代码，就可以重用像 `BERT` 和 `Faster R-CNN` 这样的训练模型。

让我们加载 TensorFlowHub Embedding 类。

```python
from langchain_community.embeddings import TensorflowHubEmbeddings
```

```python
embeddings = TensorflowHubEmbeddings()
```

```output
2023-01-30 23:53:01.652176: I tensorflow/core/platform/cpu_feature_guard.cc:193] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
2023-01-30 23:53:34.362802: I tensorflow/core/platform/cpu_feature_guard.cc:193] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
```

```python
text = "This is a test document."
```

```python
query_result = embeddings.embed_query(text)
```

```python
doc_results = embeddings.embed_documents(["foo"])
```

```python
doc_results
```