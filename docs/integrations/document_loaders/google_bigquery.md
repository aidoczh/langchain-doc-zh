# Google BigQuery

[Google BigQuery](https://cloud.google.com/bigquery) 是一个无服务器且成本效益高的企业数据仓库，可跨云工作并随数据扩展。`BigQuery` 是 `Google Cloud Platform` 的一部分。

将每行加载一个文档的 `BigQuery` 查询。

```python
%pip install --upgrade --quiet langchain-google-community[bigquery]
```

```python
from langchain_google_community import BigQueryLoader
```

```python
BASE_QUERY = """
SELECT
  id,
  dna_sequence,
  organism
FROM (
  SELECT
    ARRAY (
    SELECT
      AS STRUCT 1 AS id, "ATTCGA" AS dna_sequence, "Lokiarchaeum sp. (strain GC14_75)." AS organism
    UNION ALL
    SELECT
      AS STRUCT 2 AS id, "AGGCGA" AS dna_sequence, "Heimdallarchaeota archaeon (strain LC_2)." AS organism
    UNION ALL
    SELECT
      AS STRUCT 3 AS id, "TCCGGA" AS dna_sequence, "Acidianus hospitalis (strain W1)." AS organism) AS new_array),
  UNNEST(new_array)
"""
```

## 基本用法

```python
loader = BigQueryLoader(BASE_QUERY)
data = loader.load()
```

```python
print(data)
```

```output
[Document(page_content='id: 1\ndna_sequence: ATTCGA\norganism: Lokiarchaeum sp. (strain GC14_75).', lookup_str='', metadata={}, lookup_index=0), Document(page_content='id: 2\ndna_sequence: AGGCGA\norganism: Heimdallarchaeota archaeon (strain LC_2).', lookup_str='', metadata={}, lookup_index=0), Document(page_content='id: 3\ndna_sequence: TCCGGA\norganism: Acidianus hospitalis (strain W1).', lookup_str='', metadata={}, lookup_index=0)]
```

## 指定哪些列是内容，哪些列是元数据

```python
loader = BigQueryLoader(
    BASE_QUERY,
    page_content_columns=["dna_sequence", "organism"],
    metadata_columns=["id"],
)
data = loader.load()
```

```python
print(data)
```

```output
[Document(page_content='dna_sequence: ATTCGA\norganism: Lokiarchaeum sp. (strain GC14_75).', lookup_str='', metadata={'id': 1}, lookup_index=0), Document(page_content='dna_sequence: AGGCGA\norganism: Heimdallarchaeota archaeon (strain LC_2).', lookup_str='', metadata={'id': 2}, lookup_index=0), Document(page_content='dna_sequence: TCCGGA\norganism: Acidianus hospitalis (strain W1).', lookup_str='', metadata={'id': 3}, lookup_index=0)]
```

## 在元数据中添加来源

```python
# 注意，`id` 列被返回两次，其中一个实例被别名为 `source`
ALIASED_QUERY = """
SELECT
  id,
  dna_sequence,
  organism,
  id as source
FROM (
  SELECT
    ARRAY (
    SELECT
      AS STRUCT 1 AS id, "ATTCGA" AS dna_sequence, "Lokiarchaeum sp. (strain GC14_75)." AS organism
    UNION ALL
    SELECT
      AS STRUCT 2 AS id, "AGGCGA" AS dna_sequence, "Heimdallarchaeota archaeon (strain LC_2)." AS organism
    UNION ALL
      SELECT
      AS STRUCT 3 AS id, "TCCGGA" AS dna_sequence, "Acidianus hospitalis (strain W1)." AS organism) AS new_array),
  UNNEST(new_array)
"""
```

```python
loader = BigQueryLoader(ALIASED_QUERY, metadata_columns=["source"])
data = loader.load()
```

```python
print(data)
```

```output
[Document(page_content='id: 1\ndna_sequence: ATTCGA\norganism: Lokiarchaeum sp. (strain GC14_75).\nsource: 1', lookup_str='', metadata={'source': 1}, lookup_index=0), Document(page_content='id: 2\ndna_sequence: AGGCGA\norganism: Heimdallarchaeota archaeon (strain LC_2).\nsource: 2', lookup_str='', metadata={'source': 2}, lookup_index=0), Document(page_content='id: 3\ndna_sequence: TCCGGA\norganism: Acidianus hospitalis (strain W1).\nsource: 3', lookup_str='', metadata={'source': 3}, lookup_index=0)]
```