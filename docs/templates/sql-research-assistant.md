# SQL 研究助手

该软件包可对 SQL 数据库进行研究。

## 用法

该软件包依赖于多个模型，这些模型有以下依赖关系：

- OpenAI：设置 `OPENAI_API_KEY` 环境变量

- Ollama：[安装和运行 Ollama](https://python.langchain.com/docs/integrations/chat/ollama)

- llama2（在 Ollama 上）：`ollama pull llama2`（否则将从 Ollama 得到 404 错误）

要使用该软件包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行以下操作：

```shell
langchain app new my-app --package sql-research-assistant
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add sql-research-assistant
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from sql_research_assistant import chain as sql_research_assistant_chain
add_routes(app, sql_research_assistant_chain, path="/sql-research-assistant")
```

（可选）现在让我们配置 LangSmith。 

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。 

您可以在 [此处](https://smith.langchain.com/) 注册 LangSmith。 

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/sql-research-assistant/playground](http://127.0.0.1:8000/sql-research-assistant/playground) 访问 playground。

我们可以通过以下代码访问代码中的模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/sql-research-assistant")
```