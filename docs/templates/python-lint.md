# Python代码质量检查

这个代理专门用于生成高质量的Python代码，重点关注正确的格式和代码检查。它使用 `black`、`ruff` 和 `mypy` 来确保代码符合标准质量检查。

通过整合和响应这些检查，简化了编码过程，从而产生可靠且一致的代码输出。

它实际上不能执行它所写的代码，因为代码执行可能引入额外的依赖项和潜在的安全漏洞。这使得该代理既是一个安全又高效的代码生成解决方案。

你可以直接使用它来生成Python代码，或者将其与规划和执行代理进行网络连接。

## 环境设置

- 安装 `black`、`ruff` 和 `mypy`：`pip install -U black ruff mypy`

- 设置 `OPENAI_API_KEY` 环境变量。

## 使用方法

要使用这个包，你应该首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，你可以执行：

```shell
langchain app new my-app --package python-lint
```

如果你想将其添加到现有项目中，只需运行：

```shell
langchain app add python-lint
```

然后在你的 `server.py` 文件中添加以下代码：

```python
from python_lint import agent_executor as python_lint_agent
add_routes(app, python_lint_agent, path="/python-lint")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

你可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果你没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果你在这个目录中，那么你可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板。

我们可以在[http://127.0.0.1:8000/python-lint/playground](http://127.0.0.1:8000/python-lint/playground)访问 playground。

我们可以通过以下方式从代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/python-lint")
```