# Airtable

```python
%pip install --upgrade --quiet  pyairtable
```

```python
from langchain_community.document_loaders import AirtableLoader
```

* 在[这里](https://support.airtable.com/docs/creating-and-using-api-keys-and-access-tokens)获取你的 API 密钥。

* 在[这里](https://airtable.com/developers/web/api/introduction)获取你的数据库 ID。

* 从[这里](https://www.highviewapps.com/kb/where-can-i-find-the-airtable-base-id-and-table-id/#:~:text=Both%20the%20Airtable%20Base%20ID,URL%20that%20begins%20with%20tbl)所示的表格 URL 中获取你的表格 ID。

```python
api_key = "xxx"
base_id = "xxx"
table_id = "xxx"
```

```python
loader = AirtableLoader(api_key, table_id, base_id)
docs = loader.load()
```

将每个表格行作为 `dict` 返回。

```python
len(docs)
```

```output
3
```

```python
eval(docs[0].page_content)
```

```output
{'id': 'recF3GbGZCuh9sXIQ',
 'createdTime': '2023-06-09T04:47:21.000Z',
 'fields': {'Priority': 'High',
  'Status': 'In progress',
  'Name': 'Document Splitters'}}
```