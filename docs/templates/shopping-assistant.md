# 购物助手

这个模板创建了一个购物助手，帮助用户找到他们正在寻找的产品。

这个模板将使用 `Ionic` 来搜索产品。

## 环境设置

这个模板将默认使用 `OpenAI`。

请确保在你的环境中设置了 `OPENAI_API_KEY`。

## 使用方法

要使用这个包，你首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，你可以这样做：

```shell
langchain app new my-app --package shopping-assistant
```

如果你想将其添加到现有项目中，你只需运行：

```shell
langchain app add shopping-assistant
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from shopping_assistant.agent import agent_executor as shopping_assistant_chain
add_routes(app, shopping_assistant_chain, path="/shopping-assistant")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

你可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。

如果你没有访问权限，你可以跳过这一部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果你在这个目录中，你可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/shopping-assistant/playground](http://127.0.0.1:8000/shopping-assistant/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/shopping-assistant")
```