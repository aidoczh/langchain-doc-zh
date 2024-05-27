

# rag-mongo

这个模板使用 MongoDB 和 OpenAI 执行 RAG。

## 环境设置

您应该导出两个环境变量，一个是您的 MongoDB URI，另一个是您的 OpenAI API 密钥。

如果您没有 MongoDB URI，请参阅底部的 `设置 Mongo` 部分，了解如何操作。

```shell
export MONGO_URI=...
export OPENAI_API_KEY=...
```

## 用法

要使用此软件包，您应该首先安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，您可以执行以下操作：

```shell
langchain app new my-app --package rag-mongo
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add rag-mongo
```

然后将以下代码添加到您的 `server.py` 文件中：

```python
from rag_mongo import chain as rag_mongo_chain
add_routes(app, rag_mongo_chain, path="/rag-mongo")
```

如果要设置摄入管道，可以将以下代码添加到您的 `server.py` 文件中：

```python
from rag_mongo import ingest as rag_mongo_ingest
add_routes(app, rag_mongo_ingest, path="/rag-mongo-ingest")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您尚未拥有要连接的 Mongo 搜索索引，请查看下面的 `MongoDB 设置` 部分后再继续。

如果您已经拥有要连接的 MongoDB 搜索索引，请编辑 `rag_mongo/chain.py` 中的连接详细信息。

如果您在此目录中，可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/rag-mongo/playground](http://127.0.0.1:8000/rag-mongo/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/rag-mongo")
```

有关更多上下文，请参考[此笔记本](https://colab.research.google.com/drive/1cr2HBAHyBmwKUerJq2if0JaNhy-hIq7I#scrollTo=TZp7_CBfxTOB)。

## MongoDB 设置

如果您需要设置 MongoDB 帐户和摄入数据，请执行以下步骤。

首先，我们将按照标准的 MongoDB Atlas 设置说明进行操作 [这里](https://www.mongodb.com/docs/atlas/getting-started/)。

1. 创建一个帐户（如果尚未完成）

2. 创建一个新项目（如果尚未完成）

3. 找到您的 MongoDB URI。

可以通过转到部署概述页面并连接到您的数据库来完成此操作

然后查看可用的驱动程序

其中我们将看到我们的 URI 列出

然后在本地将其设置为环境变量：

```shell
export MONGO_URI=...
```

4. 让我们也为 OpenAI（我们将用作 LLM）设置一个环境变量

```shell
export OPENAI_API_KEY=...
```

5. 现在让我们摄入一些数据！我们可以通过进入此目录并运行 `ingest.py` 中的代码来执行：

```shell
python ingest.py
```

请注意，您可以（而且应该！）将其更改为摄入您选择的数据

6. 现在我们需要在我们的数据上设置一个向量索引。

我们首先连接到我们的数据库所在的集群

然后导航到列出所有集合的位置

然后找到我们想要的集合并查看该集合的搜索索引

那可能是空的，我们想要创建一个新的：

我们将使用 JSON 编辑器来创建它

然后我们将粘贴以下 JSON：

```text
 {
   "mappings": {
     "dynamic": true,
     "fields": {
       "embedding": {
         "dimensions": 1536,
         "similarity": "cosine",
         "type": "knnVector"
       }
     }
   }
 }
```

然后，点击“下一步”，然后“创建搜索索引”。这将需要一点时间，但您应该会在数据上有一个索引！