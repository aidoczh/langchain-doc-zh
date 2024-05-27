# Step-Back 问答提示

这个模板复制了“Step-Back”提示技术，通过首先提出一个“step back”问题来提高复杂问题的表现。

这种技术可以与常规问答应用相结合，通过对原始问题和“step back”问题进行检索。

在这篇论文中可以了解更多相关信息 [here](https://arxiv.org/abs/2310.06117)，以及 Cobus Greyling 撰写的一篇优秀博客文章 [here](https://cobusgreyling.medium.com/a-new-prompt-engineering-technique-has-been-introduced-called-step-back-prompting-b00e8954cacb)。

我们将稍微修改提示，以便在此模板中更好地与聊天模型配合使用。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用方法

要使用这个包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一包，您可以执行：

```shell
langchain app new my-app --package stepback-qa-prompting
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add stepback-qa-prompting
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from stepback_qa_prompting.chain import chain as stepback_qa_prompting_chain
add_routes(app, stepback_qa_prompting_chain, path="/stepback-qa-prompting")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在 [here](https://smith.langchain.com/) 注册 LangSmith。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，那么您可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个在本地运行的服务器的 FastAPI 应用程序，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。

我们可以在 [http://127.0.0.1:8000/stepback-qa-prompting/playground](http://127.0.0.1:8000/stepback-qa-prompting/playground) 访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/stepback-qa-prompting")
```