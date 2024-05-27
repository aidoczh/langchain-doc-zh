# xml-agent

这个软件包创建了一个代理，使用 XML 语法来传达其决策以及采取的行动。它使用 Anthropic 的 Claude 模型来编写 XML 语法，并可以选择使用 DuckDuckGo 在互联网上查找信息。

## 环境设置

需要设置两个环境变量：

- `ANTHROPIC_API_KEY`：使用 Anthropic 必需

## 使用方法

要使用这个软件包，首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的软件包安装，可以执行以下操作：

```shell
langchain app new my-app --package xml-agent
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add xml-agent
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from xml_agent import agent_executor as xml_agent_chain
add_routes(app, xml_agent_chain, path="/xml-agent")
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

这将在本地启动一个 FastAPI 应用程序，服务器正在运行在 [http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/xml-agent/playground](http://127.0.0.1:8000/xml-agent/playground) 访问 playground。

我们可以通过以下代码访问代码中的模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/xml-agent")
```