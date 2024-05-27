# 笔记链 (维基百科)

实现了由 Yu 等人在 https://arxiv.org/pdf/2311.09210.pdf 中描述的笔记链。使用维基百科进行检索。

查看此处使用的提示 https://smith.langchain.com/hub/bagatur/chain-of-note-wiki。

## 环境设置

使用 Anthropoc 克劳德-3-sonnet-20240229 聊天模型。设置 Anthropoc API 密钥：

```bash
export ANTHROPIC_API_KEY="..."
```

## 用法

要使用此软件包，您应首先安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行：

```shell
langchain app new my-app --package chain-of-note-wiki
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add chain-of-note-wiki
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from chain_of_note_wiki import chain as chain_of_note_wiki_chain
add_routes(app, chain_of_note_wiki_chain, path="/chain-of-note-wiki")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[此处](https://smith.langchain.com/)注册 LangSmith。

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

我们可以在[http://127.0.0.1:8000/chain-of-note-wiki/playground](http://127.0.0.1:8000/chain-of-note-wiki/playground)访问 playground

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/chain-of-note-wiki")
```