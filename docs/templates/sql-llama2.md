

# sql-llama2

该模板使用户能够使用自然语言与 SQL 数据库进行交互。

它使用由 [Replicate](https://python.langchain.com/docs/integrations/llms/replicate) 托管的 LLamA2-13b，但可以适配支持 LLAMA2 的任何 API，包括 [Fireworks](https://python.langchain.com/docs/integrations/chat/fireworks)。

该模板包括一个包含 2023 年 NBA 球员名单的示例数据库。

有关如何构建此数据库的更多信息，请参见[这里](https://github.com/facebookresearch/llama-recipes/blob/main/demo_apps/StructuredLlama.ipynb)。

## 环境设置

确保在您的环境中设置了 `REPLICATE_API_TOKEN`。

## 使用

要使用此软件包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package sql-llama2
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add sql-llama2
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from sql_llama2 import chain as sql_llama2_chain
add_routes(app, sql_llama2_chain, path="/sql-llama2")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分

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

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/sql-llama2/playground](http://127.0.0.1:8000/sql-llama2/playground)访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/sql-llama2")
```