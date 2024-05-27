# rag-vectara-multiquery

这个模板执行了使用 Vectara 进行多查询的 RAG。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

同时，确保设置了以下环境变量：

* `VECTARA_CUSTOMER_ID`

* `VECTARA_CORPUS_ID`

* `VECTARA_API_KEY`

## 用法

要使用这个包，你首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，你可以执行：

```shell
langchain app new my-app --package rag-vectara-multiquery
```

如果你想将其添加到现有项目中，你可以直接运行：

```shell
langchain app add rag-vectara-multiquery
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from rag_vectara import chain as rag_vectara_chain
add_routes(app, rag_vectara_chain, path="/rag-vectara-multiquery")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

你可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

如果你没有访问权限，你可以跳过这一部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "vectara-demo"
```

如果你在这个目录中，你可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个 FastAPI 应用程序，服务器在本地运行，地址为 

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/rag-vectara-multiquery/playground](http://127.0.0.1:8000/rag-vectara-multiquery/playground) 访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-vectara-multiquery")
```