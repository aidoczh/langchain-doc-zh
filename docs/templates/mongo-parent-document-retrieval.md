# 使用 MongoDB 和 OpenAI 进行父文档检索

这个模板使用 MongoDB 和 OpenAI 进行 RAG（Retrieval-Augmented Generation）。

它执行的是一种更高级的 RAG，称为父文档检索。

在这种检索方式中，首先将一个大型文档分割成中等大小的块。

然后，将这些中等大小的块再分割成小块。

为这些小块创建嵌入。

当有查询进来时，为该查询创建一个嵌入，并将其与小块进行比较。

但与其直接将小块传递给 LLM（Language Model）进行生成，而是传递来自小块的中等大小的块。

这有助于实现更精细的搜索，同时传递更大的上下文（在生成过程中可能有用）。

## 环境设置

您应该导出两个环境变量，一个是您的 MongoDB URI，另一个是您的 OpenAI API KEY。

如果您没有 MongoDB URI，请参考底部的“设置 MongoDB”部分中的说明进行设置。

```shell
export MONGO_URI=...
export OPENAI_API_KEY=...
```

## 使用方法

要使用此包，您首先应该安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，可以执行以下操作：

```shell
langchain app new my-app --package mongo-parent-document-retrieval
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add mongo-parent-document-retrieval
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from mongo_parent_document_retrieval import chain as mongo_parent_document_retrieval_chain
add_routes(app, mongo_parent_document_retrieval_chain, path="/mongo-parent-document-retrieval")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。

您可以在此处注册 LangSmith：[https://smith.langchain.com/](https://smith.langchain.com/)。

如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您还没有要连接的 Mongo 搜索索引，请在继续之前查看下面的“MongoDB 设置”部分。

请注意，由于父文档检索使用了不同的索引策略，您可能希望运行这个新的设置。

如果您已经有一个要连接的 MongoDB 搜索索引，请编辑 `mongo_parent_document_retrieval/chain.py` 中的连接详细信息。

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，本地运行的服务器位于 [http://localhost:8000](http://localhost:8000)。

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 上查看所有模板。

我们可以在 [http://127.0.0.1:8000/mongo-parent-document-retrieval/playground](http://127.0.0.1:8000/mongo-parent-document-retrieval/playground) 上访问 playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/mongo-parent-document-retrieval")
```

有关更多上下文，请参考[此笔记本](https://colab.research.google.com/drive/1cr2HBAHyBmwKUerJq2if0JaNhy-hIq7I#scrollTo=TZp7_CBfxTOB)。

## MongoDB 设置

如果您需要设置 MongoDB 帐户并导入数据，请按照以下步骤进行操作。

首先，我们将按照标准的 MongoDB Atlas 设置说明进行操作：[https://www.mongodb.com/docs/atlas/getting-started/](https://www.mongodb.com/docs/atlas/getting-started/)。

1. 创建一个帐户（如果尚未完成）。

2. 创建一个新项目（如果尚未完成）。

3. 找到您的 MongoDB URI。

可以通过转到部署概述页面并连接到您的数据库来完成此操作。

然后我们查看可用的驱动程序。

其中我们将看到我们的 URI 列出。

然后将其设置为本地环境变量：

```shell
export MONGO_URI=...
```

4. 让我们还为 OpenAI 设置一个环境变量（我们将使用它作为 LLM）：

```shell
export OPENAI_API_KEY=...
```

5. 现在让我们导入一些数据！我们可以通过进入此目录并运行 `ingest.py` 中的代码来实现，例如：

```shell
python ingest.py
```

请注意，您可以（也应该！）将其更改为导入您选择的数据。

6. 现在我们需要在我们的数据上设置一个向量索引。

我们首先可以连接到我们的数据库所在的集群。

然后我们可以导航到列出所有集合的位置。

然后我们可以找到我们想要的集合，并查看该集合的搜索索引。

那可能是空的，我们要创建一个新的。

我们将使用 JSON 编辑器创建它。

然后我们将粘贴以下 JSON：

```text
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "doc_level": [
        {
          "type": "token"
        }
      ],
      "embedding": {
        "dimensions": 1536,
        "similarity": "cosine",
        "type": "knnVector"
      }
    }
  }
}
```

```
从那里，点击“下一步”，然后点击“创建搜索索引”。这可能需要一点时间，但之后您应该就可以在您的数据上建立索引了！
```