# 使用优化和量化的嵌入器嵌入文档

使用量化嵌入器嵌入所有文档。

这些嵌入器基于优化模型，通过使用[optimum-intel](https://github.com/huggingface/optimum-intel.git)和[IPEX](https://github.com/intel/intel-extension-for-pytorch)创建。

示例文本基于[SBERT](https://www.sbert.net/docs/pretrained_cross-encoders.html)。

```python
from langchain_community.embeddings import QuantizedBiEncoderEmbeddings
model_name = "Intel/bge-small-en-v1.5-rag-int8-static"
encode_kwargs = {"normalize_embeddings": True}  # 设置为True以计算余弦相似度
model = QuantizedBiEncoderEmbeddings(
    model_name=model_name,
    encode_kwargs=encode_kwargs,
    query_instruction="Represent this sentence for searching relevant passages: ",
)
```

```output
从缓存中加载配置文件inc_config.json
INCConfig {
  "distillation": {},
  "neural_compressor_version": "2.4.1",
  "optimum_version": "1.16.2",
  "pruning": {},
  "quantization": {
    "dataset_num_samples": 50,
    "is_static": true
  },
  "save_onnx_model": false,
  "torch_version": "2.2.0",
  "transformers_version": "4.37.2"
}
使用`INCModel`加载TorchScript模型将在v1.15.0中弃用，请使用`IPEXModel`代替加载您的模型。
```

让我们提出一个问题，并比较两个文档。第一个包含问题的答案，而第二个则不包含。

我们可以检查哪个更适合我们的查询。

```python
question = "柏林有多少人口？"
```

```python
documents = [
    "柏林有 3,520,031 名注册居民，占地面积为 891.82 平方公里。",
    "柏林以其博物馆闻名。",
]
```

```python
doc_vecs = model.embed_documents(documents)
```

```output
Batches: 100%|██████████| 1/1 [00:00<00:00,  4.18it/s]
```

```python
query_vec = model.embed_query(question)
```

```python
import torch
```

```python
doc_vecs_torch = torch.tensor(doc_vecs)
```

```python
query_vec_torch = torch.tensor(query_vec)
```

```python
query_vec_torch @ doc_vecs_torch.T
```

```output
tensor([0.7980, 0.6529])
```

我们可以看到，第一个确实排名更高。