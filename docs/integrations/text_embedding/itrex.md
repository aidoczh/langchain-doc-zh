# Intel® Extension for Transformers 量化文本嵌入

加载由 [Intel® Extension for Transformers](https://github.com/intel/intel-extension-for-transformers) (ITREX) 生成的量化 BGE 嵌入模型，并使用 ITREX [神经引擎](https://github.com/intel/intel-extension-for-transformers/blob/main/intel_extension_for_transformers/llm/runtime/deprecated/docs/Installation.md)，这是一个高性能的 NLP 后端，以加速模型的推断，同时不影响准确性。

更多详情请参阅我们的博客[使用 Intel Extension for Transformers 实现高效的自然语言嵌入模型](https://medium.com/intel-analytics-software/efficient-natural-language-embedding-models-with-intel-extension-for-transformers-2b6fcd0f8f34)以及[BGE 优化示例](https://github.com/intel/intel-extension-for-transformers/tree/main/examples/huggingface/pytorch/text-embedding/deployment/mteb/bge)。

```python
from langchain_community.embeddings import QuantizedBgeEmbeddings
model_name = "Intel/bge-small-en-v1.5-sts-int8-static-inc"
encode_kwargs = {"normalize_embeddings": True}  # 设置为 True 以计算余弦相似度
model = QuantizedBgeEmbeddings(
    model_name=model_name,
    encode_kwargs=encode_kwargs,
    query_instruction="Represent this sentence for searching relevant passages: ",
)
```

```output
/home/yuwenzho/.conda/envs/bge/lib/python3.9/site-packages/tqdm/auto.py:21: TqdmWarning: IProgress not found. Please update jupyter and ipywidgets. See https://ipywidgets.readthedocs.io/en/stable/user_install.html
  from .autonotebook import tqdm as notebook_tqdm
2024-03-04 10:17:17 [INFO] 开始提取 onnx 模型操作...
2024-03-04 10:17:17 [INFO] 提取 onnxruntime 模型完成...
2024-03-04 10:17:17 [INFO] 开始实现子图匹配和替换...
2024-03-04 10:17:18 [INFO] 子图匹配和替换完成...
```

## 用法

```python
text = "这是一个测试文档。"
query_result = model.embed_query(text)
doc_result = model.embed_documents([text])
```