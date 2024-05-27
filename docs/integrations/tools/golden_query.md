# Golden Query

[Golden](https://golden.com) 提供了一套自然语言 API，用于使用 Golden 知识图谱进行查询和丰富，例如查询 `OpenAI 的产品`、`具有 A 轮融资的生成 AI 公司` 和 `投资的说唱歌手` 可以用来检索相关实体的结构化数据。

`golden-query` 是在 [Golden Query API](https://docs.golden.com/reference/query-api) 之上的一个包装工具，可以对这些结果进行编程访问。

有关更多信息，请参阅 [Golden Query API 文档](https://docs.golden.com/reference/query-api)。

本文档介绍了如何使用 `golden-query` 工具。

- 前往 [Golden API 文档](https://docs.golden.com/) 了解 Golden API 的概述。

- 从 [Golden API 设置](https://golden.com/settings/api) 页面获取您的 API 密钥。

- 将您的 API 密钥保存到 GOLDEN_API_KEY 环境变量中。

```python
import os
os.environ["GOLDEN_API_KEY"] = ""
```

```python
from langchain_community.utilities.golden_query import GoldenQueryAPIWrapper
```

```python
golden_query = GoldenQueryAPIWrapper()
```

```python
import json
json.loads(golden_query.run("companies in nanotech"))
```

```output
{'results': [{'id': 4673886,
   'latestVersionId': 60276991,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'Samsung', 'citations': []}]}]},
  {'id': 7008,
   'latestVersionId': 61087416,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'Intel', 'citations': []}]}]},
  {'id': 24193,
   'latestVersionId': 60274482,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'Texas Instruments', 'citations': []}]}]},
  {'id': 1142,
   'latestVersionId': 61406205,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'Advanced Micro Devices', 'citations': []}]}]},
  {'id': 193948,
   'latestVersionId': 58326582,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'Freescale Semiconductor', 'citations': []}]}]},
  {'id': 91316,
   'latestVersionId': 60387380,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'Agilent Technologies', 'citations': []}]}]},
  {'id': 90014,
   'latestVersionId': 60388078,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'Novartis', 'citations': []}]}]},
  {'id': 237458,
   'latestVersionId': 61406160,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'Analog Devices', 'citations': []}]}]},
  {'id': 3941943,
   'latestVersionId': 60382250,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'AbbVie Inc.', 'citations': []}]}]},
  {'id': 4178762,
   'latestVersionId': 60542667,
   'properties': [{'predicateId': 'name',
     'instances': [{'value': 'IBM', 'citations': []}]}]}],
 'next': 'https://golden.com/api/v2/public/queries/59044/results/?cursor=eyJwb3NpdGlvbiI6IFsxNzYxNiwgIklCTS04M1lQM1oiXX0%3D&pageSize=10',
 'previous': None}
```