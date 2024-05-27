

# rag-multi-modal-mv-local

视觉搜索对于许多使用 iPhone 或 Android 设备的用户来说是一个熟悉的应用程序。它允许用户使用自然语言搜索照片。

随着开源、多模态LLM的发布，您可以为自己的私人照片收藏构建这种类型的应用程序。

这个模板演示了如何在您的照片集合上执行私人视觉搜索和问答。

它使用您选择的开源多模态LLM来为每张照片创建图像摘要，嵌入这些摘要，并将它们存储在Chroma中。

给定一个问题，相关照片将被检索并传递给多模态LLM进行答案合成。

## 输入

在 `/docs` 目录中提供一组照片。

默认情况下，此模板具有一个包含3张食物图片的玩具收藏。

该应用程序将根据提供的关键词或问题查找并总结照片：

```
我吃了什么口味的冰淇淋？
```

在实践中，可以测试更大的图像语料库。

要创建图像的索引，运行：

```
poetry install
python ingest.py
```

## 存储

这是模板将用于创建幻灯片索引的过程（参见[博客](https://blog.langchain.dev/multi-modal-rag-template/)）：

* 给定一组图像

* 使用本地多模态LLM（[bakllava](https://ollama.ai/library/bakllava)）对每个图像进行摘要

* 将图像摘要与指向原始图像的链接嵌入

* 给定用户问题，它将基于图像摘要与用户输入之间的相似性（使用Ollama嵌入）检索相关图像

* 它将传递这些图像给bakllava进行答案合成

默认情况下，这将使用[LocalFileStore](https://python.langchain.com/docs/integrations/stores/file_system)存储图像，使用Chroma存储摘要。

## LLM和嵌入模型

我们将使用[Ollama](https://python.langchain.com/docs/integrations/chat/ollama#multi-modal)生成图像摘要、嵌入和最终图像问答。

下载最新版本的Ollama：https://ollama.ai/

拉取一个开源多模态LLM：例如，https://ollama.ai/library/bakllava

拉取一个开源嵌入模型：例如，https://ollama.ai/library/llama2:7b

```
ollama pull bakllava
ollama pull llama2:7b
```

该应用程序默认配置为使用 `bakllava`。但您可以在 `chain.py` 和 `ingest.py` 中更改为不同的下载模型。

该应用程序将根据文本输入和图像摘要之间的相似性检索图像，并将图像传递给 `bakllava`。

## 用法

要使用此软件包，您应首先安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package rag-multi-modal-mv-local
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-multi-modal-mv-local
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_multi_modal_mv_local import chain as rag_multi_modal_mv_local_chain
add_routes(app, rag_multi_modal_mv_local_chain, path="/rag-multi-modal-mv-local")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监视和调试LangChain应用程序。

您可以在此处注册LangSmith [here](https://smith.langchain.com/)。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，则可以通过以下方式直接启动LangServe实例：

```shell
langchain serve
```

这将启动一个带有服务器的FastAPI应用程序，本地运行的服务器位于

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板

我们可以在[http://127.0.0.1:8000/rag-multi-modal-mv-local/playground](http://127.0.0.1:8000/rag-multi-modal-mv-local/playground)访问playground

我们可以通过以下方式从代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-multi-modal-mv-local")
```