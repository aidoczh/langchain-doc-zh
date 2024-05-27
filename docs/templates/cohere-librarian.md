

# cohere-librarian

这个模板将 Cohere 变成了一个图书管理员。

它演示了使用路由器在处理不同事物的链之间切换的用法：一个带有 Cohere 嵌入向量的向量数据库；一个具有有关图书馆信息的提示的聊天机器人；最后是一个可以访问互联网的 RAG 聊天机器人。

要查看更完整的图书推荐演示，请考虑使用以下数据集中的更大样本替换 books_with_blurbs.csv：https://www.kaggle.com/datasets/jdobrow/57000-books-with-metadata-and-blurbs/ 。

## 环境设置

设置 `COHERE_API_KEY` 环境变量以访问 Cohere 模型。

## 用法

要使用这个包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包安装为唯一包，您可以执行：

```shell
langchain app new my-app --package cohere-librarian
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add cohere-librarian
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from cohere_librarian.chain import chain as cohere_librarian_chain
add_routes(app, cohere_librarian_chain, path="/cohere-librarian")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在这里注册 LangSmith [here](https://smith.langchain.com/)。

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

这将启动一个 FastAPI 应用程序，服务器在本地运行，位于

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://localhost:8000/docs](http://localhost:8000/docs) 查看所有模板

我们可以在 [http://localhost:8000/cohere-librarian/playground](http://localhost:8000/cohere-librarian/playground) 访问 playground

我们可以通过以下代码访问代码中的模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/cohere-librarian")
```