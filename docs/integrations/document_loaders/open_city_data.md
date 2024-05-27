# 开放城市数据

[Socrata](https://dev.socrata.com/foundry/data.sfgov.org/vw6y-z8j6) 提供了一个城市开放数据的 API。

对于像[旧金山犯罪数据](https://data.sfgov.org/Public-Safety/Police-Department-Incident-Reports-Historical-2003/tmnf-yvry)这样的数据集，点击右上角的 `API` 标签页。

这将为您提供 `数据集标识符`。

使用数据集标识符来获取特定表格，针对给定的城市ID (`data.sfgov.org`) -

例如，对于[旧金山 311 数据](https://dev.socrata.com/foundry/data.sfgov.org/vw6y-z8j6)，使用 `vw6y-z8j6`。

例如，对于[旧金山警察数据](https://dev.socrata.com/foundry/data.sfgov.org/tmnf-yvry)，使用 `tmnf-yvry`。

```python
%pip install --upgrade --quiet  sodapy
```

```python
from langchain_community.document_loaders import OpenCityDataLoader
```

```python
dataset = "vw6y-z8j6"  # 311 数据
dataset = "tmnf-yvry"  # 犯罪数据
loader = OpenCityDataLoader(city_id="data.sfgov.org", dataset_id=dataset, limit=2000)
```

```python
docs = loader.load()
```

```output
WARNING:root:Requests made without an app_token will be subject to strict throttling limits.
```

```python
eval(docs[0].page_content)
```

```output
{'pdid': '4133422003074',
 'incidntnum': '041334220',
 'incident_code': '03074',
 'category': 'ROBBERY',
 'descript': 'ROBBERY, BODILY FORCE',
 'dayofweek': 'Monday',
 'date': '2004-11-22T00:00:00.000',
 'time': '17:50',
 'pddistrict': 'INGLESIDE',
 'resolution': 'NONE',
 'address': 'GENEVA AV / SANTOS ST',
 'x': '-122.420084075249',
 'y': '37.7083109744362',
 'location': {'type': 'Point',
  'coordinates': [-122.420084075249, 37.7083109744362]},
 ':@computed_region_26cr_cadq': '9',
 ':@computed_region_rxqg_mtj9': '8',
 ':@computed_region_bh8s_q3mv': '309'}
```