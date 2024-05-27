# rag-chroma

这个模板使用Chroma和OpenAI进行RAG。

`chain.py`中创建了向量存储，默认情况下索引了一个[关于Agents的热门博客文章](https://lilianweng.github.io/posts/2023-06-23-agent/)，用于问答。

## 环境设置

设置`OPENAI_API_KEY`环境变量以访问OpenAI模型。

## 使用方法

要使用这个包，首先需要安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其作为唯一的包安装，可以执行以下操作：

```shell
langchain app new my-app --package rag-chroma
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-chroma
```

然后将以下代码添加到`server.py`文件中：

```python
from rag_chroma import chain as rag_chroma_chain
add_routes(app, rag_chroma_chain, path="/rag-chroma")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监视和调试LangChain应用程序。

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

这将启动FastAPI应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)上查看所有模板。

我们可以在[http://127.0.0.1:8000/rag-chroma/playground](http://127.0.0.1:8000/rag-chroma/playground)上访问playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-chroma")
```