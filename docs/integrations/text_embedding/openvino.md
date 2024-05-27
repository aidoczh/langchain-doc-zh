# OpenVINO

[OpenVINO™](https://github.com/openvinotoolkit/openvino) 是一个用于优化和部署 AI 推断的开源工具包。OpenVINO™ Runtime 支持各种硬件[设备](https://github.com/openvinotoolkit/openvino?tab=readme-ov-file#supported-hardware-matrix)，包括 x86 和 ARM CPU，以及 Intel GPU。它可以帮助提升计算机视觉、自动语音识别、自然语言处理和其他常见任务的深度学习性能。

Hugging Face 嵌入模型可以通过 ``OpenVINOEmbeddings`` 类来支持 OpenVINO。如果你有 Intel GPU，可以指定 `model_kwargs={"device": "GPU"}` 来在其上运行推断。

```python
%pip install --upgrade-strategy eager "optimum[openvino,nncf]" --quiet
```

```output
Note: you may need to restart the kernel to use updated packages.
```

```python
from langchain_community.embeddings import OpenVINOEmbeddings
```

```python
model_name = "sentence-transformers/all-mpnet-base-v2"
model_kwargs = {"device": "CPU"}
encode_kwargs = {"mean_pooling": True, "normalize_embeddings": True}
ov_embeddings = OpenVINOEmbeddings(
    model_name_or_path=model_name,
    model_kwargs=model_kwargs,
    encode_kwargs=encode_kwargs,
)
```

```python
text = "This is a test document."
```

```python
query_result = ov_embeddings.embed_query(text)
```

```python
query_result[:3]
```

```output
[-0.048951778560876846, -0.03986183926463127, -0.02156277745962143]
```

```python
doc_result = ov_embeddings.embed_documents([text])
```

## 导出 IR 模型

可以使用 ``OVModelForFeatureExtraction`` 将嵌入模型导出为 OpenVINO IR 格式，并从本地文件夹加载模型。

```python
from pathlib import Path
ov_model_dir = "all-mpnet-base-v2-ov"
if not Path(ov_model_dir).exists():
    ov_embeddings.save_model(ov_model_dir)
```

```python
ov_embeddings = OpenVINOEmbeddings(
    model_name_or_path=ov_model_dir,
    model_kwargs=model_kwargs,
    encode_kwargs=encode_kwargs,
)
```

```output
Compiling the model to CPU ...
```

## 使用 OpenVINO 进行 BGE

我们还可以通过 OpenVINO 使用 ``OpenVINOBgeEmbeddings`` 类访问 BGE 嵌入模型。

```python
from langchain_community.embeddings import OpenVINOBgeEmbeddings
model_name = "BAAI/bge-small-en"
model_kwargs = {"device": "CPU"}
encode_kwargs = {"normalize_embeddings": True}
ov_embeddings = OpenVINOBgeEmbeddings(
    model_name_or_path=model_name,
    model_kwargs=model_kwargs,
    encode_kwargs=encode_kwargs,
)
```

```python
embedding = ov_embeddings.embed_query("hi this is harrison")
len(embedding)
```

```output
384
```

更多信息请参阅：

* [OpenVINO LLM 指南](https://docs.openvino.ai/2024/learn-openvino/llm_inference_guide.html)。

* [OpenVINO 文档](https://docs.openvino.ai/2024/home.html)。

* [OpenVINO 入门指南](https://www.intel.com/content/www/us/en/content-details/819067/openvino-get-started-guide.html)。

* [LangChain 的 RAG Notebook](https://github.com/openvinotoolkit/openvino_notebooks/tree/latest/notebooks/llm-rag-langchain)。