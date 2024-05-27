# Rockset

Rockset 是一个实时分析数据库，可以在不增加操作负担的情况下对海量半结构化数据进行查询。使用 Rockset，摄入的数据可以在一秒内进行查询，并且针对该数据的分析查询通常在毫秒内执行。Rockset 是计算优化的，适用于处理子 100TB 范围内的高并发应用程序（或者使用汇总后的大于数百 TB 的数据）。

这篇笔记演示了如何在 langchain 中将 Rockset 用作文档加载程序。要开始，请确保您拥有一个 Rockset 账户和一个可用的 API 密钥。

## 设置环境

1. 前往 [Rockset 控制台](https://console.rockset.com/apikeys) 获取一个 API 密钥。从 [API 参考](https://rockset.com/docs/rest-api/#introduction) 中找到您的 API 区域。在本笔记中，我们假设您正在从 `Oregon(us-west-2)` 使用 Rockset。

2. 设置环境变量 `ROCKSET_API_KEY`。

3. 安装 Rockset Python 客户端，langchain 将使用该客户端与 Rockset 数据库进行交互。

```python
%pip install --upgrade --quiet rockset
```

## 加载文档

Rockset 与 LangChain 的集成允许您使用 SQL 查询从 Rockset 集合加载文档。为此，您必须构建一个 `RocksetLoader` 对象。以下是一个初始化 `RocksetLoader` 的示例代码段。

```python
from langchain_community.document_loaders import RocksetLoader
from rockset import Regions, RocksetClient, models
loader = RocksetLoader(
    RocksetClient(Regions.usw2a1, "<api key>"),
    models.QueryRequestSql(query="SELECT * FROM langchain_demo LIMIT 3"),  # SQL 查询
    ["text"],  # 内容列
    metadata_keys=["id", "date"],  # 元数据列
)
```

在这里，可以看到运行了以下查询：

```sql
SELECT * FROM langchain_demo LIMIT 3
```

集合中的 `text` 列用作页面内容，记录的 `id` 和 `date` 列用作元数据（如果您没有传入任何内容到 `metadata_keys`，整个 Rockset 文档将被用作元数据）。

要执行查询并访问生成的 `Document` 的迭代器，请运行：

```python
loader.lazy_load()
```

要执行查询并一次性访问所有生成的 `Document`，请运行：

```python
loader.load()
```

以下是 `loader.load()` 的示例响应：

```python
[
    Document(
        page_content="Lorem ipsum dolor sit amet, consectetur adipiscing elit. Maecenas a libero porta, dictum ipsum eget, hendrerit neque. Morbi blandit, ex ut suscipit viverra, enim velit tincidunt tellus, a tempor velit nunc et ex. Proin hendrerit odio nec convallis lobortis. Aenean in purus dolor. Vestibulum orci orci, laoreet eget magna in, commodo euismod justo.",
        metadata={"id": 83209, "date": "2022-11-13T18:26:45.000000Z"}
    ),
    Document(
        page_content="Integer at finibus odio. Nam sit amet enim cursus lacus gravida feugiat vestibulum sed libero. Aenean eleifend est quis elementum tincidunt. Curabitur sit amet ornare erat. Nulla id dolor ut magna volutpat sodales fringilla vel ipsum. Donec ultricies, lacus sed fermentum dignissim, lorem elit aliquam ligula, sed suscipit sapien purus nec ligula.",
        metadata={"id": 89313, "date": "2022-11-13T18:28:53.000000Z"}
    ),
    Document(
        page_content="Morbi tortor enim, commodo id efficitur vitae, fringilla nec mi. Nullam molestie faucibus aliquet. Praesent a est facilisis, condimentum justo sit amet, viverra erat. Fusce volutpat nisi vel purus blandit, et facilisis felis accumsan. Phasellus luctus ligula ultrices tellus tempor hendrerit. Donec at ultricies leo.",
        metadata={"id": 87732, "date": "2022-11-13T18:49:04.000000Z"}
    )
]
```

## 使用多列作为内容

您可以选择使用多列作为内容：

```python
from langchain_community.document_loaders import RocksetLoader
from rockset import Regions, RocksetClient, models
loader = RocksetLoader(
    RocksetClient(Regions.usw2a1, "<api key>"),
    models.QueryRequestSql(query="SELECT * FROM langchain_demo LIMIT 1 WHERE id=38"),
    ["sentence1", "sentence2"],  # 两个内容列
)
```

假设 "sentence1" 字段是 `"This is the first sentence."`，"sentence2" 字段是 `"This is the second sentence."`，则生成的 `Document` 的 `page_content` 将是：

```
This is the first sentence.
This is the second sentence.
```

您可以通过在 `RocksetLoader` 构造函数中设置 `content_columns_joiner` 参数来定义自己的函数以连接内容列。`content_columns_joiner` 是一个方法，接受 `List[Tuple[str, Any]]]` 作为参数，表示 (列名，列值) 元组列表。默认情况下，这是一个将每个列值与新行连接的方法。

举例来说，如果你想要用空格而不是换行符将 sentence1 和 sentence2 连接起来，你可以这样设置 `content_columns_joiner`：

```python
RocksetLoader(
    RocksetClient(Regions.usw2a1, "<api key>"),
    models.QueryRequestSql(query="SELECT * FROM langchain_demo LIMIT 1 WHERE id=38"),
    ["sentence1", "sentence2"],
    content_columns_joiner=lambda docs: " ".join(
        [doc[1] for doc in docs]
    ),  # join with space instead of /n
)
```

生成的 `Document` 的 `page_content` 将会是：

```
This is the first sentence. This is the second sentence.
```

通常你可能希望在 `page_content` 中包含列名。你可以这样做：

```python
RocksetLoader(
    RocksetClient(Regions.usw2a1, "<api key>"),
    models.QueryRequestSql(query="SELECT * FROM langchain_demo LIMIT 1 WHERE id=38"),
    ["sentence1", "sentence2"],
    content_columns_joiner=lambda docs: "\n".join(
        [f"{doc[0]}: {doc[1]}" for doc in docs]
    ),
)
```

这将导致以下 `page_content`：

```
sentence1: This is the first sentence.
sentence2: This is the second sentence.
```