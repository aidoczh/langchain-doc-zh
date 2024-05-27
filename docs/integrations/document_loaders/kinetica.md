# Kinetica

这个笔记本介绍了如何从 Kinetica 加载文档。

```python
%pip install gpudb==7.2.0.1
```

```python
from langchain_community.document_loaders.kinetica_loader import KineticaLoader
```

```python
## 加载环境变量
import os
from dotenv import load_dotenv
from langchain_community.vectorstores import (
    KineticaSettings,
)
load_dotenv()
```

```python
# Kinetica 需要连接到数据库。
# 这是如何设置的。
HOST = os.getenv("KINETICA_HOST", "http://127.0.0.1:9191")
USERNAME = os.getenv("KINETICA_USERNAME", "")
PASSWORD = os.getenv("KINETICA_PASSWORD", "")
def create_config() -> KineticaSettings:
    return KineticaSettings(host=HOST, username=USERNAME, password=PASSWORD)
```

```python
from langchain_community.document_loaders.kinetica_loader import KineticaLoader
# 以下的 `QUERY` 是一个示例，不会运行；这
# 需要替换为一个有效的 `QUERY`，它将返回
# 数据，并且 `SCHEMA.TABLE` 的组合必须存在于 Kinetica 中。
QUERY = "select text, survey_id from SCHEMA.TABLE limit 10"
kinetica_loader = KineticaLoader(
    QUERY,
    HOST,
    USERNAME,
    PASSWORD,
)
kinetica_documents = kinetica_loader.load()
print(kinetica_documents)
```

```python
from langchain_community.document_loaders.kinetica_loader import KineticaLoader
# 以下的 `QUERY` 是一个示例，不会运行；这
# 需要替换为一个有效的 `QUERY`，它将返回
# 数据，并且 `SCHEMA.TABLE` 的组合必须存在于 Kinetica 中。
QUERY = "select text, survey_id as source from SCHEMA.TABLE limit 10"
snowflake_loader = KineticaLoader(
    query=QUERY,
    host=HOST,
    username=USERNAME,
    password=PASSWORD,
    metadata_columns=["source"],
)
kinetica_documents = snowflake_loader.load()
print(kinetica_documents)
```