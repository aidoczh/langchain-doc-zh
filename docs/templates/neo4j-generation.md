# neo4j-generation

这个模板将基于LLM的知识图谱提取与Neo4j AuraDB（一个完全托管的云图数据库）相结合。

你可以在[Neo4j Aura](https://neo4j.com/cloud/platform/aura-graph-database?utm_source=langchain&utm_content=langserve)上创建一个免费实例。

当你初始化一个免费的数据库实例时，你将收到访问数据库的凭据。

这个模板非常灵活，允许用户通过指定节点标签和关系类型来指导提取过程。

有关此软件包的功能和能力的更多详细信息，请参阅[这篇博文](https://blog.langchain.dev/constructing-knowledge-graphs-from-text-using-openai-functions/)。

## 环境设置

你需要设置以下环境变量：

```
OPENAI_API_KEY=<你的OpenAI API密钥>
NEO4J_URI=<你的Neo4j URI>
NEO4J_USERNAME=<你的Neo4j用户名>
NEO4J_PASSWORD=<你的Neo4j密码>
```

## 使用方法

要使用这个软件包，你首先需要安装LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的LangChain项目并将其作为唯一的软件包安装，可以执行以下操作：

```shell
langchain app new my-app --package neo4j-generation
```

如果你想将其添加到现有项目中，只需运行：

```shell
langchain app add neo4j-generation
```

然后将以下代码添加到你的 `server.py` 文件中：

```python
from neo4j_generation.chain import chain as neo4j_generation_chain
add_routes(app, neo4j_generation_chain, path="/neo4j-generation")
```

（可选）现在让我们配置LangSmith。

LangSmith将帮助我们跟踪、监视和调试LangChain应用程序。

你可以在[这里](https://smith.langchain.com/)注册LangSmith。

如果你没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<你的API密钥>
export LANGCHAIN_PROJECT=<你的项目>  # 如果未指定，默认为"default"
```

如果你在此目录中，你可以直接启动一个LangServe实例：

```shell
langchain serve
```

这将在本地启动一个运行在[http://localhost:8000](http://localhost:8000)的FastAPI应用程序。

我们可以在[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)上看到所有模板。

我们可以在[http://127.0.0.1:8000/neo4j-generation/playground](http://127.0.0.1:8000/neo4j-generation/playground)上访问playground。

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable
runnable = RemoteRunnable("http://localhost:8000/neo4j-generation")
```
