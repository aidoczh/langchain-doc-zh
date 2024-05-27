# GigaChat

本笔记本展示了如何使用 LangChain 与 [GigaChat embeddings](https://developers.sber.ru/portal/products/gigachat)。要使用它，您需要安装 ```gigachat``` Python 包。

```python
%pip install --upgrade --quiet  gigachat
```

要获取 GigaChat 凭据，您需要[创建账户](https://developers.sber.ru/studio/login)，并[获取 API 访问权限](https://developers.sber.ru/docs/ru/gigachat/individuals-quickstart)。

## 示例

```python
import os
from getpass import getpass
os.environ["GIGACHAT_CREDENTIALS"] = getpass()
```

```python
from langchain_community.embeddings import GigaChatEmbeddings
embeddings = GigaChatEmbeddings(verify_ssl_certs=False, scope="GIGACHAT_API_PERS")
```

```python
query_result = embeddings.embed_query("The quick brown fox jumps over the lazy dog")
```

```python
query_result[:5]
```

```output
[0.8398333191871643,
 -0.14180311560630798,
 -0.6161925792694092,
 -0.17103666067123413,
 1.2884578704833984]
```