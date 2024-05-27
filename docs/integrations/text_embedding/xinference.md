# Xorbits 推理（Xinference）

本文档介绍如何在 LangChain 中使用 Xinference 嵌入。

## 安装

通过 PyPI 安装 `Xinference`：

```python
%pip install --upgrade --quiet  "xinference[all]"
```

## 本地部署或分布式集群部署 Xinference

要进行本地部署，请运行 `xinference`。

要在集群中部署 Xinference，请先使用 `xinference-supervisor` 启动 Xinference 主管。您还可以使用选项 -p 指定端口和 -H 指定主机。默认端口为 9997。

然后，在要运行它们的每台服务器上使用 `xinference-worker` 启动 Xinference 工作进程。

您可以在 [Xinference](https://github.com/xorbitsai/inference) 的 README 文件中查阅更多信息。

## 包装器

要在 LangChain 中使用 Xinference，您需要首先启动一个模型。您可以使用命令行界面（CLI）来执行此操作：

```python
!xinference launch -n vicuna-v1.3 -f ggmlv3 -q q4_0
```

```output
Model uid: 915845ee-2a04-11ee-8ed4-d29396a3f064
```

系统将返回一个模型 UID 供您使用。现在您可以使用 LangChain 与 Xinference 嵌入：

```python
from langchain_community.embeddings import XinferenceEmbeddings
xinference = XinferenceEmbeddings(
    server_url="http://0.0.0.0:9997", model_uid="915845ee-2a04-11ee-8ed4-d29396a3f064"
)
```

```python
query_result = xinference.embed_query("This is a test query")
```

```python
doc_result = xinference.embed_documents(["text A", "text B"])
```

最后，在不需要使用模型时终止它：

```python
!xinference terminate --model-uid "915845ee-2a04-11ee-8ed4-d29396a3f064"
```