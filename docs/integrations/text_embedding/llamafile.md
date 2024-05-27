# llamafile

让我们加载 [llamafile](https://github.com/Mozilla-Ocho/llamafile) 嵌入类。

## 设置

首先，有 3 个设置步骤：

1. 下载一个 llamafile。在这个笔记本中，我们使用 `TinyLlama-1.1B-Chat-v1.0.Q5_K_M`，但在 [HuggingFace](https://huggingface.co/models?other=llamafile) 上还有许多其他可用的版本。

2. 使 llamafile 可执行。

3. 启动 llamafile 服务器模式。

您可以运行以下 bash 脚本来完成所有这些操作：

```bash
%%bash
# llamafile 设置
# 步骤 1: 下载一个 llamafile。下载可能需要几分钟时间。
wget -nv -nc https://huggingface.co/jartine/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile
# 步骤 2: 使 llamafile 可执行。注意：如果您使用的是 Windows，只需在文件名后面添加 '.exe'。
chmod +x TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile
# 步骤 3: 在后台启动 llamafile 服务器。所有服务器日志将被写入 'tinyllama.log'。
# 或者，您可以在此笔记本之外打开一个单独的终端并运行：
#   ./TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile --server --nobrowser --embedding
./TinyLlama-1.1B-Chat-v1.0.Q5_K_M.llamafile --server --nobrowser --embedding > tinyllama.log 2>&1 &
pid=$!
echo "${pid}" > .llamafile_pid  # 将进程 pid 写入文件，以便稍后终止服务器
```

## 使用 LlamafileEmbeddings 嵌入文本

现在，我们可以使用 `LlamafileEmbeddings` 类与当前在 http://localhost:8080 提供我们的 TinyLlama 模型的 llamafile 服务器进行交互。

```python
from langchain_community.embeddings import LlamafileEmbeddings
```

```python
embedder = LlamafileEmbeddings()
```

```python
text = "这是一个测试文档。"
```

要生成嵌入，您可以查询单个文本，也可以查询文本列表。

```python
query_result = embedder.embed_query(text)
query_result[:5]
```

```python
doc_result = embedder.embed_documents([text])
doc_result[0][:5]
```

```bash
%%bash
# 清理：终止 llamafile 服务器进程
kill $(cat .llamafile_pid)
rm .llamafile_pid
```