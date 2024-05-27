# 命题检索

这个模板演示了陈等人提出的多向量索引策略，详见[Dense X Retrieval: What Retrieval Granularity Should We Use?](https://arxiv.org/abs/2312.06648)。该提示可以在[此处](https://smith.langchain.com/hub/wfh/proposal-indexing)尝试，指导一个LLM生成去上下文化的“命题”，这些命题可以被向量化以提高检索准确性。你可以在`proposal_chain.py`中看到完整的定义。

## 存储

在这个演示中，我们使用RecursiveUrlLoader索引了一篇简单的学术论文，并将所有的检索信息存储在本地（使用chroma和存储在本地文件系统上的bytestore）。你可以在`storage.py`中修改存储层。

## 环境设置

设置`OPENAI_API_KEY`环境变量以访问`gpt-3.5`和OpenAI Embeddings类。

## 索引

通过运行以下命令来创建索引：

```python
poetry install
poetry run python propositional_retrieval/ingest.py
```

## 使用

要使用这个包，你首先应该安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其安装为唯一的包，你可以执行：

```shell
langchain app new my-app --package propositional-retrieval
```

如果你想将其添加到现有项目中，只需运行：

```shell
langchain app add propositional-retrieval
```

并将以下代码添加到你的`server.py`文件中：

```python
from propositional_retrieval import chain
add_routes(app, chain, path="/propositional-retrieval")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监控和调试LangChain应用程序。

你可以在[这里](https://smith.langchain.com/)注册LangSmith。

如果你没有访问权限，可以跳过这一部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果你在这个目录中，你可以直接通过以下方式启动LangServe实例：

```shell
langchain serve
```

这将启动一个带有服务器的FastAPI应用程序，本地运行的服务器位于

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/propositional-retrieval/playground](http://127.0.0.1:8000/propositional-retrieval/playground)访问playground。

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/propositional-retrieval")
```