# GPT4All

[GPT4All](https://gpt4all.io/index.html) 是一个免费使用的本地运行的、注重隐私的聊天机器人。无需 GPU 或互联网。它包含了流行的模型和自己的模型，如 GPT4All Falcon、Wizard 等。

本文档介绍了如何使用 [GPT4All embeddings](https://docs.gpt4all.io/gpt4all_python_embedding.html#gpt4all.gpt4all.Embed4All) 与 LangChain。

## 安装 GPT4All 的 Python 绑定

```python
%pip install --upgrade --quiet  gpt4all > /dev/null
```

注意：您可能需要重新启动内核以使用更新的软件包。

```python
from langchain_community.embeddings import GPT4AllEmbeddings
```

```python
gpt4all_embd = GPT4AllEmbeddings()
```

```output
100%|████████████████████████| 45.5M/45.5M [00:02<00:00, 18.5MiB/s]
``````output
模型已下载至:  /Users/rlm/.cache/gpt4all/ggml-all-MiniLM-L6-v2-f16.bin
``````output
objc[45711]: Class GGMLMetalClass is implemented in both /Users/rlm/anaconda3/envs/lcn2/lib/python3.9/site-packages/gpt4all/llmodel_DO_NOT_MODIFY/build/libreplit-mainline-metal.dylib (0x29fe18208) and /Users/rlm/anaconda3/envs/lcn2/lib/python3.9/site-packages/gpt4all/llmodel_DO_NOT_MODIFY/build/libllamamodel-mainline-metal.dylib (0x2a0244208). One of the two will be used. Which one is undefined.
```

```python
text = "这是一个测试文档。"
```

## 嵌入文本数据

```python
query_result = gpt4all_embd.embed_query(text)
```

使用 embed_documents 可以嵌入多个文本片段。您还可以将这些嵌入与 [Nomic's Atlas](https://docs.nomic.ai/index.html) 进行映射，以查看数据的可视化表示。

```python
doc_result = gpt4all_embd.embed_documents([text])
```