# OpenAI Functions Agent - Gmail

曾经为了达到收件箱零封的目标而苦苦挣扎过吗？

使用这个模板，你可以创建和定制自己的 AI 助手来管理你的 Gmail 账户。它可以使用默认的 Gmail 工具，读取、搜索和起草邮件，代表你进行回复。它还可以访问 Tavily 搜索引擎，以便在撰写邮件之前搜索与邮件主题或人物相关的信息，确保草稿包含所有必要的信息，以显得博学。

## 详细信息

该助手使用 OpenAI 的[函数调用](https://python.langchain.com/docs/modules/chains/how_to/openai_functions)支持来可靠地选择和调用你提供的工具。

该模板还直接从[langchain-core](https://pypi.org/project/langchain-core/)和[`langchain-community`](https://pypi.org/project/langchain-community/)导入适当的内容。我们已经重新组织了 LangChain，让你可以选择你的用例所需的特定集成。虽然你仍然可以从`langchain`导入（我们正在使这个过渡向后兼容），但我们已经将大部分类的归属地分开，以反映所有权并使你的依赖列表更轻。你需要的大部分集成可以在`langchain-community`包中找到，如果你只是使用核心表达语言 API，甚至可以完全基于`langchain-core`构建。

## 环境设置

需要设置以下环境变量：

将`OPENAI_API_KEY`环境变量设置为访问 OpenAI 模型。

将`TAVILY_API_KEY`环境变量设置为访问 Tavily 搜索。

创建一个包含来自 Gmail 的 OAuth 客户端 ID 的[`credentials.json`](https://developers.google.com/gmail/api/quickstart/python#authorize_credentials_for_a_desktop_application)文件。要自定义身份验证，请参见下面的[自定义身份验证](#customize-auth)部分。

_*注意:* 第一次运行此应用程序时，它将强制你进行用户身份验证流程。_

（可选）：将`GMAIL_AGENT_ENABLE_SEND`设置为`true`（或修改此模板中的`agent.py`文件），以使其可以访问“发送”工具。这将使你的助手有权限代表你发送邮件，而无需你明确审查，但这并不推荐。

## 使用方法

要使用此包，你首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一的包，可以执行以下操作：

```shell
langchain app new my-app --package openai-functions-agent-gmail
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add openai-functions-agent-gmail
```

并将以下代码添加到你的`server.py`文件中：

```python
from openai_functions_agent import agent_executor as openai_functions_agent_chain
add_routes(app, openai_functions_agent_chain, path="/openai-functions-agent-gmail")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

你可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果你没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为"default"
```

如果你在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)上查看所有模板。

我们可以在[http://127.0.0.1:8000/openai-functions-agent-gmail/playground](http://127.0.0.1:8000/openai-functions-agent/playground)上访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/openai-functions-agent-gmail")
```

## 自定义身份验证

```
from langchain_community.tools.gmail.utils import build_resource_service, get_gmail_credentials
# 可在此处查看范围 https://developers.google.com/gmail/api/auth/scopes
# 例如，只读范围是 'https://www.googleapis.com/auth/gmail.readonly'
credentials = get_gmail_credentials(
    token_file="token.json",
    scopes=["https://mail.google.com/"],
    client_secrets_file="credentials.json",
)
api_resource = build_resource_service(credentials=credentials)
toolkit = GmailToolkit(api_resource=api_resource)
```