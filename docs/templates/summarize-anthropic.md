

# 简述人类中心

该模板使用 Anthropic 公司的 `claude-3-sonnet-20240229` 来总结长篇文档。

它利用了一个包含 100k 个标记的大上下文窗口，允许对超过 100 页的文档进行总结。

您可以在 `chain.py` 中看到总结提示。

## 环境设置

设置 `ANTHROPIC_API_KEY` 环境变量以访问 Anthropic 模型。

## 用法

要使用此软件包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package summarize-anthropic
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add summarize-anthropic
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from summarize_anthropic import chain as summarize_anthropic_chain
add_routes(app, summarize_anthropic_chain, path="/summarize-anthropic")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

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

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板

我们可以在[http://127.0.0.1:8000/summarize-anthropic/playground](http://127.0.0.1:8000/summarize-anthropic/playground)访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/summarize-anthropic")
```