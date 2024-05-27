# ArangoDB

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/arangodb/interactive_tutorials/blob/master/notebooks/Langchain.ipynb)

[ArangoDB](https://github.com/arangodb/arangodb) 是一个可扩展的图形数据库系统，可更快地从连接的数据中获取价值。通过单一的查询语言，它支持原生图形、集成搜索引擎和 JSON。`ArangoDB` 可在本地或云端运行。

这个笔记本展示了如何使用LLMs为[ArangoDB](https://github.com/arangodb/arangodb#readme)数据库提供自然语言接口。

## 设置

您可以通过[ArangoDB Docker镜像](https://hub.docker.com/_/arangodb)来运行本地`ArangoDB`实例：

```bash
docker run -p 8529:8529 -e ARANGO_ROOT_PASSWORD= arangodb/arangodb
```

另一种选择是使用[ArangoDB Cloud Connector包](https://github.com/arangodb/adb-cloud-connector#readme)来运行临时云实例：

```python
%%capture
%pip install --upgrade --quiet  python-arango # The ArangoDB Python Driver
%pip install --upgrade --quiet  adb-cloud-connector # The ArangoDB Cloud Instance provisioner
%pip install --upgrade --quiet  langchain-openai
%pip install --upgrade --quiet  langchain
```

```python
# 实例化ArangoDB数据库
import json
from adb_cloud_connector import get_temp_credentials
from arango import ArangoClient
con = get_temp_credentials()
db = ArangoClient(hosts=con["url"]).db(
    con["dbName"], con["username"], con["password"], verify=True
)
print(json.dumps(con, indent=2))
```

```output
Log: requesting new credentials...
Succcess: new credentials acquired
{
  "dbName": "TUT3sp29s3pjf1io0h4cfdsq",
  "username": "TUTo6nkwgzkizej3kysgdyeo8",
  "password": "TUT9vx0qjqt42i9bq8uik4v9",
  "hostname": "tutorials.arangodb.cloud",
  "port": 8529,
  "url": "https://tutorials.arangodb.cloud:8529"
}
```

```python
# 实例化ArangoDB-LangChain图
from langchain_community.graphs import ArangoGraph
graph = ArangoGraph(db)
```

## 填充数据库

我们将依赖于`Python Driver`将我们的[GameOfThrones](https://github.com/arangodb/example-datasets/tree/master/GameOfThrones)数据导入到我们的数据库中。

```python
if db.has_graph("GameOfThrones"):
    db.delete_graph("GameOfThrones", drop_collections=True)
db.create_graph(
    "GameOfThrones",
    edge_definitions=[
        {
            "edge_collection": "ChildOf",
            "from_vertex_collections": ["Characters"],
            "to_vertex_collections": ["Characters"],
        },
    ],
)
documents = [
    {
        "_key": "NedStark",
        "name": "Ned",
        "surname": "Stark",
        "alive": True,
        "age": 41,
        "gender": "male",
    },
    {
        "_key": "CatelynStark",
        "name": "Catelyn",
        "surname": "Stark",
        "alive": False,
        "age": 40,
        "gender": "female",
    },
    {
        "_key": "AryaStark",
        "name": "Arya",
        "surname": "Stark",
        "alive": True,
        "age": 11,
        "gender": "female",
    },
    {
        "_key": "BranStark",
        "name": "Bran",
        "surname": "Stark",
        "alive": True,
        "age": 10,
        "gender": "male",
    },
]
edges = [
    {"_to": "Characters/NedStark", "_from": "Characters/AryaStark"},
    {"_to": "Characters/NedStark", "_from": "Characters/BranStark"},
    {"_to": "Characters/CatelynStark", "_from": "Characters/AryaStark"},
    {"_to": "Characters/CatelynStark", "_from": "Characters/BranStark"},
]
db.collection("Characters").import_bulk(documents)
db.collection("ChildOf").import_bulk(edges)
```

```output
{'error': False,
 'created': 4,
 'errors': 0,
 'empty': 0,
 'updated': 0,
 'ignored': 0,
 'details': []}
```

## 获取和设置ArangoDB模式

在实例化`ArangoDBGraph`对象时生成了初始的`ArangoDB Schema`。以下是模式的获取器和设置器方法，如果您有兴趣查看或修改模式：

```python
# 这里应该是空的模式，因为在ArangoDB数据摄入之前已经初始化了`graph`。
import json
print(json.dumps(graph.schema, indent=4))
```

```output
{
    "Graph Schema": [],
    "Collection Schema": []
}
```

```python
graph.set_schema()
```

```python
# 现在我们可以查看生成的模式
import json
print(json.dumps(graph.schema, indent=4))
```

```output
{
    "Graph Schema": [
        {
            "graph_name": "GameOfThrones",
            "edge_definitions": [
                {
                    "edge_collection": "ChildOf",
                    "from_vertex_collections": [
                        "Characters"
                    ],
                    "to_vertex_collections": [
                        "Characters"
                    ]
                }
            ]
        }
    ],
    "Collection Schema": [
        {
            "collection_name": "ChildOf",
            "collection_type": "edge",
            "edge_properties": [
                {
                    "name": "_key",
                    "type": "str"
                },
                {
                    "name": "_id",
                    "type": "str"
                },
                {
                    "name": "_from",
                    "type": "str"
                },
                {
                    "name": "_to",
                    "type": "str"
                },
                {
                    "name": "_rev",
                    "type": "str"
                }
            ],
            "example_edge": {
                "_key": "266218884025",
                "_id": "ChildOf/266218884025",
                "_from": "Characters/AryaStark",
                "_to": "Characters/NedStark",
                "_rev": "_gVPKGSq---"
            }
        },
        {
            "collection_name": "Characters",
            "collection_type": "document",
            "document_properties": [
                {
                    "name": "_key",
                    "type": "str"
                },
                {
                    "name": "_id",
                    "type": "str"
                },
                {
                    "name": "_rev",
                    "type": "str"
                },
                {
                    "name": "name",
                    "type": "str"
                },
                {
                    "name": "surname",
                    "type": "str"
                },
                {
                    "name": "alive",
                    "type": "bool"
                },
                {
                    "name": "age",
                    "type": "int"
                },
                {
                    "name": "gender",
                    "type": "str"
                }
            ],
            "example_document": {
                "_key": "NedStark",
                "_id": "Characters/NedStark",
                "_rev": "_gVPKGPi---",
                "name": "Ned",
                "surname": "Stark",
                "alive": true,
                "age": 41,
                "gender": "male"
            }
        }
    ]
}
```

## 查询 ArangoDB 数据库

现在我们可以使用 `ArangoDB Graph` QA Chain 来查询我们的数据。

```python
import os
os.environ["OPENAI_API_KEY"] = "your-key-here"
```

```python
from langchain.chains import ArangoGraphQAChain
from langchain_openai import ChatOpenAI
chain = ArangoGraphQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True
)
```

```python
chain.run("Ned Stark 是否还活着？")
```

```output
> 进入新的 ArangoGraphQAChain 链...
AQL 查询 (1):
WITH Characters
FOR character IN Characters
FILTER character.name == "Ned" AND character.surname == "Stark"
RETURN character.alive
AQL 结果:
[True]
> 链结束。
```

```output
'是的，Ned Stark 还活着。'
```

```python
chain.run("Arya Stark 多大了？")
```

```output
> 进入新的 ArangoGraphQAChain 链...
AQL 查询 (1):
WITH Characters
FOR character IN Characters
FILTER character.name == "Arya" && character.surname == "Stark"
RETURN character.age
AQL 结果:
[11]
> 链结束。
```

```output
'Arya Stark 11 岁了。'
```

```python
chain.run("Arya Stark 和 Ned Stark 有亲戚关系吗？")
```

```output
> 进入新的 ArangoGraphQAChain 链...
AQL 查询 (1):
WITH Characters, ChildOf
FOR v, e, p IN 1..1 OUTBOUND 'Characters/AryaStark' ChildOf
    FILTER p.vertices[-1]._key == 'NedStark'
    RETURN p
AQL 结果:
[{'vertices': [{'_key': 'AryaStark', '_id': 'Characters/AryaStark', '_rev': '_gVPKGPi--B', 'name': 'Arya', 'surname': 'Stark', 'alive': True, 'age': 11, 'gender': 'female'}, {'_key': 'NedStark', '_id': 'Characters/NedStark', '_rev': '_gVPKGPi---', 'name': 'Ned', 'surname': 'Stark', 'alive': True, 'age': 41, 'gender': 'male'}], 'edges': [{'_key': '266218884025', '_id': 'ChildOf/266218884025', '_from': 'Characters/AryaStark', '_to': 'Characters/NedStark', '_rev': '_gVPKGSq---'}], 'weights': [0, 1]}
> 链结束。
```

```output
'是的，Arya Stark 和 Ned Stark 有亲戚关系。根据从数据库检索到的信息，他们之间存在关系。Arya Stark 是 Ned Stark 的孩子。'
```

```python
chain.run("Arya Stark 的父母中有人已故吗？")
```

```output
> 进入新的 ArangoGraphQAChain 链...
AQL 查询 (1):
WITH Characters, ChildOf
FOR v, e IN 1..1 OUTBOUND 'Characters/AryaStark' ChildOf
FILTER v.alive == false
RETURN e
AQL 结果:
[{'_key': '266218884027', '_id': 'ChildOf/266218884027', '_from': 'Characters/AryaStark', '_to': 'Characters/CatelynStark', '_rev': '_gVPKGSu---'}
> 链结束。
```

```output
'是的，Arya Stark 的一个父母已故。父母是 Catelyn Stark。'
```

## 链修改器

您可以修改以下 `ArangoDBGraphQAChain` 类变量的值，以修改链结果的行为。

```python
# 指定要返回的 AQL 查询结果的最大数量
chain.top_k = 10
# 指定是否在输出字典中返回 AQL 查询
chain.return_aql_query = True
# 指定是否在输出字典中返回 AQL JSON 结果
chain.return_aql_result = True
# 指定应进行的 AQL 生成尝试的最大数量
chain.max_aql_generation_attempts = 5
# 指定一组 AQL 查询示例，这些示例传递给 AQL 生成提示模板，以促进少量学习。
# 默认为空字符串。
chain.aql_examples = """
# Ned Stark 是否还活着？
RETURN DOCUMENT('Characters/NedStark').alive
# Arya Stark 是否是 Ned Stark 的孩子？
FOR e IN ChildOf
    FILTER e._from == "Characters/AryaStark" AND e._to == "Characters/NedStark"
    RETURN e
"""
```

```python
chain.run("Ned Stark 是否还活着？")
```

```output
> 进入新的 ArangoGraphQAChain 链...
AQL 查询 (1):
RETURN DOCUMENT('Characters/NedStark').alive
AQL 结果:
[True]
> 链结束。
```

```output
'是的，根据数据库中的信息，Ned Stark 还活着。'
```

```python
chain.run("Bran Stark 是否是 Ned Stark 的孩子？")
```

```output
> 进入新的 ArangoGraphQAChain 链...
AQL 查询 (1):
FOR e IN ChildOf
    FILTER e._from == "Characters/BranStark" AND e._to == "Characters/NedStark"
    RETURN e
AQL 结果:
[{'_key': '266218884026', '_id': 'ChildOf/266218884026', '_from': 'Characters/BranStark', '_to': 'Characters/NedStark', '_rev': '_gVPKGSq--_'}]
> 链结束。
```

```output
'是的，根据 ArangoDB 数据库中的信息，Bran Stark 确实是 Ned Stark 的孩子。'
```