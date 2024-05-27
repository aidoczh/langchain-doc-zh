# 如何拆分 JSON 数据

这个 JSON 拆分器可以在控制块大小的同时拆分 JSON 数据。它按深度优先的方式遍历 JSON 数据，并构建较小的 JSON 块。它会尽量保持嵌套的 JSON 对象完整，但如果需要保持块大小在最小块大小和最大块大小之间，则会将它们拆分。

如果值不是嵌套的 JSON，而是一个非常大的字符串，则不会对该字符串进行拆分。如果您需要对块大小进行硬限制，请考虑将其与递归文本拆分器一起使用。还有一个可选的预处理步骤，可以通过首先将列表转换为 JSON（字典），然后按此方式拆分它们。

1. 文本如何拆分：JSON 值。

2. 块大小如何测量：按字符数。

```python
%pip install -qU langchain-text-splitters
```

首先加载一些 JSON 数据：

```python
import json
import requests
# 这是一个大型嵌套的 JSON 对象，将被加载为 Python 字典
json_data = requests.get("https://api.smith.langchain.com/openapi.json").json()
```

## 基本用法

指定 `max_chunk_size` 来限制块大小：

```python
from langchain_text_splitters import RecursiveJsonSplitter
splitter = RecursiveJsonSplitter(max_chunk_size=300)
```

要获取 JSON 块，请使用 `.split_json` 方法：

```python
# 递归拆分 JSON 数据 - 如果您需要访问/操作较小的 JSON 块
json_chunks = splitter.split_json(json_data=json_data)
for chunk in json_chunks[:3]:
    print(chunk)
```

```output
{'openapi': '3.1.0', 'info': {'title': 'LangSmith', 'version': '0.1.0'}, 'servers': [{'url': 'https://api.smith.langchain.com', 'description': 'LangSmith API endpoint.'}]}
{'paths': {'/api/v1/sessions/{session_id}': {'get': {'tags': ['tracer-sessions'], 'summary': 'Read Tracer Session', 'description': 'Get a specific session.', 'operationId': 'read_tracer_session_api_v1_sessions__session_id__get'}}}}
{'paths': {'/api/v1/sessions/{session_id}': {'get': {'security': [{'API Key': []}, {'Tenant ID': []}, {'Bearer Auth': []}]}}}}
```

要获取 LangChain [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) 对象，请使用 `.create_documents` 方法：

```python
# 拆分器还可以输出文档
docs = splitter.create_documents(texts=[json_data])
for doc in docs[:3]:
    print(doc)
```

```output
page_content='{"openapi": "3.1.0", "info": {"title": "LangSmith", "version": "0.1.0"}, "servers": [{"url": "https://api.smith.langchain.com", "description": "LangSmith API endpoint."}]}'
page_content='{"paths": {"/api/v1/sessions/{session_id}": {"get": {"tags": ["tracer-sessions"], "summary": "Read Tracer Session", "description": "Get a specific session.", "operationId": "read_tracer_session_api_v1_sessions__session_id__get"}}}}'
page_content='{"paths": {"/api/v1/sessions/{session_id}": {"get": {"security": [{"API Key": []}, {"Tenant ID": []}, {"Bearer Auth": []}]}}}}'
```

或者使用 `.split_text` 直接获取字符串内容：

```python
texts = splitter.split_text(json_data=json_data)
print(texts[0])
print(texts[1])
```

```output
{"openapi": "3.1.0", "info": {"title": "LangSmith", "version": "0.1.0"}, "servers": [{"url": "https://api.smith.langchain.com", "description": "LangSmith API endpoint."}]}
{"paths": {"/api/v1/sessions/{session_id}": {"get": {"tags": ["tracer-sessions"], "summary": "Read Tracer Session", "description": "Get a specific session.", "operationId": "read_tracer_session_api_v1_sessions__session_id__get"}}}}
```

## 如何管理列表内容的块大小

请注意，此示例中的一个块大于指定的 `max_chunk_size`（300）。查看其中一个较大的块，我们可以看到其中有一个列表对象：

```python
print([len(text) for text in texts][:10])
print()
print(texts[3])
```

```output
[171, 231, 126, 469, 210, 213, 237, 271, 191, 232]
{"paths": {"/api/v1/sessions/{session_id}": {"get": {"parameters": [{"name": "session_id", "in": "path", "required": true, "schema": {"type": "string", "format": "uuid", "title": "Session Id"}}, {"name": "include_stats", "in": "query", "required": false, "schema": {"type": "boolean", "default": false, "title": "Include Stats"}}, {"name": "accept", "in": "header", "required": false, "schema": {"anyOf": [{"type": "string"}, {"type": "null"}], "title": "Accept"}}]}}}}
```

默认情况下，JSON 拆分器不会拆分列表。

指定 `convert_lists=True` 来预处理 JSON，将列表内容转换为具有 `index:item` 作为 `key:val` 对的字典：

```python
texts = splitter.split_text(json_data=json_data, convert_lists=True)
```

让我们看看块的大小。现在它们都在最大块大小以下：

```python
print([len(text) for text in texts][:10])
```

```output
[176, 236, 141, 203, 212, 221, 210, 213, 242, 291]
```

列表已转换为字典，即使分成多个块，仍保留所有所需的上下文信息：

```python
print(texts[1])
```

```output
{"paths": {"/api/v1/sessions/{session_id}": {"get": {"tags": {"0": "tracer-sessions"}, "summary": "Read Tracer Session", "description": "Get a specific session.", "operationId": "read_tracer_session_api_v1_sessions__session_id__get"}}}}
```

```python
# 我们也可以查看文档
docs[1]
```

```output
Document(page_content='{"paths": {"/api/v1/sessions/{session_id}": {"get": {"tags": ["tracer-sessions"], "summary": "Read Tracer Session", "description": "Get a specific session.", "operationId": "read_tracer_session_api_v1_sessions__session_id__get"}}}}')
```