# Qdrant 的 FastEmbed

[Qdrant](https://qdrant.tech) 公司推出的 [FastEmbed](https://qdrant.github.io/fastembed/) 是一个轻量、快速的 Python 库，专为嵌入式生成而设计。

- 量化模型权重

- 使用 ONNX Runtime，无需依赖 PyTorch

- 以 CPU 为先的设计

- 用于对大型数据集进行编码的数据并行处理。

## 依赖关系

要在 LangChain 中使用 FastEmbed，请安装 `fastembed` Python 包。

```python
%pip install --upgrade --quiet  fastembed
```

## 导入

```python
from langchain_community.embeddings.fastembed import FastEmbedEmbeddings
```

## 实例化 FastEmbed

### 参数

- `model_name: str` (默认值: "BAAI/bge-small-en-v1.5")

    > 要使用的 FastEmbedding 模型的名称。您可以在[这里](https://qdrant.github.io/fastembed/examples/Supported_Models/)找到支持的模型列表。

- `max_length: int` (默认值: 512)

    > 最大标记数。对于值大于 512 的行为未知。

- `cache_dir: Optional[str]`

    > 缓存目录的路径。默认为父目录中的 `local_cache`。

- `threads: Optional[int]`

    > 单个 onnxruntime 会话可以使用的线程数。默认为 None。

- `doc_embed_type: Literal["default", "passage"]` (默认值: "default")

    > "default": 使用 FastEmbed 的默认嵌入方法。

    

    > "passage": 在嵌入之前为文本加上 "passage" 前缀。

```python
embeddings = FastEmbedEmbeddings()
```

## 用法

### 生成文档嵌入

```python
document_embeddings = embeddings.embed_documents(
    ["这是一个文档", "这是另一个文档"]
)
```

### 生成查询嵌入

```python
query_embeddings = embeddings.embed_query("这是一个查询")
```