# NebulaGraph

[NebulaGraph](https://www.nebula-graph.io/) 是一款面向超大规模图数据库的开源、分布式、可扩展且响应迅速的图数据库，具有毫秒级的延迟。它使用 `nGQL` 图查询语言。

[nGQL](https://docs.nebula-graph.io/3.0.0/3.ngql-guide/1.nGQL-overview/1.overview/) 是专为 `NebulaGraph` 设计的声明式图查询语言。它允许表达丰富且高效的图模式。`nGQL` 旨在为开发人员和运维专业人员提供服务。`nGQL` 是一种类似于 SQL 的查询语言。

这篇笔记展示了如何使用LLMs为 `NebulaGraph` 数据库提供自然语言接口。

## 设置

您可以通过运行以下脚本将 `NebulaGraph` 集群作为 Docker 容器启动：

```bash
curl -fsSL nebula-up.siwei.io/install.sh | bash
```

其他选项包括：

- 安装为 [Docker Desktop Extension](https://www.docker.com/blog/distributed-cloud-native-graph-database-nebulagraph-docker-extension/)。请参阅[这里](https://docs.nebula-graph.io/3.5.0/2.quick-start/1.quick-start-workflow/)

- NebulaGraph 云服务。请参阅[这里](https://www.nebula-graph.io/cloud)

- 通过软件包、源代码或 Kubernetes 部署。请参阅[这里](https://docs.nebula-graph.io/)

一旦集群运行起来，我们可以为数据库创建 `SPACE` 和 `SCHEMA`。

```python
%pip install --upgrade --quiet  ipython-ngql
%load_ext ngql
# 连接 ngql jupyter 扩展到 nebulagraph
%ngql --address 127.0.0.1 --port 9669 --user root --password nebula
# 创建一个新的 space
%ngql CREATE SPACE IF NOT EXISTS langchain(partition_num=1, replica_factor=1, vid_type=fixed_string(128));
```

```python
# 等待几秒钟以便 space 创建完成。
%ngql USE langchain;
```

创建 schema，完整数据集，请参考[这里](https://www.siwei.io/en/nebulagraph-etl-dbt/)。

```python
%%ngql
CREATE TAG IF NOT EXISTS movie(name string);
CREATE TAG IF NOT EXISTS person(name string, birthdate string);
CREATE EDGE IF NOT EXISTS acted_in();
CREATE TAG INDEX IF NOT EXISTS person_index ON person(name(128));
CREATE TAG INDEX IF NOT EXISTS movie_index ON movie(name(128));
```

等待 schema 创建完成，然后我们可以插入一些数据。

```python
%%ngql
INSERT VERTEX person(name, birthdate) VALUES "Al Pacino":("Al Pacino", "1940-04-25");
INSERT VERTEX movie(name) VALUES "The Godfather II":("The Godfather II");
INSERT VERTEX movie(name) VALUES "The Godfather Coda: The Death of Michael Corleone":("The Godfather Coda: The Death of Michael Corleone");
INSERT EDGE acted_in() VALUES "Al Pacino"->"The Godfather II":();
INSERT EDGE acted_in() VALUES "Al Pacino"->"The Godfather Coda: The Death of Michael Corleone":();
```

```python
from langchain.chains import NebulaGraphQAChain
from langchain_community.graphs import NebulaGraph
from langchain_openai import ChatOpenAI
```

```python
graph = NebulaGraph(
    space="langchain",
    username="root",
    password="nebula",
    address="127.0.0.1",
    port=9669,
    session_pool_size=30,
)
```

## 刷新图 schema 信息

如果数据库的 schema 发生变化，您可以刷新生成 nGQL 语句所需的 schema 信息。

```python
# graph.refresh_schema()
```

```python
print(graph.get_schema)
```

```output
节点属性: [{'tag': 'movie', 'properties': [('name', 'string')]}, {'tag': 'person', 'properties': [('name', 'string'), ('birthdate', 'string')]}]
边属性: [{'edge': 'acted_in', 'properties': []}]
关系: ['(:person)-[:acted_in]->(:movie)']
```

## 查询图

现在我们可以使用图 cypher QA 链来询问图的问题。

```python
chain = NebulaGraphQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True
)
```

```python
chain.run("Who played in The Godfather II?")
```

```output
> 进入新的 NebulaGraphQAChain 链...
生成的 nGQL:
MATCH (p:`person`)-[:acted_in]->(m:`movie`) WHERE m.`movie`.`name` == 'The Godfather II'
RETURN p.`person`.`name`
完整上下文:
{'p.person.name': ['Al Pacino']}
> 链结束。
```

```output
'Al Pacino played in The Godfather II.'
```