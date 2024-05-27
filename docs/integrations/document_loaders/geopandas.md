# Geopandas

[Geopandas](https://geopandas.org/en/stable/index.html) 是一个开源项目，旨在使在 Python 中处理地理空间数据变得更加容易。

GeoPandas 扩展了 pandas 使用的数据类型，以允许对几何类型进行空间操作。

几何操作由 shapely 执行。Geopandas 还依赖于 fiona 进行文件访问和 matplotlib 进行绘图。

利用地理空间数据的 LLM 应用（聊天、问答）是一个有趣的探索领域。

```python
%pip install --upgrade --quiet  sodapy
%pip install --upgrade --quiet  pandas
%pip install --upgrade --quiet  geopandas
```

```python
import ast
import geopandas as gpd
import pandas as pd
from langchain_community.document_loaders import OpenCityDataLoader
```

以 [`Open City Data`](/docs/integrations/document_loaders/open_city_data) 为示例输入创建一个 GeoPandas dataframe。

```python
# 加载 Open City Data
dataset = "tmnf-yvry"  # 旧金山犯罪数据
loader = OpenCityDataLoader(city_id="data.sfgov.org", dataset_id=dataset, limit=5000)
docs = loader.load()
```

```python
# 将字典列表转换为 DataFrame
df = pd.DataFrame([ast.literal_eval(d.page_content) for d in docs])
# 提取纬度和经度
df["Latitude"] = df["location"].apply(lambda loc: loc["coordinates"][1])
df["Longitude"] = df["location"].apply(lambda loc: loc["coordinates"][0])
# 创建 geopandas 数据框
gdf = gpd.GeoDataFrame(
    df, geometry=gpd.points_from_xy(df.Longitude, df.Latitude), crs="EPSG:4326"
)
# 仅保留旧金山的有效经度和纬度
gdf = gdf[
    (gdf["Longitude"] >= -123.173825)
    & (gdf["Longitude"] <= -122.281780)
    & (gdf["Latitude"] >= 37.623983)
    & (gdf["Latitude"] <= 37.929824)
]
```

对旧金山犯罪数据样本进行可视化。

```python
import matplotlib.pyplot as plt
# 加载旧金山地图数据
sf = gpd.read_file("https://data.sfgov.org/resource/3psu-pn9h.geojson")
# 绘制旧金山地图和点
fig, ax = plt.subplots(figsize=(10, 10))
sf.plot(ax=ax, color="white", edgecolor="black")
gdf.plot(ax=ax, color="red", markersize=5)
plt.show()
```

将 GeoPandas 数据框加载为 `Document` 以进行下游处理（嵌入、聊天等）。

`geometry` 将成为默认的 `page_content` 列，所有其他列都放在 `metadata` 中。

但是，我们可以指定 `page_content_column`。

```python
from langchain_community.document_loaders import GeoDataFrameLoader
loader = GeoDataFrameLoader(data_frame=gdf, page_content_column="geometry")
docs = loader.load()
```

```python
docs[0]
```

```output
Document(page_content='POINT (-122.420084075249 37.7083109744362)', metadata={'pdid': '4133422003074', 'incidntnum': '041334220', 'incident_code': '03074', 'category': 'ROBBERY', 'descript': 'ROBBERY, BODILY FORCE', 'dayofweek': 'Monday', 'date': '2004-11-22T00:00:00.000', 'time': '17:50', 'pddistrict': 'INGLESIDE', 'resolution': 'NONE', 'address': 'GENEVA AV / SANTOS ST', 'x': '-122.420084075249', 'y': '37.7083109744362', 'location': {'type': 'Point', 'coordinates': [-122.420084075249, 37.7083109744362]}, ':@computed_region_26cr_cadq': '9', ':@computed_region_rxqg_mtj9': '8', ':@computed_region_bh8s_q3mv': '309', ':@computed_region_6qbp_sg9q': nan, ':@computed_region_qgnn_b9vv': nan, ':@computed_region_ajp5_b2md': nan, ':@computed_region_yftq_j783': nan, ':@computed_region_p5aj_wyqh': nan, ':@computed_region_fyvs_ahh9': nan, ':@computed_region_6pnf_4xz7': nan, ':@computed_region_jwn9_ihcz': nan, ':@computed_region_9dfj_4gjx': nan, ':@computed_region_4isq_27mq': nan, ':@computed_region_pigm_ib2e': nan, ':@computed_region_9jxd_iqea': nan, ':@computed_region_6ezc_tdp2': nan, ':@computed_region_h4ep_8xdi': nan, ':@computed_region_n4xg_c4py': nan, ':@computed_region_fcz8_est8': nan, ':@computed_region_nqbw_i6c3': nan, ':@computed_region_2dwj_jsy4': nan, 'Latitude': 37.7083109744362, 'Longitude': -122.420084075249})
```