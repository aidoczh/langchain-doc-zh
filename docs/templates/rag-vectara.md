# rag-vectara

这个模板使用 vectara 进行 RAG。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

还要确保设置了以下环境变量：

* `VECTARA_CUSTOMER_ID`

* `VECTARA_CORPUS_ID`

* `VECTARA_API_KEY`

## 使用方法

要使用这个包，首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包安装为唯一的包，可以执行以下操作：

```shell
langchain app new my-app --package rag-vectara
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-vectara
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from rag_vectara import chain as rag_vectara_chain
add_routes(app, rag_vectara_chain, path="/rag-vectara")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

你可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果你没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "vectara-demo"
```

如果你在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上查看所有模板。

我们可以在 [http://127.0.0.1:8000/rag-vectara/playground](http://127.0.0.1:8000/rag-vectara/playground) 上访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-vectara")
```
