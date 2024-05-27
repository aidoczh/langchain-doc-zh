# rag-momento-vector-index

这个模板使用 Momento Vector Index (MVI) 和 OpenAI 来执行 RAG。

> MVI：最高效、最易于使用的无服务器向量索引服务。要开始使用 MVI，只需注册一个账号。无需处理基础设施、管理服务器或担心扩展性。MVI 是一个根据需求自动扩展的服务。可以与其他 Momento 服务结合使用，如 Momento Cache 用于缓存提示和作为会话存储，或 Momento Topics 作为发布/订阅系统向应用程序广播事件。

要注册并访问 MVI，请访问[Momento 控制台](https://console.gomomento.com/)。

## 环境设置

这个模板使用 Momento Vector Index 作为向量存储，并需要设置 `MOMENTO_API_KEY` 和 `MOMENTO_INDEX_NAME`。

前往[控制台](https://console.gomomento.com/)获取 API 密钥。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用方法

要使用这个包，首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包安装为唯一的包，可以执行以下命令：

```shell
langchain app new my-app --package rag-momento-vector-index
```

如果要将此包添加到现有项目中，只需运行：

```shell
langchain app add rag-momento-vector-index
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from rag_momento_vector_index import chain as rag_momento_vector_index_chain
add_routes(app, rag_momento_vector_index_chain, path="/rag-momento-vector-index")
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

如果你在此目录中，可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将在本地启动一个运行在 [http://localhost:8000](http://localhost:8000) 的 FastAPI 应用程序。

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/rag-momento-vector-index/playground](http://127.0.0.1:8000/rag-momento-vector-index/playground) 访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-momento-vector-index")
```

## 数据索引

我们已经包含了一个用于索引数据的示例模块。该模块位于 `rag_momento_vector_index/ingest.py`。在 `chain.py` 中有一行被注释掉的代码调用了它。取消注释以使用。