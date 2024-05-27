# neo4j-advanced-rag

这个模板允许您通过实施高级检索策略来平衡精确的嵌入和上下文保留。

## 策略

1. **典型的 RAG**:

   - 传统方法，其中精确索引的数据就是检索的数据。

2. **父级检索器**:

   - 不是索引整个文档，而是将数据分成更小的块，称为父文档和子文档。

   - 子文档被索引以更好地表示特定概念，而父文档被检索以确保保留上下文。

3. **假设性问题**:

     - 处理文档以确定它们可能回答的潜在问题。

     - 然后对这些问题进行索引，以更好地表示特定概念，同时检索父文档以确保保留上下文。

4. **摘要**:

     - 不是索引整个文档，而是创建并索引文档的摘要。

     - 类似地，在 RAG 应用中检索父文档。

## 环境设置

您需要定义以下环境变量

```
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

## 数据填充

如果您想用一些示例数据填充数据库，可以运行 `python ingest.py`。

该脚本会处理并存储来自文件 `dune.txt` 的文本部分到 Neo4j 图数据库中。

首先，将文本分成较大的块（“父级”），然后进一步细分为较小的块（“子级”），其中父级和子级块都有轻微重叠以保持上下文。

将这些块存储在数据库中后，使用 OpenAI 的嵌入计算子节点的嵌入，并将其存储回图中以供将来检索或分析。

对于每个父节点，生成、嵌入并添加了假设性问题和摘要到数据库。

此外，为每种检索策略创建了一个向量索引，以便对这些嵌入进行高效查询。

*请注意，由于 LLM 生成假设性问题和摘要的速度较快，因此摄取可能需要一两分钟的时间。*

## 使用方法

要使用此软件包，您应该首先安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，可以执行以下操作：

```shell
langchain app new my-app --package neo4j-advanced-rag
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add neo4j-advanced-rag
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from neo4j_advanced_rag import chain as neo4j_advanced_chain
add_routes(app, neo4j_advanced_chain, path="/neo4j-advanced-rag")
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

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为

[http://localhost:8000](http://localhost:8000)

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)看到所有模板

我们可以在[http://127.0.0.1:8000/neo4j-advanced-rag/playground](http://127.0.0.1:8000/neo4j-advanced-rag/playground)访问 playground  

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/neo4j-advanced-rag")
```