# 思维骨架

实现自[此](https://sites.google.com/view/sot-llm)论文中的“思维骨架”。

这一技术通过首先生成一个骨架，然后逐点生成大纲的每个部分，使得能够更快地生成更长的内容。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

要获取您的 `OPENAI_API_KEY`，请转到您的 OpenAI 帐户的[API密钥](https://platform.openai.com/account/api-keys)，然后创建一个新的秘密密钥。

## 使用方法

要使用此软件包，您应首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package skeleton-of-thought
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add skeleton-of-thought
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from skeleton_of_thought import chain as skeleton_of_thought_chain
add_routes(app, skeleton_of_thought_chain, path="/skeleton-of-thought")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[此处](https://smith.langchain.com/)注册 LangSmith。

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

我们可以在[http://127.0.0.1:8000/skeleton-of-thought/playground](http://127.0.0.1:8000/skeleton-of-thought/playground)访问 playground

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/skeleton-of-thought")
```