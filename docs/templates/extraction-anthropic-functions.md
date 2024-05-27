

# 提取人类功能

该模板实现了[人类功能调用](https://python.langchain.com/docs/integrations/chat/anthropic_functions)。

这可以用于各种任务，比如提取或标记。

函数输出模式可以在 `chain.py` 中设置。

## 环境设置

设置 `ANTHROPIC_API_KEY` 环境变量以访问人类模型。

## 用法

要使用此软件包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此软件包安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package extraction-anthropic-functions
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add extraction-anthropic-functions
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from extraction_anthropic_functions import chain as extraction_anthropic_functions_chain
add_routes(app, extraction_anthropic_functions_chain, path="/extraction-anthropic-functions")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板

我们可以在[http://127.0.0.1:8000/extraction-anthropic-functions/playground](http://127.0.0.1:8000/extraction-anthropic-functions/playground)访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/extraction-anthropic-functions")
```

默认情况下，该软件包将从您在 `chain.py` 中指定的信息中提取论文的标题和作者。此模板默认使用 `Claude2`。

---