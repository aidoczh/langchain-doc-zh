# rag-semi-structured

这个模板用于对半结构化数据进行 RAG 处理，比如包含文本和表格的 PDF 文件。

参考[这个食谱](https://github.com/langchain-ai/langchain/blob/master/cookbook/Semi_Structured_RAG.ipynb)。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

这里使用 [Unstructured](https://unstructured-io.github.io/unstructured/) 进行 PDF 解析，需要进行一些系统级包安装。

在 Mac 上，你可以使用以下命令安装必要的包：

```shell
brew install tesseract poppler
```

## 使用方法

要使用这个包，你首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包安装为唯一包，可以执行：

```shell
langchain app new my-app --package rag-semi-structured
```

如果你想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-semi-structured
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from rag_semi_structured import chain as rag_semi_structured_chain
add_routes(app, rag_semi_structured_chain, path="/rag-semi-structured")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

你可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果你没有访问权限，可以跳过这一部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果你在这个目录中，你可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/rag-semi-structured/playground](http://127.0.0.1:8000/rag-semi-structured/playground)访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-semi-structured")
```

有关如何连接到模板的更多详细信息，请参考 Jupyter 笔记本 `rag_semi_structured`。