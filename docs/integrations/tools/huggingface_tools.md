# HuggingFace Hub 工具

>[Huggingface Tools](https://huggingface.co/docs/transformers/v4.29.0/en/custom_tools) 支持直接使用 `load_huggingface_tool` 函数加载支持文本输入输出的工具。

```python
# 需要 transformers>=4.29.0 和 huggingface_hub>=0.14.1
%pip install --upgrade --quiet  transformers huggingface_hub > /dev/null
```

```python
from langchain.agents import load_huggingface_tool
tool = load_huggingface_tool("lysandre/hf-model-downloads")
print(f"{tool.name}: {tool.description}")
```

```output
model_download_counter: 这是一个工具，可以返回 Hugging Face Hub 上给定任务中下载量最高的模型。它接受类别的名称（例如文本分类、深度估计等），并返回检查点的名称。
```

```python
tool.run("text-classification")
```

```output
'facebook/bart-large-mnli'
```