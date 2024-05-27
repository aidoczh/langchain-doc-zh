# pii-protected-chatbot

这个模板创建了一个聊天机器人，它会标记任何传入的个人身份信息（PII），并且不会将其传递给LLM。

## 环境设置

需要设置以下环境变量：

将 `OPENAI_API_KEY` 环境变量设置为访问OpenAI模型。

## 使用方法

要使用这个包，首先需要安装LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的LangChain项目并将其安装为唯一的包，可以执行以下操作：

```shell
langchain app new my-app --package pii-protected-chatbot
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add pii-protected-chatbot
```

然后在 `server.py` 文件中添加以下代码：

```python
from pii_protected_chatbot.chain import chain as pii_protected_chatbot
add_routes(app, pii_protected_chatbot, path="/openai-functions-agent")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监控和调试LangChain应用程序。

您可以在[这里](https://smith.langchain.com/)注册LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果您在此目录中，则可以直接启动LangServe实例：

```shell
langchain serve
```

这将在本地启动一个运行在 [http://localhost:8000](http://localhost:8000) 的FastAPI应用程序服务器。

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上查看所有模板。

我们可以在 [http://127.0.0.1:8000/pii_protected_chatbot/playground](http://127.0.0.1:8000/pii_protected_chatbot/playground) 上访问playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/pii_protected_chatbot")
```