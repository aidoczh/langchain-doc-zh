# AirbyteLoader

>[Airbyte](https://github.com/airbytehq/airbyte) 是一个用于从 API、数据库和文件到数据仓库和数据湖的 ELT 管道的数据集成平台。它拥有最大的 ELT 连接器目录，可连接到数据仓库和数据库。

这里介绍如何将 Airbyte 中的任何数据源加载到 LangChain 文档中。

## 安装

为了使用 `AirbyteLoader`，您需要安装 `langchain-airbyte` 集成包。

```python
% pip install -qU langchain-airbyte
```

注意：目前，`airbyte` 库不支持 Pydantic v2。

请降级到 Pydantic v1 以使用此包。

注意：此包目前还需要 Python 3.10+。

## 加载文档

默认情况下，`AirbyteLoader` 将从流中加载任何结构化数据，并输出 yaml 格式的文档。

```python
from langchain_airbyte import AirbyteLoader
loader = AirbyteLoader(
    source="source-faker",
    stream="users",
    config={"count": 10},
)
docs = loader.load()
print(docs[0].page_content[:500])
```
```output
```yaml

academic_degree: PhD

address:

  city: Lauderdale Lakes

  country_code: FI

  postal_code: '75466'

  province: New Jersey

  state: Hawaii

  street_name: Stoneyford

  street_number: '1112'

age: 44

blood_type: "O\u2212"

created_at: '2004-04-02T13:05:27+00:00'

email: bread2099+1@outlook.com

gender: Fluid

height: '1.62'

id: 1

language: Belarusian

name: Moses

nationality: Dutch

occupation: Track Worker

telephone: 1-467-194-2318

title: M.Sc.Tech.

updated_at: '2024-02-27T16:41:01+00:00'

weight: 6

```
您还可以为格式化文档指定自定义提示模板：
```python
from langchain_core.prompts import PromptTemplate
loader_templated = AirbyteLoader(
    source="source-faker",
    stream="users",
    config={"count": 10},
    template=PromptTemplate.from_template(
        "My name is {name} and I am {height} meters tall."
    ),
)
docs_templated = loader_templated.load()
print(docs_templated[0].page_content)
```
```output

My name is Verdie and I am 1.73 meters tall.

```
## 惰性加载文档
`AirbyteLoader` 的一个强大功能是能够从上游源加载大型文档。在处理大型数据集时，默认的 `.load()` 行为可能会很慢且占用大量内存。为了避免这种情况，您可以使用 `.lazy_load()` 方法以更节省内存的方式加载文档。
```python
import time
loader = AirbyteLoader(
    source="source-faker",
    stream="users",
    config={"count": 3},
    template=PromptTemplate.from_template(
        "My name is {name} and I am {height} meters tall."
    ),
)
start_time = time.time()
my_iterator = loader.lazy_load()
print(
    f"Just calling lazy load is quick! This took {time.time() - start_time:.4f} seconds"
)
```
```output

Just calling lazy load is quick! This took 0.0001 seconds

```
您可以在生成文档时迭代它们：
```python
for doc in my_iterator:
    print(doc.page_content)
```
```output

My name is Andera and I am 1.91 meters tall.

My name is Jody and I am 1.85 meters tall.

My name is Zonia and I am 1.53 meters tall.

```
您还可以使用 `.alazy_load()` 以异步方式惰性加载文档：
```python
loader = AirbyteLoader(
    source="source-faker",
    stream="users",
    config={"count": 3},
    template=PromptTemplate.from_template(
        "My name is {name} and I am {height} meters tall."
    ),
)
my_async_iterator = loader.alazy_load()
async for doc in my_async_iterator:
    print(doc.page_content)
```
```output

My name is Carmelina and I am 1.74 meters tall.

My name is Ali and I am 1.90 meters tall.

My name is Rochell and I am 1.83 meters tall.

```

## 配置

`AirbyteLoader` 可以配置以下选项：

- `source`（str，必填）：要从中加载的 Airbyte 源的名称。

- `stream`（str，必填）：要从中加载的流的名称（Airbyte 源可以返回多个流）。

- `config`（dict，必填）：Airbyte 源的配置。

- `template`（PromptTemplate，可选）：用于格式化文档的自定义提示模板。

- `include_metadata`（bool，可选，默认为 True）：是否在输出文档中包含所有字段作为元数据。

大部分配置将在 `config` 中进行，您可以在 [Airbyte 文档](https://docs.airbyte.com/integrations/) 中的每个源的“配置字段参考”中找到特定的配置选项。