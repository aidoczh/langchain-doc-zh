

# 多模态多向量 RAG-Chroma

多模态LLM使得视觉助手能够对图像进行问答。

该模板创建了一个用于幻灯片演示的视觉助手，幻灯片通常包含图表或图片等可视化内容。

它使用 GPT-4V 为每张幻灯片创建图像摘要，将摘要嵌入并存储在 Chroma 中。

给定一个问题，相关幻灯片将被检索并传递给 GPT-4V 进行答案合成。

## 输入

在 `/docs` 目录中提供一个 PDF 格式的幻灯片演示。

默认情况下，此模板包含有关 DataDog 的 Q3 收益的幻灯片演示，DataDog 是一家公开的科技公司。

可以询问的示例问题包括：

```
Datadog有多少客户？
Datadog平台在FY20、FY21和FY22的年度增长率是多少？
```

要创建幻灯片演示的索引，请运行：

```
poetry install
python ingest.py
```

## 存储

以下是模板将使用的过程来创建幻灯片索引（参见[博客](https://blog.langchain.dev/multi-modal-rag-template/)）：

* 提取幻灯片作为图像集合

* 使用 GPT-4V 对每个图像进行摘要

* 使用文本嵌入嵌入图像摘要，并附带原始图像的链接

* 基于图像摘要与用户输入问题之间的相似度检索相关图像

* 将这些图像传递给 GPT-4V 进行答案合成

默认情况下，这将使用 [LocalFileStore](https://python.langchain.com/docs/integrations/stores/file_system) 存储图像，使用 Chroma 存储摘要。

在生产环境中，可能希望使用远程选项，如 Redis。

您可以在 `chain.py` 和 `ingest.py` 中设置 `local_file_store` 标志以在两个选项之间切换。

对于 Redis，该模板将使用 [UpstashRedisByteStore](https://python.langchain.com/docs/integrations/stores/upstash_redis)。

我们将使用 Upstash 来存储图像，它提供带有 REST API 的 Redis。

只需在[这里](https://upstash.com/)登录并创建一个数据库。

这将为您提供一个带有以下内容的 REST API：

* `UPSTASH_URL`

* `UPSTASH_TOKEN`

将 `UPSTASH_URL` 和 `UPSTASH_TOKEN` 设置为环境变量以访问您的数据库。

我们将使用 Chroma 来存储和索引图像摘要，这些摘要将在模板目录中本地创建。

## LLM

该应用程序将根据文本输入与图像摘要之间的相似度检索图像，并将图像传递给 GPT-4V。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI GPT-4V。

如果使用 `UpstashRedisByteStore`，则将 `UPSTASH_URL` 和 `UPSTASH_TOKEN` 设置为环境变量以访问您的数据库。

## 使用

要使用此软件包，您首先应安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package rag-chroma-multi-modal-multi-vector
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-chroma-multi-modal-multi-vector
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_chroma_multi_modal_multi_vector import chain as rag_chroma_multi_modal_chain_mv
add_routes(app, rag_chroma_multi_modal_chain_mv, path="/rag-chroma-multi-modal-multi-vector")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[此处](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/rag-chroma-multi-modal-multi-vector/playground](http://127.0.0.1:8000/rag-chroma-multi-modal-multi-vector/playground)访问 playground。

我们可以通过以下方式从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-chroma-multi-modal-multi-vector")
```