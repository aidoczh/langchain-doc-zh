# Langchain - Robocorp 动作服务器

该模板使得可以将 [Robocorp 动作服务器](https://github.com/robocorp/robocorp) 提供的动作作为代理工具。

## 使用方法

要使用这个包，首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，可以执行以下命令：

```shell
langchain app new my-app --package robocorp-action-server
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add robocorp-action-server
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from robocorp_action_server import agent_executor as action_server_chain
add_routes(app, action_server_chain, path="/robocorp-action-server")
```

### 运行动作服务器

要运行动作服务器，需要先安装 Robocorp 动作服务器

```bash
pip install -U robocorp-action-server
```

然后可以通过以下命令运行动作服务器：

```bash
action-server new
cd ./your-project-name
action-server start
```

### 配置 LangSmith（可选）

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

你可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

如果你没有访问权限，可以跳过这一部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

### 启动 LangServe 实例

如果你在这个目录中，可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/robocorp-action-server/playground](http://127.0.0.1:8000/robocorp-action-server/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/robocorp-action-server")
```