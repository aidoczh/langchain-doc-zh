

# rag-gemini-multi-modal

多模态LLM使得视觉助手能够对图像进行问答。

该模板创建了一个用于幻灯片演示的视觉助手，幻灯片通常包含图表或图片等可视化内容。

它使用OpenCLIP嵌入来嵌入所有幻灯片图像，并将它们存储在Chroma中。

给定一个问题，相关幻灯片将被检索并传递给[Google Gemini](https://deepmind.google/technologies/gemini/#introduction)进行答案合成。

## 输入

在`/docs`目录中提供一个pdf格式的幻灯片演示文稿。

默认情况下，此模板包含有关DataDog的Q3收入的幻灯片演示，DataDog是一家公开的科技公司。

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

该模板将使用[OpenCLIP](https://github.com/mlfoundations/open_clip)多模态嵌入来嵌入这些图片。

您可以选择不同的嵌入模型选项（请参阅[这里](https://github.com/mlfoundations/open_clip/blob/main/docs/openclip_results.csv)的结果）。

第一次运行应用程序时，它将自动下载多模态嵌入模型。

默认情况下，LangChain将使用性能适中但内存需求较低的嵌入模型`ViT-H-14`。

您可以在`rag_chroma_multi_modal/ingest.py`中选择替代的`OpenCLIPEmbeddings`模型：

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

该应用程序将使用多模态嵌入检索图像，并将它们传递给Google Gemini。

## 环境设置

设置您的`GOOGLE_API_KEY`环境变量以访问Gemini。

## 使用

要使用此软件包，您首先应安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package rag-gemini-multi-modal
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-gemini-multi-modal
```

并将以下代码添加到您的`server.py`文件中：

```python
from rag_gemini_multi_modal import chain as rag_gemini_multi_modal_chain
add_routes(app, rag_gemini_multi_modal_chain, path="/rag-gemini-multi-modal")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监视和调试LangChain应用程序。

您可以在[这里](https://smith.langchain.com/)注册LangSmith。

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

这将启动FastAPI应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/rag-gemini-multi-modal/playground](http://127.0.0.1:8000/rag-gemini-multi-modal/playground)访问playground。

我们可以通过以下方式从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-gemini-multi-modal")
```