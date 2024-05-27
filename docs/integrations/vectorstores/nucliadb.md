# NucliaDB

你可以使用本地的 NucliaDB 实例，也可以使用[Nuclia Cloud](https://nuclia.cloud)。

当使用本地实例时，你需要一个 Nuclia Understanding API 密钥，这样你的文本才能被正确向量化和索引。你可以通过在[https://nuclia.cloud](https://nuclia.cloud)创建一个免费账户，然后[创建一个 NUA 密钥](https://docs.nuclia.dev/docs/docs/using/understanding/intro)来获取密钥。

```python
%pip install --upgrade --quiet  langchain nuclia
```

## 在 nuclia.cloud 中使用

```python
from langchain_community.vectorstores.nucliadb import NucliaDB
API_KEY = "YOUR_API_KEY"
ndb = NucliaDB(knowledge_box="YOUR_KB_ID", local=False, api_key=API_KEY)
```

## 在本地实例中使用

注意：默认情况下 `backend` 设置为 `http://localhost:8080`。

```python
from langchain_community.vectorstores.nucliadb import NucliaDB
ndb = NucliaDB(knowledge_box="YOUR_KB_ID", local=True, backend="http://my-local-server")
```

## 向你的知识库添加和删除文本

```python
ids = ndb.add_texts(["This is a new test", "This is a second test"])
```

```python
ndb.delete(ids=ids)
```

## 在你的知识库中搜索

```python
results = ndb.similarity_search("Who was inspired by Ada Lovelace?")
print(results[0].page_content)
```