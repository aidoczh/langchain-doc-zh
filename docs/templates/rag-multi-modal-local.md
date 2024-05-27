

# rag-multi-modal-local

视觉搜索对许多使用 iPhone 或 Android 设备的人来说是一个熟悉的应用程序。它允许用户使用自然语言搜索照片。

随着开源、多模态 LLM 的发布，您可以为自己的私人照片收藏构建这种类型的应用程序。

此模板演示了如何对您的照片集执行私人视觉搜索和问答。

它使用 OpenCLIP 嵌入来嵌入所有照片，并将它们存储在 Chroma 中。

给定一个问题，相关照片将被检索并传递给您选择的开源多模态 LLM 进行答案合成。

## 输入

在 `/docs` 目录中提供一组照片。

默认情况下，此模板具有 3 张食物图片的玩具收藏。

可以提出的示例问题包括：

```
我吃了什么样的软冰淇淋？
```

在实践中，可以测试更大的图像语料库。

要创建图像的索引，请运行：

```
poetry install
python ingest.py
```

## 存储

此模板将使用 [OpenCLIP](https://github.com/mlfoundations/open_clip) 多模态嵌入来嵌入图像。

您可以选择不同的嵌入模型选项（请参阅[此处的结果](https://github.com/mlfoundations/open_clip/blob/main/docs/openclip_results.csv)）。

第一次运行应用程序时，它将自动下载多模态嵌入模型。

默认情况下，LangChain 将使用性能适中但内存需求较低的嵌入模型 `ViT-H-14`。

您可以在 `rag_chroma_multi_modal/ingest.py` 中选择替代的 `OpenCLIPEmbeddings` 模型：

```
vectorstore_mmembd = Chroma(
    collection_name="multi-modal-rag",
    persist_directory=str(re_vectorstore_path),
    embedding_function=OpenCLIPEmbeddings(
        model_name="ViT-H-14", checkpoint="laion2b_s32b_b79k"
    ),
)
```

## LLM

此模板将使用 [Ollama](https://python.langchain.com/docs/integrations/chat/ollama#multi-modal)。

下载最新版本的 Ollama：https://ollama.ai/

拉取一个开源多模态 LLM：例如，https://ollama.ai/library/bakllava

```
ollama pull bakllava
```

该应用程序默认配置为 `bakllava`。但您可以在 `chain.py` 和 `ingest.py` 中更改此配置以使用不同的已下载模型。

## 用法

要使用此软件包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package rag-chroma-multi-modal
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-chroma-multi-modal
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from rag_chroma_multi_modal import chain as rag_chroma_multi_modal_chain
add_routes(app, rag_chroma_multi_modal_chain, path="/rag-chroma-multi-modal")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在此处注册 LangSmith [here](https://smith.langchain.com/)。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/rag-chroma-multi-modal/playground](http://127.0.0.1:8000/rag-chroma-multi-modal/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-chroma-multi-modal")
```