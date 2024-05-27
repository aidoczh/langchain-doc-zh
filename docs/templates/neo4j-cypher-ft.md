

# neo4j-cypher-ft

这个模板允许您使用自然语言与 Neo4j 图数据库进行交互，利用 OpenAI 的 LLM 技术。

其主要功能是将自然语言问题转换为 Cypher 查询（用于查询 Neo4j 数据库的语言），执行这些查询，并根据查询结果提供自然语言响应。

该软件包利用全文索引，有效地将文本值映射到数据库条目，从而增强准确生成 Cypher 语句的能力。

在提供的示例中，全文索引用于将用户查询中的人名和电影名称映射到相应的数据库条目。

## 环境设置

需要设置以下环境变量：

```
OPENAI_API_KEY=<您的 OpenAI API 密钥>
NEO4J_URI=<您的 Neo4j URI>
NEO4J_USERNAME=<您的 Neo4j 用户名>
NEO4J_PASSWORD=<您的 Neo4j 密码>
```

此外，如果您希望使用一些示例数据填充数据库，可以运行 `python ingest.py`。

该脚本将使用示例电影数据填充数据库，并创建一个名为 `entity` 的全文索引，用于将用户输入的人物和电影映射到数据库值，以便生成精确的 Cypher 语句。

## 使用方法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其安装为唯一软件包，可以执行以下操作：

```shell
langchain app new my-app --package neo4j-cypher-ft
```

如果要将其添加到现有项目中，只需运行：

```shell
langchain app add neo4j-cypher-ft
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from neo4j_cypher_ft import chain as neo4j_cypher_ft_chain
add_routes(app, neo4j_cypher_ft_chain, path="/neo4j-cypher-ft")
```

（可选）现在让我们配置 LangSmith。

LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。

您可以在[这里](https://smith.langchain.com/)注册 LangSmith。

如果您没有访问权限，可以跳过此部分

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<您的 API 密钥>
export LANGCHAIN_PROJECT=<您的项目>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动一个 FastAPI 应用程序，本地运行的服务器位于

[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板

我们可以在 [http://127.0.0.1:8000/neo4j-cypher-ft/playground](http://127.0.0.1:8000/neo4j-cypher-ft/playground) 访问 playground

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/neo4j-cypher-ft")
```