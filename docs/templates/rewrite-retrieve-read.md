# rewrite_retrieve_read

这个模板实现了一种查询转换（重写）的方法，用于优化 RAG，在论文[Query Rewriting for Retrieval-Augmented Large Language Models](https://arxiv.org/pdf/2305.14283.pdf)中有详细介绍。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用方法

要使用这个包，首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，可以执行以下操作：

```shell
langchain app new my-app --package rewrite_retrieve_read
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rewrite_retrieve_read
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from rewrite_retrieve_read.chain import chain as rewrite_retrieve_read_chain
add_routes(app, rewrite_retrieve_read_chain, path="/rewrite-retrieve-read")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

你可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果你没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果你在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将在本地启动一个 FastAPI 应用程序，服务器运行在 [http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上查看所有模板。

我们可以在 [http://127.0.0.1:8000/rewrite_retrieve_read/playground](http://127.0.0.1:8000/rewrite_retrieve_read/playground) 上访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rewrite_retrieve_read")
```