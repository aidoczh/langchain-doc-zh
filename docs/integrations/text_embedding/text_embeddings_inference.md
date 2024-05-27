# 文本嵌入推断

[Hugging Face 文本嵌入推断（TEI）](https://huggingface.co/docs/text-embeddings-inference/index)是一个用于部署和提供开源文本嵌入和序列分类模型的工具包。`TEI`实现了对最流行模型的高性能提取，包括`FlagEmbedding`、`Ember`、`GTE`和`E5`。

要在 langchain 中使用它，首先安装`huggingface-hub`。

```python
%pip install --upgrade huggingface-hub
```

然后使用 TEI 暴露一个嵌入模型。例如，使用 Docker，您可以如下提供`BAAI/bge-large-en-v1.5`：

```bash
model=BAAI/bge-large-en-v1.5
revision=refs/pr/5
volume=$PWD/data # 与 Docker 容器共享卷，以避免每次运行都下载权重
docker run --gpus all -p 8080:80 -v $volume:/data --pull always ghcr.io/huggingface/text-embeddings-inference:0.6 --model-id $model --revision $revision
```

最后，实例化客户端并嵌入您的文本。

```python
from langchain_huggingface.embeddings import HuggingFaceEndpointEmbeddings
```

```python
embeddings = HuggingFaceEndpointEmbeddings(model="http://localhost:8080")
```

```python
text = "What is deep learning?"
```

```python
query_result = embeddings.embed_query(text)
query_result[:3]
```

```output
[0.018113142, 0.00302585, -0.049911194]
```

```python
doc_result = embeddings.embed_documents([text])
```

```python
doc_result[0][:3]
```

```output
[0.018113142, 0.00302585, -0.049911194]
```