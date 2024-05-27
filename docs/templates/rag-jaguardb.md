# rag-jaguardb

这个模板使用 JaguarDB 和 OpenAI 执行 RAG。

## 环境设置

你需要导出两个环境变量，一个是你的 Jaguar URI，另一个是你的 OpenAI API KEY。

如果你还没有设置 JaguarDB，请参考底部的 `设置 Jaguar` 部分的说明。

```shell
export JAGUAR_API_KEY=...
export OPENAI_API_KEY=...
```

## 使用方法

要使用这个包，你首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，可以执行以下命令：

```shell
langchain app new my-app --package rag-jaguardb
```

如果你想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-jagaurdb
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from rag_jaguardb import chain as rag_jaguardb
add_routes(app, rag_jaguardb_chain, path="/rag-jaguardb")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

你可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果你没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果你在此目录中，可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，本地运行的服务器位于

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上查看所有模板。

我们可以在 [http://127.0.0.1:8000/rag-jaguardb/playground](http://127.0.0.1:8000/rag-jaguardb/playground) 上访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-jaguardb")
```

## JaguarDB 设置

要使用 JaguarDB，你可以使用 `docker pull` 和 `docker run` 命令快速设置 JaguarDB。

```shell
docker pull jaguardb/jaguardb 
docker run -d -p 8888:8888 --name jaguardb jaguardb/jaguardb
```

要启动 JaguarDB 客户端终端与 JaguarDB 服务器进行交互：

```shell 
docker exec -it jaguardb /home/jaguar/jaguar/bin/jag
```

另一种选择是在 Linux 上下载已构建的 JaguarDB 二进制包，并在单个节点或节点集群上部署数据库。简化的流程使您能够快速开始使用 JaguarDB 并利用其强大的功能和功能。[这里](http://www.jaguardb.com/download.html)。